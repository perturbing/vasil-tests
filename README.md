# On reference scripts and inline datums.
* * *
This small tutorial shows how to utilize the new [CIP 33](https://cips.cardano.org/cips/cip33/) and  [CIP 32](https://cips.cardano.org/cips/cip32/). We will use a basic plutusV2 script given by the validator
```
newtype MyCustomDatum = MyCustomDatum Integer
PlutusTx.unstableMakeIsData ''MyCustomDatum
newtype MyCustomRedeemer = MyCustomRedeemer Integer
PlutusTx.unstableMakeIsData ''MyCustomRedeemer

mkValidator :: MyCustomDatum -> MyCustomRedeemer -> PlutusV2.ScriptContext -> Bool
mkValidator _ _ _ = True

```
which will always succeeds. This compiles to the following serialized script,
```
{
    "type": "PlutusScriptV2",
    "description": "",
    "cborHex": "5907b65907b301000032323232323232323232323232323322323232323223223223232533532323201e3333573466e1cd55cea80224000466442466002006004646464646464646464646464646666ae68cdc39aab9d500c480008cccccccccccc88888888888848cccccccccccc00403403002c02802402001c01801401000c008cd4064068d5d0a80619a80c80d1aba1500b33501901b35742a014666aa03aeb94070d5d0a804999aa80ebae501c35742a01066a0320486ae85401cccd54074095d69aba150063232323333573466e1cd55cea801240004664424660020060046464646666ae68cdc39aab9d5002480008cc8848cc00400c008cd40bdd69aba150023030357426ae8940088c98c80c8cd5ce01981901809aab9e5001137540026ae854008c8c8c8cccd5cd19b8735573aa004900011991091980080180119a817bad35742a00460606ae84d5d1280111931901919ab9c033032030135573ca00226ea8004d5d09aba2500223263202e33573805e05c05826aae7940044dd50009aba1500533501975c6ae854010ccd540740848004d5d0a801999aa80ebae200135742a00460466ae84d5d1280111931901519ab9c02b02a028135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d55cf280089baa00135742a00860266ae84d5d1280211931900e19ab9c01d01c01a3333573466e1cd55cea802a400046eb4d5d09aab9e500623263201b3357380380360326666ae68cdc39aab9d5006480008dd69aba135573ca00e464c6403466ae7006c06806040644c98c8064cd5ce24810350543500019135573ca00226ea80044dd500089baa0011232230023758002640026aa02a446666aae7c004940288cd4024c010d5d080118019aba2002014232323333573466e1cd55cea8012400046644246600200600460186ae854008c014d5d09aba2500223263201433573802a02802426aae7940044dd50009191919191999ab9a3370e6aae75401120002333322221233330010050040030023232323333573466e1cd55cea80124000466442466002006004602a6ae854008cd403c050d5d09aba2500223263201933573803403202e26aae7940044dd50009aba150043335500875ca00e6ae85400cc8c8c8cccd5cd19b875001480108c84888c008010d5d09aab9e500323333573466e1d4009200223212223001004375c6ae84d55cf280211999ab9a3370ea00690001091100191931900d99ab9c01c01b019018017135573aa00226ea8004d5d0a80119a805bae357426ae8940088c98c8054cd5ce00b00a80989aba25001135744a00226aae7940044dd5000899aa800bae75a224464460046eac004c8004d5404888c8cccd55cf80112804119a8039991091980080180118031aab9d5002300535573ca00460086ae8800c0484d5d080088910010910911980080200189119191999ab9a3370ea0029000119091180100198029aba135573ca00646666ae68cdc3a801240044244002464c6402066ae700440400380344d55cea80089baa001232323333573466e1d400520062321222230040053007357426aae79400c8cccd5cd19b875002480108c848888c008014c024d5d09aab9e500423333573466e1d400d20022321222230010053007357426aae7940148cccd5cd19b875004480008c848888c00c014dd71aba135573ca00c464c6402066ae7004404003803403002c4d55cea80089baa001232323333573466e1cd55cea80124000466442466002006004600a6ae854008dd69aba135744a004464c6401866ae700340300284d55cf280089baa0012323333573466e1cd55cea800a400046eb8d5d09aab9e500223263200a33573801601401026ea80048c8c8c8c8c8cccd5cd19b8750014803084888888800c8cccd5cd19b875002480288488888880108cccd5cd19b875003480208cc8848888888cc004024020dd71aba15005375a6ae84d5d1280291999ab9a3370ea00890031199109111111198010048041bae35742a00e6eb8d5d09aba2500723333573466e1d40152004233221222222233006009008300c35742a0126eb8d5d09aba2500923333573466e1d40192002232122222223007008300d357426aae79402c8cccd5cd19b875007480008c848888888c014020c038d5d09aab9e500c23263201333573802802602202001e01c01a01801626aae7540104d55cf280189aab9e5002135573ca00226ea80048c8c8c8c8cccd5cd19b875001480088ccc888488ccc00401401000cdd69aba15004375a6ae85400cdd69aba135744a00646666ae68cdc3a80124000464244600400660106ae84d55cf280311931900619ab9c00d00c00a009135573aa00626ae8940044d55cf280089baa001232323333573466e1d400520022321223001003375c6ae84d55cf280191999ab9a3370ea004900011909118010019bae357426aae7940108c98c8024cd5ce00500480380309aab9d50011375400224464646666ae68cdc3a800a40084244400246666ae68cdc3a8012400446424446006008600c6ae84d55cf280211999ab9a3370ea00690001091100111931900519ab9c00b00a008007006135573aa00226ea80048c8cccd5cd19b87500148008801c8cccd5cd19b8750024800084880048c98c8018cd5ce00380300200189aab9d37540029309000a490350543100122002112323001001223300330020020011"
}

```
To showcast the new CIP's we will first create an output that contains the script which we can reference. Then we will create an output at the script address which we want to claim. Lastly we will show how we can claim the latter output without attaching a script to the transaction but by referencing the former output without consuming it. At the end there is some discussion.

## Creating an output at a key witnessed address to reference
First we create an output at a normal key witnessed address to which we also attach the above script to this address for later referencing in transactions. To create such an out put we use 
```
$ cardano-cli transaction build --babbage-era --testnet-magic 9 \
--tx-in 832015d4218db60e6bd9e38bbb73dc10f92f9c8d83b191e208bddb444aa103e2#0 \
--tx-out $(cat ../../../keys/key1.addr)+15000000 \
--tx-out-reference-script-file ../../../typedAlwaysSucceeds/typedAlwaysSucceeds.plutus \
--change-address $(cat ../../../keys/key1.addr) \
--out-file tx.body

```
Creating a witness for this transaction,
```
$ cardano-cli transaction witness --tx-body-file tx.body --signing-key-file ../../../keys/key1.skey --testnet-magic 9 --out-file txWitness
```
Assamble witness and tx,
```
$ cardano-cli transaction assemble --tx-body-file tx.body --witness-file txWitness --out-file tx.signed
```
Send it and get the txid,
```
$ cardano-cli transaction submit --testnet-magic 9 --tx-file tx.signed && cardano-cli transaction txid --tx-body-file tx.body
Transaction successfully submitted
69163b01cb8f2dd0e710419340148d518c2160a7f587c7187e8ad2d3e74f6c41
```
To validate that our new output has the plutus script we use
```
$ cardano-cli query utxo --tx-in 69163b01cb8f2dd0e710419340148d518c2160a7f587c7187e8ad2d3e74f6c41#1 --testnet-magic 9 --out-file test && cat test
{
    "69163b01cb8f2dd0e710419340148d518c2160a7f587c7187e8ad2d3e74f6c41#1": {
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
We see that the `"referenceScript"` entry cointains the plutus script. Also note that we see the new `"inlineDatum"` field here as well.

## Creating an output at the script address to claim
This is done via the below command
```
cardano-cli transaction build --babbage-era --testnet-magic 9 \
--tx-in 69163b01cb8f2dd0e710419340148d518c2160a7f587c7187e8ad2d3e74f6c41#0 \
--tx-out $(cat ../../../typedAlwaysSucceeds/typedAlwaysSucceeds.addr)+15000000 \
--tx-out-inline-datum-file ../../../typedAlwaysSucceeds/myDatum.json  \
--change-address $(cat ../../../keys/key1.addr) \
--out-file tx.body
```
Here we used an arbitrary datum of the format 
```
{"constructor":0,"fields":[{"int":42}]}
```
Creating a witness for this transaction,
```
$ cardano-cli transaction witness --tx-body-file tx.body --signing-key-file ../../../keys/key1.skey --testnet-magic 9 --out-file txWitness
```
Assamble witness and tx,
```
$ cardano-cli transaction assemble --tx-body-file tx.body --witness-file txWitness --out-file tx.signed
```
Send it and get the txid,
```
$ cardano-cli transaction submit --testnet-magic 9 --tx-file tx.signed && cardano-cli transaction txid --tx-body-file tx.body
Transaction successfully submitted.
2f95f685ebaa867b9fd59a646b85679716ae14b3505dbbf560a68e179606fad4
```
To verify our output we check again with
```
cardano-cli query utxo --tx-in 2f95f685ebaa867b9fd59a646b85679716ae14b3505dbbf560a68e179606fad4#1 --testnet-magic 9 --out-file test && cat test
{
    "2f95f685ebaa867b9fd59a646b85679716ae14b3505dbbf560a68e179606fad4#1": {
        "address": "addr_test1wrtzze7eefhgzaksqu2qaechgykl92ht9d7ng3cyc9ydn7cr87gc2",
        "datum": null,
        "inlineDatum": {
            "constructor": 0,
            "fields": [
                {
                    "int": 42
                }
            ]
        },
        "inlineDatumhash": "fcaa61fb85676101d9e3398a484674e71c45c3fd41b492682f3b0054f4cf3273",
        "referenceScript": null,
        "value": {
            "lovelace": 15000000
        }
    }
}

```
We indeed see that the `"inlineDatum"` field contains our datum.

## Claiming the output at the script address
To claim the output we will reference the first output that has the plutusV2 script attached. We construct this via the following command
```
cardano-cli transaction build --babbage-era --testnet-magic 9 \
--tx-in 2f95f685ebaa867b9fd59a646b85679716ae14b3505dbbf560a68e179606fad4#1 \
--tx-in-collateral 661bd89947505647071afd05b9a0b9d60b8b0d5b8fcafd828539ddca3a37aa91#0 \
--tx-in-reference 69163b01cb8f2dd0e710419340148d518c2160a7f587c7187e8ad2d3e74f6c41#1 \
--plutus-script-v2 \
--reference-tx-in-inline-datum-present \
--reference-tx-in-redeemer-file ../../../typedAlwaysSucceeds/myRedeemer.json \
--change-address $(cat ../../../keys/key1.addr) \
--protocol-params-file ../../protocol.json \
--out-file tx.body
```
Here we used an arbitrary redeemer of the format 
```
{"constructor":0,"fields":[{"int":42}]}
```
Creating a witness for this transaction,
```
$ cardano-cli transaction witness --tx-body-file tx.body --signing-key-file ../../../keys/key1.skey --testnet-magic 9 --out-file txWitness
```
Assamble witness and tx,
```
$ cardano-cli transaction assemble --tx-body-file tx.body --witness-file txWitness --out-file tx.signed
```
Send it and get the txid,
```
$ cardano-cli transaction submit --testnet-magic 9 --tx-file tx.signed && cardano-cli transaction txid --tx-body-file tx.body
Transaction successfully submitted.
19829ec330e55172f21fce6821345047fb4228229a8b5870678470bd3500b5a2
```
Now to we check that our reference input is still there and our claimed funds
```
cardano-cli query utxo --address $(cat ../../../keys/key1.addr) --testnet-magic 9
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
19829ec330e55172f21fce6821345047fb4228229a8b5870678470bd3500b5a2     0        14786555 lovelace + TxOutDatumNone
69163b01cb8f2dd0e710419340148d518c2160a7f587c7187e8ad2d3e74f6c41     1        15000000 lovelace + TxOutDatumNone
```
## Discusion
Note that in the above transaction the referenced script was still unspent and stayed this way, it was not consumed. Outputs that are already spent are not able to reference scripts anymore.
