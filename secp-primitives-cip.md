# How to use the new SECP256K1 primitives
In this tutorial, we will discuss the usage of the new built-in functions of both the ECDSA and Schnorr signatures algorithm over the SECP256K1 elliptical curve in plutus core [(1)](https://cips.cardano.org/cips/cip49/).

## Cryptography background
In modern cryptography, elliptic curves play a crucial role. These curves are generally defined by the equation,

$$
y^2 = x^3 + ax + b,
$$

where $a$ and $b$ are constants. They allow for complex mathematical operations such as addition and multiplication, which can be used to create secure cryptographic systems like public-key encryption and digital signature schemes. The security of these systems is rooted in the difficulty of solving mathematical problems related to the curve, such as the discrete logarithm problem. In essence, elliptic curves are a type of mathematical equation that have specific properties that can be utilized to create encryption or signature schemes that are highly resistant to being broken.

There are various types of elliptic curves, each with distinct properties. For instance, Cardano uses the Ed25519 elliptic curve for its signatures, while Bitcoin and other cryptocurrencies use the SECP256k1 elliptic curve. The SECP256k1 curve in particular is defined by the equation,

$$ 
y^2 = x^3 + 7.
$$

The constants, $a$ and $b$, in this equation are chosen as $a=0$ and $b=7$. It's important to note that the selection of these constants plays a crucial role in determining the security of the curve and are chosen with great care.

Two of the most widely used signature schemes that make use of the SECP256k1 curve are the Schnorr signature algorithm and the Elliptic Curve Digital Signature Algorithm (ECDSA). Both of these schemes aim to provide a secure method for creating digital signatures utilizing the properties of the elliptic curve.
In these signature schemes, a private key is used to generate a digital signature for a message, and the corresponding public key is used to verify the authenticity of the signature for that specific message. 

## What does the CIP introduce?
The recent upgrade to Plutus now includes two new built-in functions for efficiently verifying both Schnorr and ECDSA signatures inside a plutus scripts. These functions are necessary for interoperability with other blockchains, as they typically use the SECP256k1 curve for signature validation, rather than the Cardano native Ed25619 curve. This allows for the creation of trustless bridges between different blockchain systems. Additionally, the Schnorr signature algorithm can easily be extended to allow for a aggregated multi party or treshold signature.

The functions are defined [here](https://github.com/input-output-hk/plutus/blob/e470420424cc8c8fb9eb140702b39c6dfeb6a443/plutus-tx/src/PlutusTx/Builtins.hs#L170-L245) with an additional note on its implementation and usage.

## Using ECDSA over SECP256k1 in plutus
In the following section we will showcase the usage of one of the functions, namely the verification function of an ECDSA signature over SECP256k1. In this example we will create a simple minting validator that allows minting if our ECDSA singature check evaluates correctly.

This simple minting validator then takes the form
```haskell
{-# INLINABLE mkVerifyEcdsaPolicy #-}
mkVerifyEcdsaPolicy :: (BI.BuiltinByteString, BI.BuiltinByteString, BI.BuiltinByteString)
                               -> PlutusV2.ScriptContext
                               -> Bool
mkVerifyEcdsaPolicy (v, m, s) _sc = BI.verifyEcdsaSecp256k1Signature v m s
```
Here in `(v, m ,s)`, the `v` stands for the verification key (also called the public key), `m` the hash of the message we signed and `s` the signature. This validator compiles to
<details>
  <summary>Script.plutus</summary>
    <code>
        {
    "type": "PlutusScriptV2",
    "description": "",
    "cborHex": "5907c35907c00100003232323233223232323232323232323322323232323222323253353322350022223335734666ed000c00800406c068c8c8c8cccd5cd19b8735573aa00690001199911091998008020018011bae35742a0066eb8d5d0a8011bae357426ae8940088c98c8070cd5ce00e80e00d09aba25001135573ca00226ea8010cccd5cd19b8735573aa0049000119910919800801801191919191919191919191919191999ab9a3370e6aae754031200023333333333332222222222221233333333333300100d00c00b00a00900800700600500400300233501401535742a01866a02802a6ae85402ccd4050058d5d0a805199aa80c3ae501735742a012666aa030eb9405cd5d0a80419a80a00f9aba150073335501802075a6ae854018c8c8c8cccd5cd19b8735573aa00490001199109198008018011919191999ab9a3370e6aae754009200023322123300100300233502a75a6ae854008c0acd5d09aba2500223263202f33573806005e05a26aae7940044dd50009aba150023232323333573466e1cd55cea8012400046644246600200600466a054eb4d5d0a80118159aba135744a004464c6405e66ae700c00bc0b44d55cf280089baa001357426ae8940088c98c80accd5ce01601581489aab9e5001137540026ae854014cd4051d71aba150043335501801c200135742a006666aa030eb88004d5d0a801180f1aba135744a004464c6404e66ae700a009c0944d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135573ca00226ea8004d5d0a80118071aba135744a004464c6403266ae7006806405c40604c98c8060cd5ce2490350543500018135573ca00226ea800448c88c008dd6000990009aa80b111999aab9f0012500a233500930043574200460066ae880080588c8c8cccd5cd19b8735573aa004900011991091980080180118061aba150023005357426ae8940088c98c8058cd5ce00b80b00a09aab9e5001137540024646464646666ae68cdc39aab9d5004480008cccc888848cccc00401401000c008c8c8c8cccd5cd19b8735573aa0049000119910919800801801180a9aba1500233500f014357426ae8940088c98c806ccd5ce00e00d80c89aab9e5001137540026ae854010ccd54021d728039aba150033232323333573466e1d4005200423212223002004357426aae79400c8cccd5cd19b875002480088c84888c004010dd71aba135573ca00846666ae68cdc3a801a400042444006464c6403a66ae7007807406c0680644d55cea80089baa00135742a00466a016eb8d5d09aba2500223263201733573803002e02a26ae8940044d5d1280089aab9e500113754002266aa002eb9d6889119118011bab00132001355013223233335573e0044a010466a00e66442466002006004600c6aae754008c014d55cf280118021aba200301413574200222440042442446600200800624464646666ae68cdc3a800a40004642446004006600a6ae84d55cf280191999ab9a3370ea0049001109100091931900919ab9c01301201000f135573aa00226ea80048c8c8cccd5cd19b875001480188c848888c010014c01cd5d09aab9e500323333573466e1d400920042321222230020053009357426aae7940108cccd5cd19b875003480088c848888c004014c01cd5d09aab9e500523333573466e1d40112000232122223003005375c6ae84d55cf280311931900919ab9c01301201000f00e00d135573aa00226ea80048c8c8cccd5cd19b8735573aa004900011991091980080180118029aba15002375a6ae84d5d1280111931900719ab9c00f00e00c135573ca00226ea80048c8cccd5cd19b8735573aa002900011bae357426aae7940088c98c8030cd5ce00680600509baa001232323232323333573466e1d4005200c21222222200323333573466e1d4009200a21222222200423333573466e1d400d2008233221222222233001009008375c6ae854014dd69aba135744a00a46666ae68cdc3a8022400c4664424444444660040120106eb8d5d0a8039bae357426ae89401c8cccd5cd19b875005480108cc8848888888cc018024020c030d5d0a8049bae357426ae8940248cccd5cd19b875006480088c848888888c01c020c034d5d09aab9e500b23333573466e1d401d2000232122222223005008300e357426aae7940308c98c8054cd5ce00b00a80980900880800780700689aab9d5004135573ca00626aae7940084d55cf280089baa0012323232323333573466e1d400520022333222122333001005004003375a6ae854010dd69aba15003375a6ae84d5d1280191999ab9a3370ea0049000119091180100198041aba135573ca00c464c6401c66ae7003c03803002c4d55cea80189aba25001135573ca00226ea80048c8c8cccd5cd19b875001480088c8488c00400cdd71aba135573ca00646666ae68cdc3a8012400046424460040066eb8d5d09aab9e500423263200b33573801801601201026aae7540044dd500089119191999ab9a3370ea00290021091100091999ab9a3370ea00490011190911180180218031aba135573ca00846666ae68cdc3a801a400042444004464c6401866ae700340300280240204d55cea80089baa0012323333573466e1d40052002200523333573466e1d40092000200523263200833573801201000c00a26aae74dd5000891001091000a4c240029210350543100112323001001223300330020020011"
}
    </code>
</details>

To start using this minting policy we first need a private and public key, for educational purposes these can be constructed in their hex notation via [this](https://paulmillr.com/noble/)* unsecure online demo of SECP256k1.

In the rest of this tutorial we will use the private key (in hex)
```
0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
```
with public key
```
034646ae5047316b4230d0086c8acec687f00b1cd9d1dc634f6cb358ac0a9a8fff
```
The message that we will sign is "The new SECP primitives are cool!" which hashes to
```
8cb0165749d5e26b3400587d811df45fbae00cc9119500d4646d43ef2eff6b07
```
Via the just mentioned online demo we retrieve for this particular message and public key the signature
```
e5f202b2334ecdea7158d2764683e81658a068eb30e48a76f9862f651e3450783602f1554c9cc802551fe698bd9899edaef374d989ab2c29ec3faa0f2c1aadec
```
Together this can be combined in the redeemer (which has the form `(v, m, s)`)
```json
{"constructor":0,"fields":[{"bytes":"034646ae5047316b4230d0086c8acec687f00b1cd9d1dc634f6cb358ac0a9a8fff"},{"bytes":"8cb0165749d5e26b3400587d811df45fbae00cc9119500d4646d43ef2eff6b07"},{"bytes":"e5f202b2334ecdea7158d2764683e81658a068eb30e48a76f9862f651e3450783602f1554c9cc802551fe698bd9899edaef374d989ab2c29ec3faa0f2c1aadec"}]}
```
To construct a transaction that mints an asset under this policy we will use the Lucid* library in combination with blockfrost*. A Blockfrost API key can be retrieved via their website and Lucid code can easily be run via the software Deno*.

To generate a Cardano private key we can use the file `writeKey.ts`
```ts
import {
  Blockfrost,
  Lucid
} from "https://deno.land/x/lucid@0.8.8/mod.ts";

const lucid = await Lucid.new(
  new Blockfrost(
    "https://cardano-preview.blockfrost.io/api/v0",
    ""
  ),
  "Preview"
);

const privateKey = lucid.utils.generatePrivateKey();
await Deno.writeTextFile("CardanoSKey.sk", privateKey);

const addr = await lucid.selectWalletFromPrivateKey(privateKey).wallet.address();
console.log(addr)
```
which we can run via the command
```bash
deno run -A writeKey.ts
```
Now we can fund the address via the [faucet](https://docs.cardano.org/cardano-testnet/tools/faucet).

To mint an native asset under this policy you can use `mint.ts`
```ts
import {
  Blockfrost,
  fromText,
  toUnit,
  Lucid,
  MintingPolicy,
  PolicyId,
  TxHash,
  Constr,
  Data,
  Unit,
} from "https://deno.land/x/lucid@0.8.8/mod.ts";

const lucid = await Lucid.new(
  new Blockfrost(
    "https://cardano-preview.blockfrost.io/api/v0",
    ""
  ),
  "Preview"
);

const privateKey: string = await Deno.readTextFile("./CardanoSKey.sk")

lucid.selectWalletFromPrivateKey(privateKey);
const addr = await lucid.wallet.address();

const mintScript: MintingPolicy = {
  type: "PlutusV2",
  script: "5907c35907c00100003232323233223232323232323232323322323232323222323253353322350022223335734666ed000c00800406c068c8c8c8cccd5cd19b8735573aa00690001199911091998008020018011bae35742a0066eb8d5d0a8011bae357426ae8940088c98c8070cd5ce00e80e00d09aba25001135573ca00226ea8010cccd5cd19b8735573aa0049000119910919800801801191919191919191919191919191999ab9a3370e6aae754031200023333333333332222222222221233333333333300100d00c00b00a00900800700600500400300233501401535742a01866a02802a6ae85402ccd4050058d5d0a805199aa80c3ae501735742a012666aa030eb9405cd5d0a80419a80a00f9aba150073335501802075a6ae854018c8c8c8cccd5cd19b8735573aa00490001199109198008018011919191999ab9a3370e6aae754009200023322123300100300233502a75a6ae854008c0acd5d09aba2500223263202f33573806005e05a26aae7940044dd50009aba150023232323333573466e1cd55cea8012400046644246600200600466a054eb4d5d0a80118159aba135744a004464c6405e66ae700c00bc0b44d55cf280089baa001357426ae8940088c98c80accd5ce01601581489aab9e5001137540026ae854014cd4051d71aba150043335501801c200135742a006666aa030eb88004d5d0a801180f1aba135744a004464c6404e66ae700a009c0944d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135744a00226ae8940044d5d1280089aba25001135573ca00226ea8004d5d0a80118071aba135744a004464c6403266ae7006806405c40604c98c8060cd5ce2490350543500018135573ca00226ea800448c88c008dd6000990009aa80b111999aab9f0012500a233500930043574200460066ae880080588c8c8cccd5cd19b8735573aa004900011991091980080180118061aba150023005357426ae8940088c98c8058cd5ce00b80b00a09aab9e5001137540024646464646666ae68cdc39aab9d5004480008cccc888848cccc00401401000c008c8c8c8cccd5cd19b8735573aa0049000119910919800801801180a9aba1500233500f014357426ae8940088c98c806ccd5ce00e00d80c89aab9e5001137540026ae854010ccd54021d728039aba150033232323333573466e1d4005200423212223002004357426aae79400c8cccd5cd19b875002480088c84888c004010dd71aba135573ca00846666ae68cdc3a801a400042444006464c6403a66ae7007807406c0680644d55cea80089baa00135742a00466a016eb8d5d09aba2500223263201733573803002e02a26ae8940044d5d1280089aab9e500113754002266aa002eb9d6889119118011bab00132001355013223233335573e0044a010466a00e66442466002006004600c6aae754008c014d55cf280118021aba200301413574200222440042442446600200800624464646666ae68cdc3a800a40004642446004006600a6ae84d55cf280191999ab9a3370ea0049001109100091931900919ab9c01301201000f135573aa00226ea80048c8c8cccd5cd19b875001480188c848888c010014c01cd5d09aab9e500323333573466e1d400920042321222230020053009357426aae7940108cccd5cd19b875003480088c848888c004014c01cd5d09aab9e500523333573466e1d40112000232122223003005375c6ae84d55cf280311931900919ab9c01301201000f00e00d135573aa00226ea80048c8c8cccd5cd19b8735573aa004900011991091980080180118029aba15002375a6ae84d5d1280111931900719ab9c00f00e00c135573ca00226ea80048c8cccd5cd19b8735573aa002900011bae357426aae7940088c98c8030cd5ce00680600509baa001232323232323333573466e1d4005200c21222222200323333573466e1d4009200a21222222200423333573466e1d400d2008233221222222233001009008375c6ae854014dd69aba135744a00a46666ae68cdc3a8022400c4664424444444660040120106eb8d5d0a8039bae357426ae89401c8cccd5cd19b875005480108cc8848888888cc018024020c030d5d0a8049bae357426ae8940248cccd5cd19b875006480088c848888888c01c020c034d5d09aab9e500b23333573466e1d401d2000232122222223005008300e357426aae7940308c98c8054cd5ce00b00a80980900880800780700689aab9d5004135573ca00626aae7940084d55cf280089baa0012323232323333573466e1d400520022333222122333001005004003375a6ae854010dd69aba15003375a6ae84d5d1280191999ab9a3370ea0049000119091180100198041aba135573ca00c464c6401c66ae7003c03803002c4d55cea80189aba25001135573ca00226ea80048c8c8cccd5cd19b875001480088c8488c00400cdd71aba135573ca00646666ae68cdc3a8012400046424460040066eb8d5d09aab9e500423263200b33573801801601201026aae7540044dd500089119191999ab9a3370ea00290021091100091999ab9a3370ea00490011190911180180218031aba135573ca00846666ae68cdc3a801a400042444004464c6401866ae700340300280240204d55cea80089baa0012323333573466e1d40052002200523333573466e1d40092000200523263200833573801201000c00a26aae74dd5000891001091000a4c240029210350543100112323001001223300330020020011"
};
const mintingPolicy: PolicyId = lucid.utils.mintingPolicyToId(mintScript);

const redeemer = new Constr(0, ["021bc968d1e44e4aadc18f7119b4a4afa201e7423f42e88468f8c23bbbb9f2f061", "8cb0165749d5e26b3400587d811df45fbae00cc9119500d4646d43ef2eff6b07", "545d797d75f4849e56077eb8956fced83b79801a8e8640603f7ff8061a3fef3455572cea4cc9f3d6e35a797141493a2b03fa6a7f48b1c8ea31a6880b4680b3d7"])

export async function mint(
  name: string
): Promise<TxHash> {
  const unit: Unit = toUnit(mintingPolicy,fromText(name));

  const tx = await lucid
    .newTx()
    .mintAssets({ [unit]: 1n }, Data.to(redeemer))
    .attachMintingPolicy(mintScript)
    .complete({nativeUplc: false});

  const signedTx = await tx.sign().complete();
  const txHash = await signedTx.submit();
  return txHash
}

console.log(addr);
console.log(await mint("testToken"))
```
Which we can run via the command
```bash
deno run -A mint.ts
```

\* The information contained or reflected herein is provided for informational purposes only and (1) does not constitute an endorsement of any product or project by Input Output Global Inc. (IOG); (2) does not constitute investment advice, nor represent an expert opinion or negative assurance letter; (3) is not a substitute for a piece of professional advice; (4)  has not been submitted to, nor received approval from, any relevant regulatory bodies.
The information contained or reflected herein is made available by third parties, subject to continuous change and therefore is not warranted as to their merchantability, completeness, accuracy, up-to-dateness or fitness for a particular purpose. The information and data are provided “as is” only.
IOG does not accept any liability for damage arising from the use of the information herein.
