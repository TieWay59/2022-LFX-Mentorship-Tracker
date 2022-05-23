# week-00

## Use of ring-rs in rustls

Searched keyword `ring::` among the Rustls project, I found the invokes as below:

Although they import the module at the beginning, however, Rustls devs don't always use the imported module.

Instead, they tend to write full mod paths like `ring::aead::NONCE_LEN` in the code more frequently even when it's already imported by `use`.

```text
85 个结果 - 22 文件

rustls/src/cipher.rs:
   5: use ring::{aead, hkdf};
  33: pub(crate) struct Iv(pub(crate) [u8; ring::aead::NONCE_LEN]);
  37:     fn new(value: [u8; ring::aead::NONCE_LEN]) -> Self {
  43:         debug_assert_eq!(value.len(), ring::aead::NONCE_LEN);
  71: pub(crate) fn make_nonce(iv: &Iv, seq: u64) -> ring::aead::Nonce {
  72:     let mut nonce = [0u8; ring::aead::NONCE_LEN];

rustls/src/conn.rs:
  1382:     pub(crate) early_secret: Option<ring::hkdf::Prk>,

rustls/src/hash_hs.rs:
    4: use ring::digest;
  167:     use ring::digest;

rustls/src/kx.rs:
   8:     privkey: ring::agreement::EphemeralPrivateKey,
   9:     pub(crate) pubkey: ring::agreement::PublicKey,
  28:         let rng = ring::rand::SystemRandom::new();
  30:             ring::agreement::EphemeralPrivateKey::generate(skxg.agreement_algorithm, &rng).ok()?;
  55:         let peer_key = ring::agreement::UnparsedPublicKey::new(self.skxg.agreement_algorithm, peer);
  56:         ring::agreement::agree_ephemeral(self.privkey, &peer_key, (), f)
  71:     agreement_algorithm: &'static ring::agreement::Algorithm,
  77:     agreement_algorithm: &ring::agreement::X25519,
  83:     agreement_algorithm: &ring::agreement::ECDH_P256,
  89:     agreement_algorithm: &ring::agreement::ECDH_P384,

rustls/src/quic.rs:
   12: use ring::{aead, hkdf};
  462: fn nonce_for(packet_number: u64, iv: &Iv) -> ring::aead::Nonce {

rustls/src/rand.rs:
  5: use ring::rand::{SecureRandom, SystemRandom};

rustls/src/sign.rs:
    6: use ring::io::der;
    7: use ring::signature::{self, EcdsaKeyPair, Ed25519KeyPair, RsaKeyPair};
  249:         let rng = ring::rand::SystemRandom::new();
  369:         let rng = ring::rand::SystemRandom::new();

rustls/src/suites.rs:
  46:     pub(crate) aead_algorithm: &'static ring::aead::Algorithm,
  64:     pub fn hash_algorithm(&self) -> &'static ring::digest::Algorithm {

rustls/src/ticketer.rs:
   5: use ring::aead;
  71:         let nonce = ring::aead::Nonce::assume_unique_for_key(nonce_buf);
  72:         let aad = ring::aead::Aad::empty();
  94:         let nonce = ring::aead::Nonce::try_assume_unique_for_key(nonce).ok()?;

rustls/src/verify.rs:
  10: use ring::digest::Digest;

rustls/src/x509.rs:
  3: use ring::io::der;

rustls/src/client/tls12.rs:
  29: use ring::agreement::PublicKey;
  30: use ring::constant_time;

rustls/src/client/tls13.rs:
   40: use ring::constant_time;
  806:     verify_data: ring::hmac::Tag,

rustls/src/server/tls12.rs:
  23: use ring::constant_time;

rustls/src/server/tls13.rs:
  30: use ring::constant_time;

rustls/src/tls12/cipher.rs:
   9: use ring::aead;
  18: ) -> ring::aead::Aad<[u8; TLS12_AAD_SIZE]> {
  24:     ring::aead::Aad::from(out)

rustls/src/tls12/mod.rs:
   11: use ring::aead;
   12: use ring::digest::Digest;
   28:             aead_algorithm: &ring::aead::CHACHA20_POLY1305,
   35:         hmac_algorithm: ring::hmac::HMAC_SHA256,
   45:             aead_algorithm: &ring::aead::CHACHA20_POLY1305,
   52:         hmac_algorithm: ring::hmac::HMAC_SHA256,
   62:             aead_algorithm: &ring::aead::AES_128_GCM,
   69:         hmac_algorithm: ring::hmac::HMAC_SHA256,
   79:             aead_algorithm: &ring::aead::AES_256_GCM,
   86:         hmac_algorithm: ring::hmac::HMAC_SHA384,
   96:             aead_algorithm: &ring::aead::AES_128_GCM,
  103:         hmac_algorithm: ring::hmac::HMAC_SHA256,
  113:             aead_algorithm: &ring::aead::AES_256_GCM,
  120:         hmac_algorithm: ring::hmac::HMAC_SHA384,
  146:     pub(crate) hmac_algorithm: ring::hmac::Algorithm,
  182:     pub fn hash_algorithm(&self) -> &'static ring::digest::Algorithm {

rustls/src/tls12/prf.rs:
   1: use ring::hmac;
  40:     use ring::hmac::{HMAC_SHA256, HMAC_SHA512};

rustls/src/tls13/key_schedule.rs:
    7: use ring::{
   65:     algorithm: ring::hkdf::Algorithm,
  548:     use ring::{aead, hkdf};

rustls/src/tls13/mod.rs:
   10: use ring::{aead, hkdf};
   25:         aead_algorithm: &ring::aead::CHACHA20_POLY1305,
   27:     hkdf_algorithm: ring::hkdf::HKDF_SHA256,
   40:             aead_algorithm: &ring::aead::AES_256_GCM,
   42:         hkdf_algorithm: ring::hkdf::HKDF_SHA384,
   57:         aead_algorithm: &ring::aead::AES_128_GCM,
   59:     hkdf_algorithm: ring::hkdf::HKDF_SHA256,
   70:     pub(crate) hkdf_algorithm: ring::hkdf::Algorithm,
  101:     pub fn hash_algorithm(&self) -> &'static ring::digest::Algorithm {
  154: fn make_tls13_aad(len: usize) -> ring::aead::Aad<[u8; TLS13_AAD_SIZE]> {
  155:     ring::aead::Aad::from([

rustls/tests/api.rs:
  3242:         use ring::rand::SecureRandom;
  3252:         let rng = ring::rand::SystemRandom::new();
  3257:         let kx = ring::agreement::EphemeralPrivateKey::generate(&ring::agreement::X25519, &rng)
  3306:         use ring::rand::SecureRandom;
  3316:         let rng = ring::rand::SystemRandom::new();
  3321:         let kx = ring::agreement::EphemeralPrivateKey::generate(&ring::agreement::X25519, &rng)

rustls-mio/tests/common/mod.rs:
  17: use ring::rand::SecureRandom;
  57:             ring::rand::SystemRandom::new()

```

