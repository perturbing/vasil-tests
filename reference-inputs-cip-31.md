# How to use Reference inputs?
In this tutorial, we will show how to use reference inputs as a new feature of the Vasil HF.

## Input vs. Reference Input?
The difference between a simple Input and a Reference input is that the reference input will never be spent. But, why do we need it? It is used to read data contained in the input. As we saw in the *referencing-scripts-cip-33* tutorial, we use the Script contained in the input. In the Babbage era, an input contains the following information:

``` haskell
-- | An input of a pending transaction.
data TxInInfo = TxInInfo
    { txInInfoOutRef   :: TxOutRef
    , txInInfoResolved :: TxOut
    } deriving stock (Generic, Haskell.Show, Haskell.Eq)
```

The `txInInfoOutRef` only contains the transaction hash and the index of the input. But the TxOut contains 4 things:

```haskell
data TxOut = TxOut {
    txOutAddress         :: Address,
    txOutValue           :: Value,
    txOutDatum           :: OutputDatum,
    txOutReferenceScript :: Maybe ScriptHash
    }
```
The target address, a value, optionally a datum/datum hash, and optionally a reference script. There are multiple benefits of having access to data of an input without spending them:
* Cheaper transactions since we are not consuming the input.
* Faster transactions because we're not creating new inputs.
* More concurrency, many transactions can reference the input at the same time.

## An example
In this example, we will write a simple contract that reads the datum of an input without spending it. The example that fits better to this feature is an Oracle. We sometimes need to get information from outside the blockchain. We will store that data in the datum of an input. Furthermore, we will only show a simple usage of the reference input to keep it simple.

We need an input containing some information to make use of the reference input. To do that, we will store information in an address with the following `cardano-cli` commands:

First, we need to get UTxOs of my address:

```
$ cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 9
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
ac5dbe96fca116e9fdcc24d098a265af5491b9b41da8c5a638e2820c13941121     0        100988999123 lovelace + TxOutDatumNone
```

Then I'll select the `ac5dbe...941121#0` UTxO to create a new one.

```
$ cardano-cli transaction build --babbage-era --testnet-magic 9 \
--tx-in ac5dbe96fca116e9fdcc24d098a265af5491b9b41da8c5a638e2820c13941121#0 \
--tx-out $(cat payment.addr)+50000000 \
--tx-out-inline-datum-value 42 \
--change-address $(cat payment.addr) \
--out-file tx.raw
Estimated transaction fee: Lovelace 166073

$ cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--testnet-magic 9 \
--out-file tx.signed

$ cardano-cli transaction submit \
--testnet-magic 9 \
--tx-file tx.signed
Transaction successfully submitted.

```
After signing and submitting the transaction, we can now query my address again:
```
$ cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 9
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
9b1eacf97a6a8ea87f2e677daec26e0a81fe500d6243a7682a72c17d950df1db     0        100938833050 lovelace + TxOutDatumNone
9b1eacf97a6a8ea87f2e677daec26e0a81fe500d6243a7682a72c17d950df1db     1        50000000 lovelace + TxOutDatumInline ReferenceTxInsScriptsInlineDatumsInBabbageEra (ScriptDataNumber 42)
```
Now that we have a new UTxO in my address that contains a datum, we can use it as reference input in subsequent transactions. For this example, we will create a new Validator script that only allows you to spend funds if you include that UTxO as Reference input.

