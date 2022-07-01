# How to use collateral outputs
In this tutorial we will discuss the usage of the new functionality of collateral outputs [(1)](https://github.com/cardano-foundation/CIPs/blob/138565ea4c2303fabc576c0f7f67228a54124b17/CollateralOutput/README.md).
## What is collateral? 
When an users sends a transaction to the Cardano network that uses a script this inflicts some strain of computation on the validation of that transaction. This strain is expressed in the abstract execution units and validators need to be compensated for these calculations. If such a transaction is successful the transactions costs that come from the inputs cover these validation costs. But if the transaction fails by incorrect executing of the script, these transaction inputs can not cover these fees since they are not spendable without correct execution of the script. This while the validator did perform a calculation. To cover for this the creator of the transaction also attaches a collateral to the transaction for it to be even considered for validation. This unspent transaction output is only consumed when the validation of the script execution fails. 

## What will change in the new Babbage era
Before the Babbage era there where some restrictions on how and what outputs could be used as collateral inputs. Firstly, collateral could only be consumed in its entirety, even if the normal fee for executing the script was less than the value of the collateral given. Secondly, outputs that contained native assets where not able to serve as collateral inputs. In the new Babbage era users will be able to specify exactly how an output that serves as a collateral input in a transaction can be spend if execution of a script fails. This gives the possibility to precisely specify how the collateral is used and creates the possibility for native assets to be also present in the collateral, though they will not serve as a payment for the fee.

## An example
As an example we will construct a transaction that will consume an output that sits at the script address of the following validator.
```haskell
{-# LANGUAGE DataKinds         		#-}
{-# LANGUAGE NoImplicitPrelude 		#-}
{-# LANGUAGE TemplateHaskell 		#-}
{-# LANGUAGE ScopedTypeVariables	#-}
{-# LANGUAGE TypeApplications    	#-}
{-# LANGUAGE ImportQualifiedPost	#-}

module Onchain where

import PlutusTx qualified
import PlutusTx.Prelude
import Plutus.V2.Ledger.Api qualified as PlutusV2
import Plutus.Script.Utils.V2.Scripts as Utils
import Ledger.Address as Addr

newtype SecretDatum = SecretDatum Integer
PlutusTx.unstableMakeIsData ''SecretDatum
newtype GuessRedeemer = GuessRedeemer Integer
PlutusTx.unstableMakeIsData ''GuessRedeemer

-- This validator always validates true
{-# INLINABLE mkValidator #-}
newtype MyCustomDatum = MyCustomDatum Integer
PlutusTx.unstableMakeIsData ''MyCustomDatum
newtype MyCustomRedeemer = MyCustomRedeemer Integer
PlutusTx.unstableMakeIsData ''MyCustomRedeemer

mkValidator :: MyCustomDatum -> MyCustomRedeemer -> PlutusV2.ScriptContext -> Bool
mkValidator _ _ _ = True

validator :: PlutusV2.Validator
validator = PlutusV2.mkValidatorScript
    $$(PlutusTx.compile [|| wrap ||])
 where
    wrap = Utils.mkUntypedValidator mkValidator
```
This validator always validates true. Note that this script is not for safe usage on mainnet since nothing hinders spending funds at this script address, only use the testnet for this example  as it serves as an example only. The above validator compiles to the following serialized `PlutusScriptV2`
```
$ cat typedAlwaysSucceeds.plutus
{
    "type": "PlutusScriptV2",
    "description": "",
    "cborHex": "5907b65907b301000032323232323232323232323232323322323232323223223223232533532323201e3333573466e1cd55cea80224000466442466002006004646464646464646464646464646666ae68cdc39aab9d500c480008cccccccccccc88888888888848cccccccccccc00403403002c02802402001c01801401000c008cd4064068d5d0a80619a80c80d1aba1500b33501901b35742a014666aa03aeb94070d5d0a804999aa80ebae501c35742a01066a0320486ae85401cccd54074095d69aba150063232323333573466e1cd55cea801240004664424660020060046464646666ae68cdc39aab9d5002480008cc8848cc00400c008cd40bdd69aba150023030357426ae8940088c98c80c8cd5ce01981901809aab9e5001137540026ae854008c8c8c8cccd5cd19b8735573aa004900011991091980080180119a817bad35742a00460606ae84d5d1280111931901919ab9c033032030135573ca00226ea8004d5d09aba2500223263202e33573805e05c05826aae7940044dd50009aba1500533501975c6ae854010ccd540740848004d5d0a801999aa80ebae200135742a00460466ae84d5d1280111931901519ab9c02b02a028135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d55cf280089baa00135742a00860266ae84d5d1280211931900e19ab9c01d01c01a3333573466e1cd55cea802a400046eb4d5d09aab9e500623263201b3357380380360326666ae68cdc39aab9d5006480008dd69aba135573ca00e464c6403466ae7006c06806040644c98c8064cd5ce24810350543500019135573ca00226ea80044dd500089baa0011232230023758002640026aa02a446666aae7c004940288cd4024c010d5d080118019aba2002014232323333573466e1cd55cea8012400046644246600200600460186ae854008c014d5d09aba2500223263201433573802a02802426aae7940044dd50009191919191999ab9a3370e6aae75401120002333322221233330010050040030023232323333573466e1cd55cea80124000466442466002006004602a6ae854008cd403c050d5d09aba2500223263201933573803403202e26aae7940044dd50009aba150043335500875ca00e6ae85400cc8c8c8cccd5cd19b875001480108c84888c008010d5d09aab9e500323333573466e1d4009200223212223001004375c6ae84d55cf280211999ab9a3370ea00690001091100191931900d99ab9c01c01b019018017135573aa00226ea8004d5d0a80119a805bae357426ae8940088c98c8054cd5ce00b00a80989aba25001135744a00226aae7940044dd5000899aa800bae75a224464460046eac004c8004d5404888c8cccd55cf80112804119a8039991091980080180118031aab9d5002300535573ca00460086ae8800c0484d5d080088910010910911980080200189119191999ab9a3370ea0029000119091180100198029aba135573ca00646666ae68cdc3a801240044244002464c6402066ae700440400380344d55cea80089baa001232323333573466e1d400520062321222230040053007357426aae79400c8cccd5cd19b875002480108c848888c008014c024d5d09aab9e500423333573466e1d400d20022321222230010053007357426aae7940148cccd5cd19b875004480008c848888c00c014dd71aba135573ca00c464c6402066ae7004404003803403002c4d55cea80089baa001232323333573466e1cd55cea80124000466442466002006004600a6ae854008dd69aba135744a004464c6401866ae700340300284d55cf280089baa0012323333573466e1cd55cea800a400046eb8d5d09aab9e500223263200a33573801601401026ea80048c8c8c8c8c8cccd5cd19b8750014803084888888800c8cccd5cd19b875002480288488888880108cccd5cd19b875003480208cc8848888888cc004024020dd71aba15005375a6ae84d5d1280291999ab9a3370ea00890031199109111111198010048041bae35742a00e6eb8d5d09aba2500723333573466e1d40152004233221222222233006009008300c35742a0126eb8d5d09aba2500923333573466e1d40192002232122222223007008300d357426aae79402c8cccd5cd19b875007480008c848888888c014020c038d5d09aab9e500c23263201333573802802602202001e01c01a01801626aae7540104d55cf280189aab9e5002135573ca00226ea80048c8c8c8c8cccd5cd19b875001480088ccc888488ccc00401401000cdd69aba15004375a6ae85400cdd69aba135744a00646666ae68cdc3a80124000464244600400660106ae84d55cf280311931900619ab9c00d00c00a009135573aa00626ae8940044d55cf280089baa001232323333573466e1d400520022321223001003375c6ae84d55cf280191999ab9a3370ea004900011909118010019bae357426aae7940108c98c8024cd5ce00500480380309aab9d50011375400224464646666ae68cdc3a800a40084244400246666ae68cdc3a8012400446424446006008600c6ae84d55cf280211999ab9a3370ea00690001091100111931900519ab9c00b00a008007006135573aa00226ea80048c8cccd5cd19b87500148008801c8cccd5cd19b8750024800084880048c98c8018cd5ce00380300200189aab9d37540029309000a490350543100122002112323001001223300330020020011"
}
```
The address of this script can be found via the following the command
```
$ cardano-cli address build --payment-script-file typedAlwaysSucceeds.plutus --testnet-magic 9 --out-file typedAlwaysSucceeds.addr
```
Note that this example is executed on a private testnet with magic number `9`, on the Cardano testnet this is `1097911063`. To view the current outputs located at the script address we use 
```
$ cardano-cli query utxo --address $(cat typedAlwaysSucceeds.addr) --testnet-magic 9
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
2729cafb2e474c4ddfaae54e4654a37ca52961b3c9d043c1c91f73ee7a3f3418     1        20000000 lovelace + TxOutDatumInline ReferenceTxInsScriptsInlineDatumsInBabbageEra (ScriptDataConstructor 0 [ScriptDataNumber 42])
```
This output has an inline datum and contains a reference script that corresponds to the script address at hand.
## Constructing a transaction with a failing script execution
To showcase how we can specify the spending of a collateral input we construct a transaction that does not meet the requirements to successfully validate this always validating script. We do this by explicitly wrongfully stating in the transaction how many abstract execution units are needed for execution of the script. This amount is characterized by two values, `exBudgetCPU` and `exBudgetMemory` which we represent with a tuple of integers in the transaction. The amount will be less than the true value needed for correct validation and thus execution will fail. This results in the collateral being taken. The output we will use as a collateral is the output 
```
$ cardano-cli query utxo --address $(cat key1.addr) --testnet-magic 9
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
932704ece8010bb6603c2b43fba97390f4c88dac0283f26fdb62fdf2ed22abfa     1        100000000 lovelace + 1 4844bf6c04c26579aebb72d416167f5cce8446ae022a4e712130bc94.5072696d65436f756e746572 + TxOutDatumNone
```
Note that this transaction output contains a native asset. To explicitly mention the available abstract execution units we use the `build-raw` client command which allows us to specify more details about the transaction and can allow us to force the creation of a failing transaction with the flag `--script-invalid`  flag. 
```
$ cardano-cli transaction build-raw \
--babbage-era \
--tx-in 2729cafb2e474c4ddfaae54e4654a37ca52961b3c9d043c1c91f73ee7a3f3418#1 \
--spending-tx-in-reference 2729cafb2e474c4ddfaae54e4654a37ca52961b3c9d043c1c91f73ee7a3f3418#1 \
--spending-plutus-script-v2 \
--spending-reference-tx-in-inline-datum-present \
--spending-reference-tx-in-redeemer-file myRedeemer.json \
--spending-reference-tx-in-execution-units "(1, 1)" \
--tx-in-collateral 932704ece8010bb6603c2b43fba97390f4c88dac0283f26fdb62fdf2ed22abfa#1 \
--tx-out $(cat key1.addr)+19700000 \
--protocol-params-file protocol.json \
--fee 300000 \
--tx-out-return-collateral $(cat key1.addr)+99550000+"1 4844bf6c04c26579aebb72d416167f5cce8446ae022a4e712130bc94.5072696d65436f756e746572" \
--tx-total-collateral 450000 \
--script-invalid \
--out-file tx.body
```
While using the `build-raw` command one has to explicitly calculate the transaction fee's themselves. For ease of the construction we chose to pay more than the minimum fee. As an overview we state here the flow of value in both cases of successful execution and script failure.

|  | Value consumed  | Value output| Fee
| :------------ |:---------------:| -----:| ------:
| succesful execution | input at `typedAlwaysSucceeds.addr` with `20000000 lovelace` | `19700000 lovelace` at the `key1.addr` | `300000 lovelace`
| script failure     | collateral input at `key1.addr` with `100000000 + 1 native asset`        |   `99550000 lovelace + 1 native asset` at the `key1.addr` |  `45000`

With the current protocol parameter the minimum collateral is `1.5` times the transaction fee. 

Now to get the transaction id for later identification we use
```
$ cardano-cli transaction txid --tx-body-file tx.body 
56e82d6887ca6f0352e931cc0bebb34ae5d568522c767c705bbb16171a69229b
```
This will be part of the new output identifier. Then we sign the `tx.body` with the witness associated to the output we are spending, in our case this is just a key. We use
```
$ cardano-cli transaction sign --tx-body-file tx.body --signing-key-file key1.skey --testnet-magic 9 --out-file tx.signed
```
To submit the signed transaction we use
```
$ cardano-cli transaction submit --testnet-magic 9 --tx-file tx.signed 
Transaction successfully submitted.
```
We verify that due to the script failure the collateral was consumed and the residue `99550000+"1 4844bf6c04c26579aebb72d416167f5cce8446ae022a4e712130bc94.5072696d65436f756e746572"` is returned to the `key1.addr` as expected.
```
cardano-cli query utxo --address $(cat key1.addr) --testnet-magic 9
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
56e82d6887ca6f0352e931cc0bebb34ae5d568522c767c705bbb16171a69229b     1        99550000 lovelace + 1 4844bf6c04c26579aebb72d416167f5cce8446ae022a4e712130bc94.5072696d65436f756e746572 + TxOutDatumNone
```
