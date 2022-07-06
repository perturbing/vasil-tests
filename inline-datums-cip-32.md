# How to use inline datums
In this tutorial, we will discuss the usage of the new functionality of inline datums [(1)](https://cips.cardano.org/cips/cip32/). 
## What are datums?
Functionally, datums are pieces of data that are attached to outputs. In this way, data can be represented and handled on the Cardano blockchain. Before the Vasil hard fork in the Alonzo era, there were two ways of attaching these datums, namely with a hash or embedding it. In the first case, a commitment was made to what the datum is without publicly revealing the value of the datum, by only adding the hash of the datum to the output. The second case does the opposite, it reveals the value on the blockchain so that anyone can see it. The location of the embedded datum is in the transaction that create the output. To retrieve it, one must use a db-sync instance to retrieve the transaction body.

## What will change in the new Babbage era?
In practice, the construction of a transaction that consumes an output with a datum requires the spender to attach the full datum to the transaction, even if the datum is already embedded in the output. This is not optimal since it requires the user to search for the datum. Furthermore, this way of providing the data twice is redundant and takes up unnecessary space in a transaction if the datum is already know on the blockchain (the current transaction size limit is 16384 bytes). With the upcoming hardfork, there will be an additional method of adding datums to outputs that is called 'inline' datums. This way of attaching datums to outputs lets users who consume this output specify that the data is already at the output so that they do not need to provide it to their transactions. Below we will have a look at an example.

## An example
To showcase this new feature, we will use the trivial validator given by
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

-- This validator checks if the redeemer guess integer corresponds to the datum secret integer.
{-# INLINABLE mkValidator #-}
mkValidator :: SecretDatum -> GuessRedeemer -> PlutusV2.ScriptContext -> Bool
mkValidator (SecretDatum secret) (GuessRedeemer guess) _ = secret == guess

validator :: PlutusV2.Validator
validator = PlutusV2.mkValidatorScript
    $$(PlutusTx.compile [|| wrap ||])
 where
    wrap = Utils.mkUntypedValidator mkValidator

```
This is a validator that check if the provided integer in the redeemer is the same as the integer in the datum. Note that this datum is not for safe use on mainnet since guessing the integer is not that hard, use the testnet for this example. The above validator compiles to the following serialized`PlutusScriptV2`
```
$ cat typedGuessGame.plutus
{
    "type": "PlutusScriptV2",
    "description": "",
    "cborHex": "5907c35907c001000032332232323232323232323232323233223232323232232232232325335323232333573466e1c00c00807c078cccd5cd19b8735573aa0089000119910919800801801191919191919191919191919191999ab9a3370e6aae754031200023333333333332222222222221233333333333300100d00c00b00a00900800700600500400300233501901a35742a01866a0320346ae85402ccd406406cd5d0a805199aa80ebae501c35742a012666aa03aeb94070d5d0a80419a80c8121aba150073335501d02575a6ae854018c8c8c8cccd5cd19b8735573aa00490001199109198008018011919191999ab9a3370e6aae754009200023322123300100300233502f75a6ae854008c0c0d5d09aba2500223263203233573806606406026aae7940044dd50009aba150023232323333573466e1cd55cea8012400046644246600200600466a05eeb4d5d0a80118181aba135744a004464c6406466ae700cc0c80c04d55cf280089baa001357426ae8940088c98c80b8cd5ce01781701609aab9e5001137540026ae854014cd4065d71aba150043335501d021200135742a006666aa03aeb88004d5d0a80118119aba135744a004464c6405466ae700ac0a80a04d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135573ca00226ea8004d5d0a80218099aba135744a008464c6403866ae70074070068cccd5cd19b8735573aa00a900011bad357426aae7940188c98c806ccd5ce00e00d80c9999ab9a3370e6aae75401920002375a6ae84d55cf280391931900d19ab9c01b01a01810191326320193357389210350543500019135573ca00226ea80044dd500089baa0011232230023758002640026aa02c446666aae7c004940288cd4024c010d5d080118019aba2002014232323333573466e1cd55cea8012400046644246600200600460186ae854008c014d5d09aba2500223263201433573802a02802426aae7940044dd50009191919191999ab9a3370e6aae75401120002333322221233330010050040030023232323333573466e1cd55cea80124000466442466002006004602a6ae854008cd403c050d5d09aba2500223263201933573803403202e26aae7940044dd50009aba150043335500875ca00e6ae85400cc8c8c8cccd5cd19b875001480108c84888c008010d5d09aab9e500323333573466e1d4009200223212223001004375c6ae84d55cf280211999ab9a3370ea00690001091100191931900d99ab9c01c01b019018017135573aa00226ea8004d5d0a80119a805bae357426ae8940088c98c8054cd5ce00b00a80989aba25001135744a00226aae7940044dd5000899aa800bae75a224464460046eac004c8004d5404c88c8cccd55cf80112804119a8039991091980080180118031aab9d5002300535573ca00460086ae8800c0484d5d080088910010910911980080200189119191999ab9a3370ea0029000119091180100198029aba135573ca00646666ae68cdc3a801240044244002464c6402066ae700440400380344d55cea80089baa001232323333573466e1d400520062321222230040053007357426aae79400c8cccd5cd19b875002480108c848888c008014c024d5d09aab9e500423333573466e1d400d20022321222230010053007357426aae7940148cccd5cd19b875004480008c848888c00c014dd71aba135573ca00c464c6402066ae7004404003803403002c4d55cea80089baa001232323333573466e1cd55cea80124000466442466002006004600a6ae854008dd69aba135744a004464c6401866ae700340300284d55cf280089baa0012323333573466e1cd55cea800a400046eb8d5d09aab9e500223263200a33573801601401026ea80048c8c8c8c8c8cccd5cd19b8750014803084888888800c8cccd5cd19b875002480288488888880108cccd5cd19b875003480208cc8848888888cc004024020dd71aba15005375a6ae84d5d1280291999ab9a3370ea00890031199109111111198010048041bae35742a00e6eb8d5d09aba2500723333573466e1d40152004233221222222233006009008300c35742a0126eb8d5d09aba2500923333573466e1d40192002232122222223007008300d357426aae79402c8cccd5cd19b875007480008c848888888c014020c038d5d09aab9e500c23263201333573802802602202001e01c01a01801626aae7540104d55cf280189aab9e5002135573ca00226ea80048c8c8c8c8cccd5cd19b875001480088ccc888488ccc00401401000cdd69aba15004375a6ae85400cdd69aba135744a00646666ae68cdc3a80124000464244600400660106ae84d55cf280311931900619ab9c00d00c00a009135573aa00626ae8940044d55cf280089baa001232323333573466e1d400520022321223001003375c6ae84d55cf280191999ab9a3370ea004900011909118010019bae357426aae7940108c98c8024cd5ce00500480380309aab9d50011375400224464646666ae68cdc3a800a40084244400246666ae68cdc3a8012400446424446006008600c6ae84d55cf280211999ab9a3370ea00690001091100111931900519ab9c00b00a008007006135573aa00226ea80048c8cccd5cd19b8750014800880208cccd5cd19b8750024800080208c98c8018cd5ce00380300200189aab9d37540029309000a4810350543100122002122001112323001001223300330020020011"
}
```
The address of this script can be found via the following the command
```
$ cardano-cli address build --payment-script-file typedGuessGame.plutus --testnet-magic 9 --out-file typedGuessGame.addr
```
Note that this example is executed on a private testnet with magic number `9`, on the Cardano testnet this is `1097911063`. To view the current outputs located at the script address, we use, 
```
$ cardano-cli query utxo --address $(cat typedGuessGame.addr) --testnet-magic 9
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
```
Currently, the address has no outputs located at it.
## Creating an output at the script address
Now we will create an output at the above script address with the chosen secret number 42. This datum is represented as the following encoded JSON file,
```
$ cat secret.json
{
	"constructor": 0,
	"fields": [{
		"int": 42
	}]
}
```
To construct the transaction that creates this output with an inline datum, we use the `cardano-cli` with flag `--tx-out-inline-datum-file`
```
$ cardano-cli transaction build --babbage-era --testnet-magic 9 \
--tx-in 932704ece8010bb6603c2b43fba97390f4c88dac0283f26fdb62fdf2ed22abfa#0 \
--tx-out $(cat typedGuessGame.addr)+50000000 \
--tx-out-inline-datum-file secret.json \
--change-address $(cat key1.addr) \
--out-file tx.body
Estimated transaction fee: Lovelace 166513
```
To get the transaction ID for identification, we use
```
$ cardano-cli transaction txid --tx-body-file tx.body 
3db0bbe89a032ec57519f3785fcd0c70a5177e705ecfbf469494c3f412b31d22
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
$ cardano-cli query utxo --address $(cat typedGuessGame.addr) --testnet-magic 9
                            TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
3db0bbe89a032ec57519f3785fcd0c70a5177e705ecfbf469494c3f412b31d22     1        50000000 lovelace + TxOutDatumInline ReferenceTxInsScriptsInlineDatumsInBabbageEra (ScriptDataConstructor 0 [ScriptDataNumber 42])
```
Notice that at the end of our newly create output `3db.....d22#1`  the inline datum is shown. 
## Spending the output at the script address
Now since the secret number is public it is super easy to claim it, we construct a transaction to spend the output and send the value it contains to our address. With the presence of the inline datum, we do this with the new flag `--tx-in-inline-datum-present`. Note that with multiple inputs that contain an inline datum all have to have this flag after their `--tx-in TxId#TxIx` flag, we also add the script.
```
$ cardano-cli transaction build --babbage-era --testnet-magic 9 \
--tx-in 3db0bbe89a032ec57519f3785fcd0c70a5177e705ecfbf469494c3f412b31d22#1 \
--tx-in-script-file typedGuessGame.plutus \
--tx-in-inline-datum-present \
--tx-in-redeemer-file rightGuess.json \
--tx-in-collateral f0c268356ebe7d40b76b75e5948d8418df431f2c716b1ad06f903288c36c8b67#1 \
--change-address $(cat key1.addr) \
--protocol-params-file protocol.json \
--out-file tx.body
Estimated transaction fee: Lovelace 291841
```
Here the redeemer file `rightGuess.json` is given by 
```
$ cat rightGuess.json
{
	"constructor": 0,
	"fields": [{
		"int": 42
	}]
}
```
The collateral is an output sitting at the address `key1.addr` and we output all funds via the flag `--change-address` to the same address. The protocol parameters that are used can be retrieved via the following command,
```
cardano-cli query protocol-parameters --testnet-magic 9 --out-file protocol.json
```
We now sign the transaction with `key1.skey` since it uses collateral from this key and send it.
```
$ cardano-cli transaction sign --tx-body-file tx.body --signing-key-file key1.skey --testnet-magic 9 --out-file tx.signed
$ cardano-cli transaction submit --testnet-magic 9 --tx-file tx.signed 
Transaction successfully submitted.
```
We also calculate the `TxId` to be able to identify the new output.
```
$ cardano-cli transaction  txid  --tx-body-file tx.body 
50d14e8624aada75fc2661b903ada2ef53e31c3ac63f4aedfcf96a4d375434fb
```
Lastly, we look at the address `key1.addr` to see if the transaction was successful
```
cardano-cli query utxo --address $(cat key1.addr) --testnet-magic 9
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
50d14e8624aada75fc2661b903ada2ef53e31c3ac63f4aedfcf96a4d375434fb     0        49708159 lovelace + TxOutDatumNone
```
In this example, we used an inline datum for a plutusV2 script, older plutusV1 scripts also support this. 