```haskell
{-# LANGUAGE DataKinds             #-}
{-# LANGUAGE DeriveAnyClass        #-}
{-# LANGUAGE DeriveGeneric         #-}
{-# LANGUAGE FlexibleContexts      #-}
{-# LANGUAGE FlexibleInstances     #-}
{-# LANGUAGE MultiParamTypeClasses #-}
{-# LANGUAGE NamedFieldPuns        #-}
{-# LANGUAGE NoImplicitPrelude     #-}
{-# LANGUAGE OverloadedStrings     #-}
{-# LANGUAGE ScopedTypeVariables   #-}
{-# LANGUAGE TemplateHaskell       #-}
{-# LANGUAGE TypeApplications      #-}
{-# LANGUAGE TypeFamilies          #-}
{-# LANGUAGE TypeOperators         #-}

module Cardano.PlutusOracle.ReferenceInput
  ( apiReferenceInputScript
  , scriptAsCbor
  , writeSerialisedScript
  ) where
import           Cardano.Api
import           Cardano.Api.Shelley             (PlutusScript (..))
import           Codec.Serialise
import qualified Data.ByteString.Lazy            as LBS
import qualified Data.ByteString.Short           as SBS
import qualified Plutus.Script.Utils.V2.Scripts  as PSU.V2
import qualified Plutus.V2.Ledger.Api            as PlutusV2
import qualified Plutus.V2.Ledger.Contexts       as PlutusV2
import           Plutus.V2.Ledger.Tx
import qualified PlutusTx
import           PlutusTx.Prelude                as P hiding (unless, (.))

import qualified Prelude

{-# INLINABLE mkReferenceInputValidator #-}
mkReferenceInputValidator :: BuiltinData -> Integer -> PlutusV2.ScriptContext -> Bool
mkReferenceInputValidator  _ r ctx =
    traceIfFalse "token missing from input"  correctDatum

  where
    info :: PlutusV2.TxInfo
    info = PlutusV2.scriptContextTxInfo ctx

    correctDatum :: Bool
    correctDatum = getDatum == Just r

    getDatum :: Maybe Integer
    getDatum = case PlutusV2.txInfoReferenceInputs info of
      [reference] -> case txOutDatum $ PlutusV2.txInInfoResolved reference of
        NoOutputDatum       -> Nothing
        OutputDatumHash odh -> case PlutusV2.findDatum odh info of
                                 Just d  -> PlutusTx.fromBuiltinData $ PlutusV2.getDatum $ d
                                 Nothing -> Nothing
        OutputDatum     od  -> PlutusTx.fromBuiltinData $ PlutusV2.getDatum od
      _           -> Nothing

referenceInputValidator :: PlutusV2.Validator
referenceInputValidator = PlutusV2.mkValidatorScript
    ($$(PlutusTx.compile [|| wrap ||]))
  where
    wrap = PSU.V2.mkUntypedValidator mkReferenceInputValidator

scriptAsCbor :: SBS.ShortByteString
scriptAsCbor = SBS.toShort $ LBS.toStrict $ serialise referenceInputValidator

script :: PlutusV2.Script
script = PlutusV2.unValidatorScript referenceInputValidator

referenceInputScriptAsShortBs :: SBS.ShortByteString
referenceInputScriptAsShortBs = SBS.toShort $ LBS.toStrict $ serialise script

apiReferenceInputScript :: PlutusScript PlutusScriptV2
apiReferenceInputScript = PlutusScriptSerialised referenceInputScriptAsShortBs

writeSerialisedScript :: Prelude.IO ()
writeSerialisedScript = do
       result <- writeFileTextEnvelope "reference-input.plutus" Nothing apiReferenceInputScript
       case result of
         Left err -> Prelude.print $ displayError err
         Right () -> Prelude.putStrLn $ "wrote script to file reference-input.plutus"

```

The function `writeSerialisedScript` will write the `reference-input.plutus`, it contains the following:

