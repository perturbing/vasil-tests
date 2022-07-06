# How to reference scripts
In this tutorial, we will discuss the usage of the new functionality of referencing scripts [(1)](https://cips.cardano.org/cips/cip33/). 
## What are scripts?
Functionally, scripts are witnesses that validate if an action is permitted on the Cardano blockchain. It takes some input and returns with either true or false. The Cardano Blockchain currently has two types of scripts.

- Simple scripts
- Plutus scripts

Simple scripts are explained in detail here [(2)](https://github.com/input-output-hk/cardano-node/blob/master/doc/reference/simple-scripts.md) and plutus scripts are explained more in detail here [(3)](https://docs.cardano.org/plutus/Plutus-validator-scripts). Besides locking funds, scripts also determine if a transaction can forge or burn native assets. Currently, **every** transaction that utilizes a script must also provide the script corresponding to the action that they witness. The relation between scripts and their action is via the hash of the script. For minting or burning assets, this relation is embedded in the policy ID of the assets. Scripts that lock transaction outputs at an address, this relation is embedded in the address.

## What will change in the new Babbage?
With the introduction of the new Babbage era, it is now possible to reference scripts from previously create output. This means that one can create an output that contains a script and later reference this transaction output in another transaction that uses that script to validate some action. The power of this is that this last transaction does not consume the output containing the script, but only references the script. In this way, multiple transactions can reference the script without appending the script again to the blockchain. This saves on fee's and on space on the ledger.

## An example
To showcase this new feature, we will use the trivial validator. We will first create an output at its script address and an output at our key witnessed address that holds the script. Then we are going to spend the output at the script address with a transaction that references the script at the key witnessed address without consuming it. The used validator for this example is,
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

newtype MyCustomDatum = MyCustomDatum Integer
PlutusTx.unstableMakeIsData ''MyCustomDatum
newtype MyCustomRedeemer = MyCustomRedeemer Integer
PlutusTx.unstableMakeIsData ''MyCustomRedeemer

-- This validator always validates true
{-# INLINABLE mkValidator #-}
mkValidator :: MyCustomDatum -> MyCustomRedeemer -> PlutusV2.ScriptContext -> Bool
mkValidator _ _ _ = True

validator :: PlutusV2.Validator
validator = PlutusV2.mkValidatorScript
    $$(PlutusTx.compile [|| wrap ||])
 where
    wrap = Utils.mkUntypedValidator mkValidator
```
This is a validator, always validates true. Note that this script is not for safe use on mainnet since nothing hinders spending funds at this script address, only use the testnet for this example. The above validator compiles to the following serialized `PlutusScriptV2`
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
Note that this example is executed on a private testnet with magic number `9`, on the Cardano testnet this is `1097911063`. To view the current outputs located at the script address, we use, 
```
$ cardano-cli query utxo --address $(cat typedAlwaysSucceeds.addr) --testnet-magic 9
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
```
Currently, the address has no outputs located at it.
## Creating the output at the script address
Since the validator does not validate on the conditions of what the datum or redeemer will be, we will use the datum and redeemer given by
```
$ cat myDatum.json
{
	"constructor": 0,
	"fields": [{
		"int": 42
	}]
}
```
To create the first output, we build the transaction with the following command
```
$ cardano-cli transaction build --babbage-era --testnet-magic 9 \
--tx-in 3db0bbe89a032ec57519f3785fcd0c70a5177e705ecfbf469494c3f412b31d22#0 \
--tx-out $(cat typedAlwaysSucceeds.addr)+50000000 \
--tx-out-datum-hash-file myDatum.json \
--change-address $(cat key1.addr) \
--out-file tx.body
Estimated transaction fee: Lovelace 167349
```
To get the transaction ID for identification, we use
```
$ cardano-cli transaction txid --tx-body-file tx.body 
93724d399b58a70385ad7ce7ba6471b7b4fcb691a28266cd7700d902e0a500c5
```
This will be part of the new output identifier. Then we sign the `tx.body` with the witness associated to the output we are spending, in our case this is just a key. We use,
```
$ cardano-cli transaction sign --tx-body-file tx.body --signing-key-file key1.skey --testnet-magic 9 --out-file tx.signed
```
To submit the signed transaction, we use,
```
$ cardano-cli transaction submit --testnet-magic 9 --tx-file tx.signed 
Transaction successfully submitted.
```
We verify that the output was created at the script address
```
$ cardano-cli query utxo --address $(cat typedAlwaysSucceeds.addr) --testnet-magic 9
                            TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
93724d399b58a70385ad7ce7ba6471b7b4fcb691a28266cd7700d902e0a500c5     1        50000000 lovelace + TxOutDatumHash ScriptDataInBabbageEra "fcaa61fb85676101d9e3398a484674e71c45c3fd41b492682f3b0054f4cf3273"
```
## Creating the output at the key witnessed address
Now we will create an output that contains a script so that we can later reference it. We build this transaction with,
```
$ cardano-cli transaction build --babbage-era --testnet-magic 9 \
--tx-in 93724d399b58a70385ad7ce7ba6471b7b4fcb691a28266cd7700d902e0a500c5#0 \
--tx-out $(cat key1.addr)+15000000 \
--tx-out-reference-script-file typedAlwaysSucceeds.plutus \
--change-address $(cat key1.addr) \
--out-file tx.body
Estimated transaction fee: Lovelace 253061
```
To get the transaction ID for identification, we use
```
$ cardano-cli transaction txid --tx-body-file tx.body 
d73a722b223431e5496077e7c432a63b6159a36f0aa41d3bf12f7beb1abd8bfb
```
This will be part of the new output identifier. Then we sign the `tx.body` with the witness associated to the output we are spending, in our case this is just a key. We use,
```
$ cardano-cli transaction sign --tx-body-file tx.body --signing-key-file key1.skey --testnet-magic 9 --out-file tx.signed
```
To submit the signed transaction, we use
```
$ cardano-cli transaction submit --testnet-magic 9 --tx-file tx.signed 
Transaction successfully submitted.
```
We verify that the output was created at the key witnessed address
```
$ cardano-cli query utxo --address $(cat key1.addr) --testnet-magic 9
                            TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
d73a722b223431e5496077e7c432a63b6159a36f0aa41d3bf12f7beb1abd8bfb     0        99768434346 lovelace + TxOutDatumNone
d73a722b223431e5496077e7c432a63b6159a36f0aa41d3bf12f7beb1abd8bfb     1        15000000 lovelace + TxOutDatumNone
```
If we take a closer look at the output `TxIx 0` we see the script attached to the output.
```
$ cardano-cli query utxo --tx-in d73a722b223431e5496077e7c432a63b6159a36f0aa41d3bf12f7beb1abd8bfb#1 --testnet-magic 9 --out-file test && cat test
{
    "d73a722b223431e5496077e7c432a63b6159a36f0aa41d3bf12f7beb1abd8bfb#1": {
        "address": "addr_test1vqdslrftd4z8ftfc6ndealutkpl5lk5ksmuphhakn4xjfxgls3xaz",
        "datum": null,
        "datumhash": null,
        "inlineDatum": null,
        "referenceScript": {
            "script": {
                "cborHex": "5907b65907b301000032323232323232323232323232323322323232323223223223232533532323201e3333573466e1cd55cea80224000466442466002006004646464646464646464646464646666ae68cdc39aab9d500c480008cccccccccccc88888888888848cccccccccccc00403403002c02802402001c01801401000c008cd4064068d5d0a80619a80c80d1aba1500b33501901b35742a014666aa03aeb94070d5d0a804999aa80ebae501c35742a01066a0320486ae85401cccd54074095d69aba150063232323333573466e1cd55cea801240004664424660020060046464646666ae68cdc39aab9d5002480008cc8848cc00400c008cd40bdd69aba150023030357426ae8940088c98c80c8cd5ce01981901809aab9e5001137540026ae854008c8c8c8cccd5cd19b8735573aa004900011991091980080180119a817bad35742a00460606ae84d5d1280111931901919ab9c033032030135573ca00226ea8004d5d09aba2500223263202e33573805e05c05826aae7940044dd50009aba1500533501975c6ae854010ccd540740848004d5d0a801999aa80ebae200135742a00460466ae84d5d1280111931901519ab9c02b02a028135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d55cf280089baa00135742a00860266ae84d5d1280211931900e19ab9c01d01c01a3333573466e1cd55cea802a400046eb4d5d09aab9e500623263201b3357380380360326666ae68cdc39aab9d5006480008dd69aba135573ca00e464c6403466ae7006c06806040644c98c8064cd5ce24810350543500019135573ca00226ea80044dd500089baa0011232230023758002640026aa02a446666aae7c004940288cd4024c010d5d080118019aba2002014232323333573466e1cd55cea8012400046644246600200600460186ae854008c014d5d09aba2500223263201433573802a02802426aae7940044dd50009191919191999ab9a3370e6aae75401120002333322221233330010050040030023232323333573466e1cd55cea80124000466442466002006004602a6ae854008cd403c050d5d09aba2500223263201933573803403202e26aae7940044dd50009aba150043335500875ca00e6ae85400cc8c8c8cccd5cd19b875001480108c84888c008010d5d09aab9e500323333573466e1d4009200223212223001004375c6ae84d55cf280211999ab9a3370ea00690001091100191931900d99ab9c01c01b019018017135573aa00226ea8004d5d0a80119a805bae357426ae8940088c98c8054cd5ce00b00a80989aba25001135744a00226aae7940044dd5000899aa800bae75a224464460046eac004c8004d5404888c8cccd55cf80112804119a8039991091980080180118031aab9d5002300535573ca00460086ae8800c0484d5d080088910010910911980080200189119191999ab9a3370ea0029000119091180100198029aba135573ca00646666ae68cdc3a801240044244002464c6402066ae700440400380344d55cea80089baa001232323333573466e1d400520062321222230040053007357426aae79400c8cccd5cd19b875002480108c848888c008014c024d5d09aab9e500423333573466e1d400d20022321222230010053007357426aae7940148cccd5cd19b875004480008c848888c00c014dd71aba135573ca00c464c6402066ae7004404003803403002c4d55cea80089baa001232323333573466e1cd55cea80124000466442466002006004600a6ae854008dd69aba135744a004464c6401866ae700340300284d55cf280089baa0012323333573466e1cd55cea800a400046eb8d5d09aab9e500223263200a33573801601401026ea80048c8c8c8c8c8cccd5cd19b8750014803084888888800c8cccd5cd19b875002480288488888880108cccd5cd19b875003480208cc8848888888cc004024020dd71aba15005375a6ae84d5d1280291999ab9a3370ea00890031199109111111198010048041bae35742a00e6eb8d5d09aba2500723333573466e1d40152004233221222222233006009008300c35742a0126eb8d5d09aba2500923333573466e1d40192002232122222223007008300d357426aae79402c8cccd5cd19b875007480008c848888888c014020c038d5d09aab9e500c23263201333573802802602202001e01c01a01801626aae7540104d55cf280189aab9e5002135573ca00226ea80048c8c8c8c8cccd5cd19b875001480088ccc888488ccc00401401000cdd69aba15004375a6ae85400cdd69aba135744a00646666ae68cdc3a80124000464244600400660106ae84d55cf280311931900619ab9c00d00c00a009135573aa00626ae8940044d55cf280089baa001232323333573466e1d400520022321223001003375c6ae84d55cf280191999ab9a3370ea004900011909118010019bae357426aae7940108c98c8024cd5ce00500480380309aab9d50011375400224464646666ae68cdc3a800a40084244400246666ae68cdc3a8012400446424446006008600c6ae84d55cf280211999ab9a3370ea00690001091100111931900519ab9c00b00a008007006135573aa00226ea80048c8cccd5cd19b87500148008801c8cccd5cd19b8750024800084880048c98c8018cd5ce00380300200189aab9d37540029309000a490350543100122002112323001001223300330020020011",
                "description": "",
                "type": "PlutusScriptV2"
            },
            "scriptLanguage": "PlutusScriptLanguage PlutusScriptV2"
        },
        "value": {
            "lovelace": 15000000
        }
    }
}
```
## Spending the output at the script address
Now we are going to spend the output at the script address while referencing the just created output with the script. We create the transaction with,
```
$ cardano-cli transaction build --babbage-era --testnet-magic 9 \
--tx-in 93724d399b58a70385ad7ce7ba6471b7b4fcb691a28266cd7700d902e0a500c5#1 \
--tx-in-collateral d73a722b223431e5496077e7c432a63b6159a36f0aa41d3bf12f7beb1abd8bfb#0 \
--spending-tx-in-reference d73a722b223431e5496077e7c432a63b6159a36f0aa41d3bf12f7beb1abd8bfb#1 \
--spending-plutus-script-v2 \
--spending-reference-tx-in-datum-file myDatum.json \
--spending-reference-tx-in-redeemer-file myRedeemer.json \
--change-address $(cat key1.addr) \
--protocol-params-file protocol.json \
--out-file tx.body
Estimated transaction fee: Lovelace 214478
```
To get the transaction ID for identification, we use
```
$ cardano-cli transaction txid --tx-body-file tx.body 
7f27988fb39cd92bcbc79dcd1717fb6805aa1400fa8c60a0ccc30a20ce534afa
```
This will be part of the new output identifier. Then we sign the `tx.body` with the witness associated to the output we are spending, in our case this is just a key. We use,
```
$ cardano-cli transaction sign --tx-body-file tx.body --signing-key-file key1.skey --testnet-magic 9 --out-file tx.signed
```
To submit the signed transaction, we use
```
$ cardano-cli transaction submit --testnet-magic 9 --tx-file tx.signed 
Transaction successfully submitted.
```
We verify that the output that had the script and the collateral at the key witnessed address are still there with the newly create output
```
$ cardano-cli query utxo --address $(cat key1.addr) --testnet-magic 9
                            TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
d73a722b223431e5496077e7c432a63b6159a36f0aa41d3bf12f7beb1abd8bfb     0        99768434346 lovelace + TxOutDatumNone
d73a722b223431e5496077e7c432a63b6159a36f0aa41d3bf12f7beb1abd8bfb     1        15000000 lovelace + TxOutDatumNone
7f27988fb39cd92bcbc79dcd1717fb6805aa1400fa8c60a0ccc30a20ce534afa     0        49785522 lovelace + TxOutDatumNone
```
