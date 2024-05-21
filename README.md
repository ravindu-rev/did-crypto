# did-crypto
[![crates.io](https://buildstats.info/crate/did_crypto)](https://crates.io/crates/did_crypto)
![Test](https://github.com/ravindu-rev/did-crypto/actions/workflows/test.yaml/badge.svg)
![Package publish](https://github.com/ravindu-rev/did-crypto/actions/workflows/publish.yaml/badge.svg)
![Doc](https://github.com/ravindu-rev/did-crypto/actions/workflows/publish-doc.yaml/badge.svg)

Did-Crypto library is focused on managing the signing and verification. API documentation on [docs.rs](https://docs.rs/did-crypto/latest/did_crypto/)

## Algorithms

This library currently supports the following:

- HS256
- HS384
- HS512
- RS256
- RS384
- RS512
- PS256
- PS384
- PS512
- ES256
- ES384
- ES512
- ES256K
- EdDSA

## Signer

Signs a string content using a provided algorithm and a signing key

```rust, ignore
    use did_crypto::{
        algorithms::Algorithm,
        crypto::ecdsa::{
                _256k::{P256kSigningKey},
            },
        signer::sign
        }

    let signature_result = sign(
        String::from(CONTENT),
        P256kSigningKey::from_bytes(&hex::decode(PRIVATE_KEY_256K_HEX).unwrap().as_slice())
            .unwrap(),
        Algorithm::ES256K,
    );

    let signature = match signature_result {
        Ok(val) => val,
        Err(error) => {
            eprintln!("{}", error);
            panic!()
        }
    };
```

## Verifier

Verifies the content with the signature using a provided algorithm.

```rust, ignore
    use did_crypto::{
        algorithms::Algorithm,
        crypto::ecdsa::{
                _256k::{P256kVerifyingKey},
            },
        verifier::verify
        }

    let verified = match verify(
        String::from(CONTENT),
        signature,
        P256kVerifyingKey::from_bytes(hex::decode(PUBLIC_KEY_256K_HEX).unwrap().as_slice())
            .unwrap(),
        Algorithm::ES256K
    ) {
        Ok(val) => val,
        Err(error) => {
            println!("{}", error.to_string());
            panic!()
        }
    };
```

## JWT

```rust, ignore
    use chrono::Utc;
    use did_crypto::{
        algorithms::Algorithm,
        crypto::ecdsa::_512::{P512SigningKey, P512VerifyingKey},
        jwt::{Header, Payload, JWT},
    };
    use serde_json::json;

    let now = Utc::now().timestamp_millis() / 1000;

    let payload_content: Value = json!(
        {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true,
    "iat": 151623902,
    "exp": now + (6 * 24 * 60 * 60)
    }
    );

    let mut jwt = JWT {
        header: Header::new(String::from("id:129877"), Algorithm::ES512),
        payload: Payload(payload_content),
        signature: None,
    };

    match jwt.sign(P512SigningKey::from_pem(PRIVATE_KEY).unwrap()) {
        Ok(()) => {}
        Err(error) => {
            println!("{}", error);
            panic!()
        }
    };

    match jwt.to_token() {
        Ok(val) => val,
        Err(error) => {
            println!("{}", error);
            panic!()
        }
    };
```

## WASM

### Sign a JWT token

```JS 
const didCrypto = await import("did-crypto");

let header = new Header(KID, Algorithm.HS256)

let payload = new Payload({
    exp: EXPIRE,
    sub: test
})

let jwtObject = new didCrypto.JWT(header, payload, null);
// Either a byte array of a private key or 
// {pem: PEM_CONTENT}, {passphrase: PASSPHRASE} or {n: N_VALUE,e: E_VALUE, ...} 
jwtObject.sign(SIGNING_OBJECT);
let token = jwtObject.toToken();
```

### Verify a JWT token

```JS 
const didCrypto = await import("did-crypto");

didCrypto.JWT.validate_token(
    JWT_TOKEN,
    Array.prototype.slice.call(
        Buffer.from(
        "04115b3fa39fae41b4e32f7721ca72f8c1781483647dabd514f08e66128bd47fce9067b90e0488c9c2a9f30f5a266a07841d6c077413ba07e74569b99d4fd3cec6",
        "hex"
        ),
        0
    )
)
```