```
cat reference-input.plutus 
{
    "type": "PlutusScriptV2",
    "description": "",
    "cborHex": "5908955908920100003233223232323232332232323232323232323232332232323232322223232533532323232325335001101d13357389211e77726f6e67207573616765206f66207265666572656e636520696e7075740001c3232533500221533500221333573466e1c00800408007c407854cd4004840784078d40900114cd4c8d400488888888888802d40044c08526221533500115333533550222350012222002350022200115024213355023320015021001232153353235001222222222222300e00250052133550253200150233355025200100115026320013550272253350011502722135002225335333573466e3c00801c0940904d40b00044c01800c884c09526135001220023333573466e1cd55cea80224000466442466002006004646464646464646464646464646666ae68cdc39aab9d500c480008cccccccccccc88888888888848cccccccccccc00403403002c02802402001c01801401000c008cd405c060d5d0a80619a80b80c1aba1500b33501701935742a014666aa036eb94068d5d0a804999aa80dbae501a35742a01066a02e0446ae85401cccd5406c08dd69aba150063232323333573466e1cd55cea801240004664424660020060046464646666ae68cdc39aab9d5002480008cc8848cc00400c008cd40b5d69aba15002302e357426ae8940088c98c80c0cd5ce01901a01709aab9e5001137540026ae854008c8c8c8cccd5cd19b8735573aa004900011991091980080180119a816bad35742a004605c6ae84d5d1280111931901819ab9c03203402e135573ca00226ea8004d5d09aba2500223263202c33573805c06005426aae7940044dd50009aba1500533501775c6ae854010ccd5406c07c8004d5d0a801999aa80dbae200135742a00460426ae84d5d1280111931901419ab9c02a02c026135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d55cf280089baa00135742a00860226ae84d5d1280211931900d19ab9c01c01e018375a00a6666ae68cdc39aab9d375400a9000100e11931900c19ab9c01a01c016101b132632017335738921035054350001b135573ca00226ea800448c88c008dd6000990009aa80d911999aab9f0012500a233500930043574200460066ae880080608c8c8cccd5cd19b8735573aa004900011991091980080180118061aba150023005357426ae8940088c98c8050cd5ce00b00c00909aab9e5001137540024646464646666ae68cdc39aab9d5004480008cccc888848cccc00401401000c008c8c8c8cccd5cd19b8735573aa0049000119910919800801801180a9aba1500233500f014357426ae8940088c98c8064cd5ce00d80e80b89aab9e5001137540026ae854010ccd54021d728039aba150033232323333573466e1d4005200423212223002004357426aae79400c8cccd5cd19b875002480088c84888c004010dd71aba135573ca00846666ae68cdc3a801a400042444006464c6403666ae7007407c06406005c4d55cea80089baa00135742a00466a016eb8d5d09aba2500223263201533573802e03202626ae8940044d5d1280089aab9e500113754002266aa002eb9d6889119118011bab00132001355018223233335573e0044a010466a00e66442466002006004600c6aae754008c014d55cf280118021aba200301613574200222440042442446600200800624464646666ae68cdc3a800a400046a02e600a6ae84d55cf280191999ab9a3370ea00490011280b91931900819ab9c01201400e00d135573aa00226ea80048c8c8cccd5cd19b875001480188c848888c010014c01cd5d09aab9e500323333573466e1d400920042321222230020053009357426aae7940108cccd5cd19b875003480088c848888c004014c01cd5d09aab9e500523333573466e1d40112000232122223003005375c6ae84d55cf280311931900819ab9c01201400e00d00c00b135573aa00226ea80048c8c8cccd5cd19b8735573aa004900011991091980080180118029aba15002375a6ae84d5d1280111931900619ab9c00e01000a135573ca00226ea80048c8cccd5cd19b8735573aa002900011bae357426aae7940088c98c8028cd5ce00600700409baa001232323232323333573466e1d4005200c21222222200323333573466e1d4009200a21222222200423333573466e1d400d2008233221222222233001009008375c6ae854014dd69aba135744a00a46666ae68cdc3a8022400c4664424444444660040120106eb8d5d0a8039bae357426ae89401c8cccd5cd19b875005480108cc8848888888cc018024020c030d5d0a8049bae357426ae8940248cccd5cd19b875006480088c848888888c01c020c034d5d09aab9e500b23333573466e1d401d2000232122222223005008300e357426aae7940308c98c804ccd5ce00a80b80880800780700680600589aab9d5004135573ca00626aae7940084d55cf280089baa0012323232323333573466e1d400520022333222122333001005004003375a6ae854010dd69aba15003375a6ae84d5d1280191999ab9a3370ea0049000119091180100198041aba135573ca00c464c6401866ae700380400280244d55cea80189aba25001135573ca00226ea80048c8c8cccd5cd19b875001480088c8488c00400cdd71aba135573ca00646666ae68cdc3a8012400046424460040066eb8d5d09aab9e500423263200933573801601a00e00c26aae7540044dd500089119191999ab9a3370ea00290021091100091999ab9a3370ea00490011190911180180218031aba135573ca00846666ae68cdc3a801a400042444004464c6401466ae7003003802001c0184d55cea80089baa0012323333573466e1d40052002200623333573466e1d40092000200623263200633573801001400800626aae74dd5000a4c244004244002921035054310012333333357480024a00c4a00c4a00c46a00e6eb400894018008480044488c0080049400848488c00800c4488004448c8c00400488cc00cc0080080041"
}
```

The next step is to send funds to the hash of the script.
First we need to get the address of the script, we do it with this line:

```
$ cardano-cli address build --payment-script-file reference-input.plutus --out-file reference-input.addr --testnet-magic 9

$ cat reference-input.addr
addr_test1wzem0yuxjqyrmzvrsr8xfqhumyy555ngyjxw7wrg2pav90q8cagu2
```