## Table related symbols

| symbol                                   | type       | intro                                                                                                                                                                                                                     | notes                                                               |
| ---------------------------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| `ring::aead`                             | Module     | Authenticated Encryption with Associated Data (AEAD).                                                                                                                                                                     |                                                                     |
| `aead::NONCE_LEN`                        | Constant   | The maximum length of a tag for the algorithms in this module.                                                                                                                                                            |                                                                     |
| `aead::Nonce `                           | Struct     | A nonce for a single AEAD opening or sealing operation.                                                                                                                                                                   | `aead::Nonce::assume_unique_for_key(nonce)`                         |
| `aead::quic`                             | sub Module | QUIC Header Protection.                                                                                                                                                                                                   |                                                                     |
| `aead::quic::HeaderProtectionKey`        | Struct     | A key for generating QUIC Header Protection masks.                                                                                                                                                                        |                                                                     |
| `aead::quic::AES_128`                    | Static     |                                                                                                                                                                                                                           |                                                                     |
| `aead::quic::AES_256`                    | Static     |                                                                                                                                                                                                                           |                                                                     |
| `aead::quic::CHACHA20`                   | Static     |                                                                                                                                                                                                                           |                                                                     |
| `aead::LessSafeKey`                      | Struct     | Immutable keys for use in situations where OpeningKey/SealingKey and NonceSequence cannot reasonably be used.                                                                                                             |                                                                     |
| `aead::Aad`                              | Struct     | The additionally authenticated data (AAD) for an opening or sealing operation. This data is authenticated but is not encrypted.                                                                                           |                                                                     |
| `aead::Tag`                              | Struct     | An authentication tag.                                                                                                                                                                                                    |                                                                     |
| `ring::agreement`                        | Module     | Key Agreement: ECDH, including X25519.                                                                                                                                                                                    |                                                                     |
| `agreement::Algorithm`                   | Struct     | A key agreement algorithm.                                                                                                                                                                                                |                                                                     |
| `agreement::EphemeralPrivateKey`         | Struct     | An ephemeral private key for use (only) with agree_ephemeral. The signature of agree_ephemeral ensures that an EphemeralPrivateKey can be used for at most one key agreement.                                             |                                                                     |
| `agreement::PublicKey`                   | Struct     | A public key for key agreement.                                                                                                                                                                                           |                                                                     |
| `agreement::UnparsedPublicKey`           | Struct     | An unparsed, possibly malformed, public key for key agreement.                                                                                                                                                            |                                                                     |
| `agreement::agree_ephemeral`             | Function   | Performs a key agreement with an ephemeral private key and the given public key.                                                                                                                                          |                                                                     |
| `agreement::X25519`                      | Static     |                                                                                                                                                                                                                           |                                                                     |
| `agreement::ECDH_P256`                   | Static     |                                                                                                                                                                                                                           |                                                                     |
| `agreement::ECDH_P384`                   | Static     |                                                                                                                                                                                                                           |                                                                     |
| `ring::constant_time`                    | Module     | Constant-time operations.                                                                                                                                                                                                 |                                                                     |
| `constant_time::verify_slices_are_equal` | Function   | Returns Ok(()) if a == b and Err(error::Unspecified) otherwise. The comparison of a and b is done in constant time with respect to the contents of each, but NOT in constant time with respect to the lengths of a and b. |                                                                     |
| `ring::digest`                           | Module     | SHA-2 and the legacy SHA-1 digest algorithm.                                                                                                                                                                              |                                                                     |
| `digest::Algorithm`                      | Struct     | A digest algorithm.                                                                                                                                                                                                       |                                                                     |
| ` digest::Context`                       | Struct     | A context for multi-step (Init-Update-Finish) digest calculations.                                                                                                                                                        |                                                                     |
| `digest::Digest`                         | Struct     | A calculated digest value.                                                                                                                                                                                                |                                                                     |
| `digest::SHA256`                         | Static     | SHA-256 as specified in [FIPS 180-4](http://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf).                                                                                                                          |                                                                     |
| `ring::hkdf`                             | Module     | HMAC-based Extract-and-Expand Key Derivation Function.                                                                                                                                                                    |                                                                     |
| `hkdf::Okm`                              | Struct     | An HKDF OKM (Output Keying Material)                                                                                                                                                                                      | `From<hkdf::Okm<'_, IvLen>>`                                        |
| `hkdf::Prk`                              | Struct     | A HKDF PRK (pseudorandom key).                                                                                                                                                                                            |                                                                     |
| `hkdf::Salt`                             | Struct     | A salt for HKDF operations.                                                                                                                                                                                               |                                                                     |
| `hkdf::HKDF_SHA256`                      | Static     | HKDF using HMAC-SHA-256.                                                                                                                                                                                                  |                                                                     |
| `hkdf::KeyType`                          | trait      | The length of the OKM (Output Keying Material) for a `Prk::expand()` call.                                                                                                                                                |                                                                     |
| `ring::hmac`                             | Module     | HMAC is specified in [RFC 2104](https://tools.ietf.org/html/rfc2104).                                                                                                                                                     |                                                                     |
| `hmac::Tag`                              | Struct     | An HMAC tag.                                                                                                                                                                                                              |                                                                     |
| `ring::rand`                             | Module     | Cryptographic pseudo-random number generation.                                                                                                                                                                            |                                                                     |
| `rand::SecureRandom`                     | Trait      | A secure random number generator.                                                                                                                                                                                         |                                                                     |
| `rand::SystemRandom`                     | Struct     | A secure random number generator where the random values come directly from the operating system.                                                                                                                         |                                                                     |
| `ring::signature`                        | Module     | Public key signatures: signing and verification.                                                                                                                                                                          |                                                                     |
| `signature::EcdsaKeyPair`                | Struct     | An ECDSA key pair, used for signing.                                                                                                                                                                                      |                                                                     |
| `signature::Ed25519KeyPair`              | Struct     | An Ed25519 key pair, for signing.                                                                                                                                                                                         |                                                                     |
| `signature::RsaKeyPair`                  | Struct     | An RSA key pair, used for signing.                                                                                                                                                                                        |                                                                     |
| `ring::io::der`                          |            | can't found in official doc .                                                                                                                                                                                             | Parse `der` as any ECDSA key type, returning the first which works. |
| `der::Tag::Sequence`                     |            | can't found in official doc .                                                                                                                                                                                             |                                                                     |