We can send funds to the script:
```
$ cardano-cli transaction build --babbage-era --testnet-magic 9 \
--tx-in 9b1eacf97a6a8ea87f2e677daec26e0a81fe500d6243a7682a72c17d950df1db#0 \
--tx-out $(cat reference-input.addr)+10000000 \
--tx-out-inline-datum-file unit.json \
--change-address $(cat payment.addr) \
--out-file tx.raw
Estimated transaction fee: Lovelace 166117

$ cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--testnet-magic 9 \
--out-file tx.signed

$ cardano-cli transaction submit \
--testnet-magic 9 \
--tx-file tx.signed
Transaction successfully submitted.

$ cardano-cli query utxo --address $(cat reference-input.addr) --testnet-magic 9
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
5e341149c550ab4f246f838d9400b6784989cf45c46b94e16c4cc0e311abe216     1        10000000 lovelace + TxOutDatumInline ReferenceTxInsScriptsInlineDatumsInBabbageEra (ScriptDataConstructor 0 [])
```

After a while, we see that there are 10 ADAs blocked in the reference input script. To unlock the ADAs we will need a new transaction with two regular inputs and one input reference.

First, let's query the UTxOs in the payment address:
```
$ cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 9
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
5e341149c550ab4f246f838d9400b6784989cf45c46b94e16c4cc0e311abe216     0        100928666933 lovelace + TxOutDatumNone
9b1eacf97a6a8ea87f2e677daec26e0a81fe500d6243a7682a72c17d950df1db     1        50000000 lovelace + TxOutDatumInline ReferenceTxInsScriptsInlineDatumsInBabbageEra (ScriptDataNumber 42)
```
Then we can create a new transaction:
```
$ cardano-cli transaction build --babbage-era --testnet-magic 9 \
--tx-in-collateral 5e341149c550ab4f246f838d9400b6784989cf45c46b94e16c4cc0e311abe216#0 \
--tx-in 5e341149c550ab4f246f838d9400b6784989cf45c46b94e16c4cc0e311abe216#0 \
--tx-in 5e341149c550ab4f246f838d9400b6784989cf45c46b94e16c4cc0e311abe216#1 \
--tx-in-script-file reference-input.plutus \
--tx-in-inline-datum-present \
--tx-in-redeemer-value 42 \
--read-only-tx-in-reference 9b1eacf97a6a8ea87f2e677daec26e0a81fe500d6243a7682a72c17d950df1db#1 \
--change-address $(cat payment.addr) \
--protocol-params-file pparams.json \
--out-file tx.raw
Estimated transaction fee: Lovelace 325359

$ cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--testnet-magic 9 \
--out-file tx.signed

$ cardano-cli transaction submit \
--testnet-magic 9 \
--tx-file tx.signed
Transaction successfully submitted.
```
## The mkReferenceInputValidator function
The contract will be explained in this section.

```haskell
{-# INLINABLE mkReferenceInputValidator #-}
mkReferenceInputValidator :: BuiltinData -> Integer -> PlutusV2.ScriptContext -> Bool
mkReferenceInputValidator  _ r ctx =
    traceIfFalse "token missing from input"  correctDatum

  where
    info :: PlutusV2.TxInfo
    info = PlutusV2.scriptContextTxInfo ctx

    correctDatum :: Bool
    correctDatum = getDatum == Just r

    getDatum :: Maybe Integer
    getDatum = case PlutusV2.txInfoReferenceInputs info of
      [reference] -> case txOutDatum $ PlutusV2.txInInfoResolved reference of
        NoOutputDatum       -> Nothing
        OutputDatumHash odh -> case PlutusV2.findDatum odh info of
                                 Just d  -> PlutusTx.fromBuiltinData $ PlutusV2.getDatum $ d
                                 Nothing -> Nothing
        OutputDatum     od  -> PlutusTx.fromBuiltinData $ PlutusV2.getDatum od
      _           -> Nothing
```
The purpose of this validator is to make sure that there is only one reference input and the Datum of it matches the redeemer. In this case, we only use the redeemer to match the datum of the Reference Input, but it can also be used for more purposes. Notice that we used `txInfoReferenceInputs` instead `txInfoInputs`. It means that we selected the inputs that won't be spent.

## Considerations
* In this example, we also used CIP 32 on inline Datum.
* The purpose of this tutorial is only to show the usage of Reference Inputs.
* A realistic Oracle should have more programming techniques (For example, having a NFT in the Reference Input to prove the validity).
* In this example, we used a UTxO sitting in a Payment Address as Reference Input, but we also can use a UTxO of Script Addresses.
* As we will see in the Referencing Scripts tutorial, CIP 31 on reference Inputs could be used with CIP 33 on reference Script to upload Plutus script once and reference it every time we need it.

