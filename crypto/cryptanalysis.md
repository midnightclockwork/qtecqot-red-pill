# Cryptanalysis — 96-Byte Base64 Blob

**Artifact**

```text
aRgyjPiynAbJQ1KPAIWoBZuxgKuzBX4aqfk8XaJsLcfJSI2ymo5AbwWPJE+muMC8ZP2GBHHHh9cgdNjIbDp/xFp5xPk91F0u8llj3X7mpTzGzMdA0+bLalGKr4BHYY4k
```

**Purpose:** Technical analysis of the artifact's encoding, statistical structure, possible cryptographic representations, and recoverability.

**Status:** Unsolved.

---

## 1. Executive Verdict

The Base64 string decodes cleanly to exactly **96 bytes / 768 bits** of binary data.

The decoded bytes are not readable ASCII or valid UTF-8 text. They have no repeated byte n-grams of lengths 2–16 and no repeated aligned 16-byte blocks. Their byte frequencies, bit balance, runs, and tested lag statistics are ordinary for a short uniform-random sample. The payload fails direct parsing as several common compressed and cryptographic container formats.

The strongest defensible description is:

> **96 bytes of random-looking binary data, compatible with encryption, cryptographic objects, or other low-redundancy representations. The bytes do not identify a cipher, mode, key size, KDF, or field layout.**

The data fits AES-CBC, AES-GCM, AES-SIV, stream-like modes, several other authenticated constructions, and some fixed-size cryptographic objects. No statistically justified ordering of these possibilities follows from this one sample.

An exact constraint excludes **repeating-key XOR with periods 1–94 when the entire plaintext is 7-bit**. Exhaustive tests of several simple transform families also produce no complete printable-text output.

No verified plaintext has been recovered. Further decryption requires a candidate key and a sufficiently specified construction, or an identifiable weakness in the generating process.

---

## 2. Ground Truth: Base64 Decoding

```text
aRgyjPiynAbJQ1KPAIWoBZuxgKuzBX4aqfk8XaJsLcfJSI2ymo5AbwWPJE+muMC8ZP2GBHHHh9cgdNjIbDp/xFp5xPk91F0u8llj3X7mpTzGzMdA0+bLalGKr4BHYY4k
```

It has 128 characters and decodes, with strict alphabet validation, to 96 bytes. Re-encoding produces the same string. The `+` and `/` characters belong to standard Base64; no padding is required because 96 is divisible by three. These are encoding facts, independent of encryption. [RFC 4648](https://www.rfc-editor.org/rfc/rfc4648)

```text
offset  hexadecimal bytes
00      69 18 32 8c f8 b2 9c 06 c9 43 52 8f 00 85 a8 05
10      9b b1 80 ab b3 05 7e 1a a9 f9 3c 5d a2 6c 2d c7
20      c9 48 8d b2 9a 8e 40 6f 05 8f 24 4f a6 b8 c0 bc
30      64 fd 86 04 71 c7 87 d7 20 74 d8 c8 6c 3a 7f c4
40      5a 79 c4 f9 3d d4 5d 2e f2 59 63 dd 7e e6 a5 3c
50      c6 cc c7 40 d3 e6 cb 6a 51 8a af 80 47 61 8e 24
```

Offsets above are hexadecimal; array offsets elsewhere are zero-based decimal. The SHA-256 of the decoded bytes is:

```text
48cd09a278502fee33b38159ec08e82f2097d2141947f491759a70cc286c30ad
```

---

## 3. Verified Measurements

| Measurement | Recomputed result | Assessment |
|---|---:|---|
| Distinct byte values | 78 | Uniform expectation: 80.1831 |
| Multiplicities | 62 singletons, 14 doubletons, 2 tripletons | Directly measured |
| Tripletons | `05`, `c7` | Neither is unusual |
| Empirical byte entropy | 6.194235678 bits/byte | Histogram statistic, not source entropy |
| Index of coincidence | 0.0043859649 | 20 equal unordered byte pairs |
| Pearson χ², 256 bins | 266.6666667 | Sparse bins; avoid naïve asymptotic significance |
| One bits | 370 / 768 | Exact two-sided binomial p ≈ 0.329921 |
| Bit runs, MSB-first | 373 | Ordinary in the simulation below |
| Printable ASCII, `20`–`7e` | 35 / 96 | Uniform expectation: 35.625 |
| Bytes with high bit set | 52 / 96 | Ordinary |
| Repeated sliding byte n-grams, lengths 2–16 | None | No repeated byte sequences in this range |
| Distinct aligned 16-byte blocks | 6 / 6 | Does not exclude ECB |
| Pairwise block Hamming distances | 56–73; mean 64.1333 / 128 | Consistent with independent random blocks |
| Distinct Base64 symbols | 57 / 64 | Directly measured |
| Base64 index of coincidence | 0.0139025591 | Directly measured |
| Repeated Base64 digrams | `Ab`, `BH`, `HH`, each twice | No repeated trigrams |

The MSB-to-LSB one counts are `52, 50, 44, 38, 50, 51, 42, 43`. The 15 block distances are `64, 64, 67, 62, 67, 62, 65, 56, 73, 73, 62, 57, 67, 60, 63`.

---

## 4. Randomness and Structure Analysis

### 4.1 Byte distribution and entropy

The 96 bytes contain 78 distinct values: 62 occur once, 14 occur twice, and two occur three times. There is no marked concentration on a small alphabet.

The empirical byte entropy is **6.194235678 bits/byte**. With only 96 observations, its maximum is `log2(96) = 6.58496`, not eight. A sample from a uniform byte source therefore need not have an empirical entropy close to eight.

Source entropy cannot be established from this histogram. A deterministic generator could emit the same sample. The appropriate conclusion is compatibility with a random source under the tested statistics.

### 4.2 Statistical dependence

Byte index of coincidence and Pearson χ² are algebraically related through the byte counts. Entropy and occupancy summarize those same counts, and Base64 statistics re-express the same underlying bits. These measurements should not be multiplied together as independent evidence for encryption.

### 4.3 Inter-block Hamming distances

The six 16-byte blocks have pairwise Hamming distances of 56–73 bits, averaging 64.1333. Independent uniform 128-bit strings have expected distance 64.

This is compatible with independently random-looking blocks. It does not measure a cipher's avalanche behavior, which requires controlled changes to inputs or keys.

### 4.4 Random-sample calibration



The analysis generated 50,000 independent uniform 96-byte samples using NumPy's seeded generator, seed `20260911`. The interval below is the central 95% of simulated statistics, **not** a confidence interval for the artifact's entropy or origin.

| Statistic | Artifact | Simulated central 95% |
|---|---:|---:|
| Distinct bytes | 78 | 74–86 |
| Equal unordered byte pairs | 20 | 10–27 |
| Empirical entropy | 6.19424 | 6.09007–6.37663 |
| Bit runs | 373 | 357–411 |

For byte lags 1–32, the test compared the bit Hamming distance between overlapping shifted copies with its uniform expectation. The largest absolute standardized deviation was 2.19971, at lag 3. The **same maximum-over-32-lags selection rule** was applied to each simulated sample. The resulting scan-adjusted p-value was approximately **0.58781**, with Monte Carlo standard error about 0.0022. There is no useful lag anomaly in this test.

This calibration does not make the tests powerful against every alternative. It establishes only that these observations are unexceptional under the specified null model.



### 4.5 Absence of repeats

The expected number of matching byte-pair windows in a uniform sample of this length is only about `C(95,2)/65536 ≈ 0.0681`. Absence of repeated digrams is therefore unsurprising and does not, by itself, eliminate a repeating-key cipher.

---

## 5. Compression Analysis

All five full-stream decompression attempts failed. The analysis also tried the suffix beginning at **every byte offset 0–95** with each decoder: no suffix reached the decoder's end-of-stream flag. This adds limited coverage of prepended metadata. It does not exhaust bit offsets, preprocessing, dictionaries, custom compression settings, or every codec.

| Decoder | Whole-blob result |
|---|---|
| zlib | Incorrect header check |
| gzip | Incorrect header check |
| Raw DEFLATE | Invalid stored block lengths |
| bzip2 | Invalid data stream |
| LZMA auto, XZ/legacy container detection | Unsupported input format |

The raw-DEFLATE failure has a direct explanation: `69` begins a final stored block when read in DEFLATE bit order. After byte alignment, its length fields are `0x3218` and `0xf88c`; they are not one's complements. This excludes the exact byte-aligned complete stream without appealing to compressibility. [RFC 1951](https://www.rfc-editor.org/rfc/rfc1951)

Recompression yields 101 bytes with raw DEFLATE, 107 with zlib, 119 with gzip, 175 with bzip2, and 152 with XZ. At this size expansion is ordinary; it neither establishes encryption nor excludes already-compressed data. The 152-byte result uses the default XZ container, rather than raw LZMA.



**Conclusion:** the complete byte stream and tested suffixes do not parse under the tested decoders. Compression before encryption, unsupported compression formats, and fragments from a larger compressed stream remain possible.

---

## 6. Simple Transform and Classical-Cipher Analysis

### 6.1 Repeating-key XOR

Let `C[i] = P[i] XOR K[i mod p]`, where the key has period `p`. If every plaintext byte is 7-bit, its high bit is zero. Consequently, the high bit of `Cᵢ` must itself repeat with period `p`.

For **every p from 1 through 94**, this blob contains at least one pair of positions `i` and `i+p` whose high bits differ. Thus no key of any of those periods can produce an entirely 7-bit plaintext from the complete 96-byte blob.

Periods 95 and 96 pass that necessary test. They also admit plaintexts drawn entirely from printable ASCII plus tab, LF, and CR, but such nearly message-length keys do not offer a useful cryptanalytic constraint.

This exclusion does **not** apply unchanged to a payload segment after removing binary metadata, UTF-8 text containing non-ASCII characters, compressed data, a non-repeating keystream, or additional preprocessing. It is an exact result for a particular model, not a universal rejection of XOR.

For periodic addition modulo 256, exhaustive per-column feasibility against printable ASCII plus tab/LF/CR excludes all periods below 62. The feasible periods through 96 are `62, 63, 65, 78, 83, 86, 90, 91, 92, 94, 95, 96`. Feasibility is only a character-set condition, not recovered English.

### 6.2 Exhaustive Simple-Transform Tests

“Text byte” in this table means `0x20`–`0x7e`, tab, LF, or CR. No tested transform produced an output consisting entirely of those bytes.

| Family | Trials | Most text bytes in any result |
|---|---:|---:|
| Single-byte XOR | 256 | 48 / 96 |
| Addition modulo 256 | 256 | 50 / 96 |
| Invertible byte-affine maps `a*x+b mod 256`, odd a | 32,768 | 56 / 96 |
| All permutations of the 8 bit positions within each byte | 40,320 | 52 / 96 |
| Per-byte bit rotations, forward/reversed byte order | 16 | 47 / 96 |
| Adjacent XOR, forward/reversed | 2 | 40 / 95 |
| Adjacent modular difference, forward/reversed | 2 | 40 / 95 |
| Base64 alphabet index shifts/reflections on original, reversed, and case-swapped strings | 384 | 50 / 96 |

These 74,004 trials overlap; they are not 74,004 independent pieces of evidence. Compositions of every listed family were not exhausted. An unrestricted alphabet permutation was not searched.

Plain byte transposition cannot remove the original nontext bytes. A fixed one-to-one substitution preserves the count of 78 distinct symbols, inconsistent with ordinary prose using a small alphabet, but not with every conceivable text or binary representation. A classical cipher applied *before* compression or other binary conversion remains a separate hypothesis.

### 6.3 Vigenère and Kasiski Examination

No repeated byte n-grams of lengths 2–16 are present. Kasiski examination therefore has no repeated fragments from which to infer a period. This is an absence of diagnostic evidence, not a universal exclusion of Vigenère-like mappings. Ordinary alphabet-preserving textual ciphers are poor fits for the immediate decoded representation.

---

## 7. Block Structure and ECB

The payload comprises six distinct aligned 16-byte blocks. No repeated sliding byte sequences of lengths 2–16 occur.

ECB maps equal plaintext blocks to equal ciphertext blocks under one key. Six different plaintext blocks can therefore produce six different ciphertext blocks under ECB.

**Conclusion:** the absence of repeated blocks does not rule out ECB. CBC, ECB, and stream-based ciphertext can all exhibit the observed lack of repetition. Cipher preference cannot be inferred from implementation popularity alone.

---

## 8. The 96-Byte Length

The length has several compatible partitions:

```text
96 = 6 × 16
96 = 12 × 8
96 = 3 × 32
96 = 2 × 48
96 = 16 + 80
96 = 12 + 68 + 16
96 = 24 + 56 + 16
96 = 16 + 16 + 64
96 = 32 + 64
```

A padded 16-byte block cipher naturally produces ciphertext in multiples of 16. However, the observed length can include an IV, nonce, salt, authentication tag, or other metadata. Stream and authenticated constructions can also produce exactly 96 bytes.

A likelihood ratio of approximately 16:1 for block-aligned output would require a competing model whose output lengths are uniform modulo 16. Without a specified message-length and overhead model, that ratio cannot be assigned to this artifact.

**Conclusion:** length constrains candidate layouts but does not identify the primitive or establish a mode ranking.

---

## 9. Candidate Layouts

All sizes below are bytes. These are examples of compatibility, **not likelihood rankings**. Plaintext length means bytes immediately entering the listed construction, before considering compression or an inner layer.

| Construction | A possible 96-byte layout | Implied plaintext size |
|---|---|---:|
| AES-CBC with PKCS#7 | IV16 + ciphertext80 | 64–79 |
| AES-CBC/ECB with PKCS#7, external IV if needed | ciphertext96 | 80–95 |
| Password envelope with CBC | salt16 + IV16 + ciphertext64 | 48–63 |
| AES-GCM or IETF ChaCha20-Poly1305 | nonce12 + ciphertext68 + tag16 | 68 |
| AES-GCM with 16-byte nonce | nonce16 + ciphertext64 + tag16 | 64 |
| Password envelope with GCM | salt16 + nonce12 + ciphertext52 + tag16 | 52 |
| AEAD, externally supplied nonce | ciphertext80 + tag16 | 80 |
| AES-SIV | SIV16 + ciphertext80 | 80 |
| CTR/CFB/OFB | IV16 + ciphertext80 | 80 |
| XChaCha20-Poly1305 | nonce24 + ciphertext56 + tag16 | 56 |
| XSalsa20-Poly1305 secretbox, nonce prefixed | nonce24 + MAC16 + ciphertext56 | 56 |
| Encrypt-then-MAC example | IV16 + CBC ciphertext48 + MAC32 | 32–47 with PKCS#7 |
| AES Key Wrap | wrapped output96 | 88 |
| AES Key Wrap with Padding | wrapped output96 | 81–88 |
| 8-byte-block CBC with byte-count padding | IV8 + ciphertext88 | 80–87 |

---

## 10. AES-CBC Hypothesis

### 10.1 Prepended IV

A compatible layout is:

```text
bytes  0–15: 16-byte IV
bytes 16–95: 80-byte ciphertext
```

With PKCS#7 padding, this implies **64–79 plaintext bytes**. Padding always adds 1–16 bytes, including a complete block when the input is already aligned.

### 10.2 Other layouts

All 96 bytes could instead be ciphertext with an external IV, implying 80–95 plaintext bytes with PKCS#7. A layout of `salt16 || IV16 || ciphertext64` implies 48–63 plaintext bytes.

Nothing in the first 16 bytes identifies them as an IV. Random IVs and ciphertext blocks can have the same statistical appearance.

### 10.3 Testing with an unknown IV

For contiguous CBC ciphertext blocks:

```text
P1 = D_K(C1) XOR IV
Pi = D_K(Ci) XOR C(i−1), for i ≥ 2
```

An unknown external IV affects only recovery of the first plaintext block. Given a candidate key and the correct ciphertext boundary, later blocks can still be tested.

### 10.4 Verification

A random last block satisfies PKCS#7 with probability approximately `1/255 ≈ 0.392%`. Valid padding is a useful filter but does not authenticate a key or establish a message. Require coherent plaintext structure and independent support for the parameterization. [RFC 5652 §6.3](https://www.rfc-editor.org/rfc/rfc5652#section-6.3)

---

## 11. AES-GCM and Other AEAD

A conventional GCM interpretation is:

```text
12-byte nonce || 68-byte ciphertext || 16-byte tag
```

A 12-byte nonce is the conventional GCM choice. A 16-byte nonce is also compatible, giving `16 + 64 + 16`. An external nonce could leave `80-byte ciphertext || 16-byte tag`. [NIST SP 800-38D](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf)

IETF ChaCha20-Poly1305 also uses a 12-byte nonce and 16-byte tag, so the same `12 + 68 + 16` partition is compatible. Nonce and tag placement depend on serialization. [RFC 8439](https://www.rfc-editor.org/rfc/rfc8439)

Authentication success is a strong verification signal. Failure rejects only the tested key, nonce, ciphertext, tag, and associated-data combination. It does not independently distinguish a wrong password from a wrong KDF or layout.

Other authenticated modes, including CCM, OCB, and GCM-SIV, can also fit. The length alone does not select between them.

---

## 12. AES-SIV

AES-SIV can represent the artifact as:

```text
16-byte synthetic IV || 80-byte ciphertext
```

The plaintext would be 80 bytes. AES-SIV permits deterministic operation and does not require a conventional random nonce in every use.

Its total key length is **32, 48, or 64 bytes**, split into two AES keys. Candidate testing must account for these lengths rather than using only ordinary single-AES key lengths. AES-SIV and AES-GCM-SIV are different constructions. [RFC 5297](https://www.rfc-editor.org/rfc/rfc5297)

**Assessment:** structurally compatible; no identifying evidence.

---

## 13. AES Key Wrap

RFC 3394 Key Wrap adds one 8-byte block. A 96-byte output can therefore wrap **88 bytes** of input. The algorithm's input need not be a single conventional AES key. [RFC 3394](https://www.rfc-editor.org/info/rfc3394/)

Key Wrap with Padding rounds the input up to an 8-byte boundary before adding the overhead. A 96-byte output is compatible with **81–88 bytes** of input. [RFC 5649](https://www.rfc-editor.org/rfc/rfc5649)

**Assessment:** both fit the size. Neither is established by that fit.

---

## 14. Base64-Level Structure

The visible encoding uses **57 of 64 symbols**, with an index of coincidence of **0.0139025591**. The digrams `Ab`, `BH`, and `HH` each occur twice; no trigrams repeat.

The first three symbols fix 18 decoded bits. A chosen visible prefix can belong to an exposed IV, nonce, salt, or other field; it cannot identify that field's role. Clear field bytes can be selected directly, whereas targeting an encrypted-output prefix may require varying an input.

Tests of alphabet-index shifts and reflections on the original, reversed, and case-swapped string yielded no entirely printable decoded output. These tests do not cover an arbitrary permutation of the Base64 alphabet, nor arbitrary compositions of transformations.

Deleting a non-multiple-of-four number of Base64 symbols shifts decoding boundaries. Such deletion should not be treated as removal of an ordinary byte header.

---

## 15. Format and Container Tests

| Specific representation | Finding and boundary |
|---|---|
| Immediate ASCII hex or another ordinary Base64 text layer | Excluded: decoded bytes do not have the required textual alphabet |
| Entire decoded UTF-8 prose | Excluded as valid UTF-8; other character encodings can decode arbitrary binary and require semantic assessment |
| Standard Fernet | Excluded by version `69` instead of `80`, and independently by length: decoded Fernet length is `57 + 16k`, not 96 |
| OpenSSL-style envelope beginning `Salted__` | Prefix absent; raw OpenSSL output or an explicitly supplied salt is not excluded |
| Compact JWT/JWE serialization | Neither supplied string nor decoded bytes has the required segmented textual form |
| One complete DER object starting at offset zero | Outer tag `69` and short length `18` describe only 26 bytes, not the entire 96 |
| Complete OpenPGP packet stream beginning at zero | First byte `69` lacks the required packet-header high bit |
| Common files requiring gzip/ZIP/PNG/JPEG/PDF/XZ headers | Required start signatures absent; this says nothing about embedded fragments or encrypted files |

Fernet's length rule follows from version + timestamp + IV + padded AES ciphertext + HMAC. Replacing URL-safe characters alone would not fix this artifact. [Fernet specification](https://github.com/fernet/spec/blob/master/Spec.md). OpenPGP packet-header requirements are specified in [RFC 9580](https://www.rfc-editor.org/rfc/rfc9580).

---

## 16. Public-Key and Signature Hypotheses

| Candidate | Result |
|---|---|
| Raw P-384 `x || y`, 48-byte fields | Coordinates are in range but fail the curve equation, in both big- and little-endian interpretations |
| Raw P-384 ECDSA `r || s` | Both scalars are in range in both byte orders; nearly every random pair would pass, so this provides negligible positive evidence |
| Ed25519 `public key || R || S` | Final 32-byte little-endian S is out of range |
| Ed25519 `R || S || public key` | Middle 32-byte little-endian S is out of range |
| Standard compressed BLS12-381 G2, 96 bytes | Compression flag absent in `69`; excluded in the referenced serialization |
| Standard uncompressed BLS12-381 G1, 96 bytes | Flags indicate an invalid infinity/sign combination with nonzero payload; also excluded |
| Canonical X25519 public key in first 32 bytes, e.g. untouched libsodium sealed-box prefix | Byte 31 is `c7`, so the little-endian field element is noncanonical; incompatible with a canonically generated prefix |
| RSA ciphertext in a 96-byte modulus-width representation | Size-compatible; no modulus or exponent is supplied, so no RSA computation was attempted |
| Concatenated hashes/MACs, XOF output, random key material | Compatible; these may be verifiers or key material rather than encrypted prose |

P-384 checks use the parameters in [SEC 2](https://www.secg.org/sec2-v2.pdf). Ed25519 requires `S < L` under [RFC 8032](https://www.rfc-editor.org/rfc/rfc8032). BLS statements are scoped to the [blst/Zcash-compatible serialization](https://github.com/supranational/blst#serialization-format), not every custom BLS encoding.

The X25519 distinction needs care: [RFC 7748](https://www.rfc-editor.org/rfc/rfc7748) requires receivers to mask the high bit and accept some noncanonical inputs. This observation excludes an **unaltered canonical generated prefix**, not every blob a permissive receiver might accept. [Libsodium sealed boxes](https://doc.libsodium.org/public-key_cryptography/sealed_boxes) prepend an ephemeral public key, making that representation worth checking. No recipient-key decryption was performed.

---

## 17. Hashes, MACs, and Key Material

The 96 bytes can be partitioned as six 16-byte values, three 32-byte values, or two 48-byte values. These sizes are compatible with collections of hashes, MACs, keys, or other fixed-size binary objects. An extendable-output function can also produce 96 bytes directly.

A bare hash or signature is not ordinarily an encrypted message. Recovery would require a different approach, such as testing candidate inputs or using the value within a larger protocol.

**Assessment:** these possibilities remain compatible with the observed statistics. Their likelihood depends on information not present in the bytes.

---

## 18. Padding

In CBC and ECB, padding is applied before encryption. The final ciphertext bytes therefore cannot reveal whether PKCS#7, zero padding, or another convention was used.

For PKCS#7 on a 16-byte block cipher, valid padding is one of:

```text
01
02 02
03 03 03
...
10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10
```

Testing is meaningful after decryption. A padding pass alone is not authentication. Related key/layout trials may share the same final-block computation, so their padding outcomes need not be statistically independent.

---

## 19. IV, Nonce, Salt, and Tag Placement

A random-looking segment does not reveal whether it is an IV, nonce, salt, ciphertext, or tag. In particular, small variations in byte means or sample entropy across candidate segments cannot establish their boundaries.

Compatible arrangements include:

```text
IV || ciphertext
nonce || ciphertext || tag
nonce || tag || ciphertext
salt || IV || ciphertext
salt || nonce || ciphertext || tag
ciphertext || MAC
ciphertext with external parameters
```

Each arrangement defines a separate test. There is no observable boundary marker in this artifact that selects one of them.

---

## 20. Key Size and Key Derivation

AES uses 16-byte blocks with 16-, 24-, or 32-byte keys. Output alignment therefore does not distinguish AES-128, AES-192, and AES-256.

The artifact also does not identify whether key material was raw bytes, a hash of a password, or the output of PBKDF2, scrypt, Argon2, HKDF, or another derivation.

A password test must specify:

- Exact input bytes and character encoding.
- Normalization, whitespace, punctuation, and newline handling.
- KDF and all of its parameters.
- Salt bytes and location.
- Derived output length and division into keys or IVs.
- Cipher, mode, nonce, tag, and associated data.

A failed decryption excludes the complete tested parameter tuple. It does not rule out the password under every derivation.

---

## 21. Authentication and Candidate Verification

### Authenticated encryption

For a correctly specified authenticated construction, tag verification is a strong way to test a candidate key. Associated data must also be correct. Empty and nonempty associated data are different parameterizations.

### CBC and ECB

Use padding, text encoding, expected syntax, and overall coherence together. Short printable fragments occur by chance in large searches. Padding validity alone is insufficient.

### Stream-like modes

CTR, CFB, OFB, and unauthenticated stream encryption produce some output under every key. Candidate assessment depends on independently expected plaintext properties.

### Re-encryption

Re-encryption under the same parameters should reproduce the ciphertext. For unauthenticated modes this is an implementation consistency check, not independent evidence that the key is correct: an arbitrary key can also round-trip its own decryption.

---

## 22. Excluded Representations

“Excluded” means incompatible with the specific representation and assumptions stated here.

| Representation | Basis |
|---|---|
| Immediate ASCII plaintext or valid UTF-8 text | Decoded bytes fail the required encoding conditions |
| Immediate ASCII hex or ordinary second Base64 layer | Required alphabet absent |
| Full-payload repeating XOR of 7-bit plaintext, key periods 1–94 | Exact high-bit periodicity contradiction |
| Tested simple transforms producing entirely printable ASCII plus tab/LF/CR | Exhaustive finite sweeps produce none |
| Tested complete compression streams and byte-aligned suffix streams | Decoder failures |
| Standard Fernet | Version and length contradictions |
| Envelope requiring an initial `Salted__` marker | Marker absent |
| Compact JWT/JWE serialization | Textual structure absent |
| One complete DER object starting at offset zero | Outer length does not consume the payload |
| Complete OpenPGP packet stream starting at offset zero | Invalid leading packet-header bit |
| Raw P-384 point in the tested byte orders | Curve-equation failure |
| Two standard Ed25519 key/signature concatenations | Signature scalar out of range |
| Referenced standard BLS compressed-G2 and uncompressed-G1 encodings | Invalid metadata bits |
| Unaltered canonical X25519 public key as the first 32 bytes | Noncanonical field encoding |

These exclusions do not extend to arbitrary preprocessing, extracted fragments, nonstandard serializations, or different field boundaries.

---

## 23. Still-Compatible Constructions

The following remain compatible with the basic artifact, subject to their required parameters:

```text
AES-CBC and AES-ECB
AES-GCM, AES-SIV, and AES-GCM-SIV
AES-CTR, CFB, and OFB
CCM and OCB
ChaCha20-Poly1305 and XChaCha20-Poly1305
XSalsa20-Poly1305 secretbox-style constructions
Other 128-bit-block ciphers
8-byte-block legacy ciphers
Encrypt-then-MAC arrangements
AES Key Wrap and Key Wrap with Padding
Password-derived encryption
Raw P-384 ECDSA signatures
RSA ciphertext in a 96-byte representation
Hashes, MACs, XOF output, and key material
Custom constructions
Compression or additional encoding before encryption
Externally supplied IVs, nonces, salts, or associated data
```

Compatibility is not a claim that these alternatives are equally common, equally secure, or equally useful. The artifact alone does not provide a calibrated likelihood ranking.

---

## 24. Recommended Testing Strategy

### 24.1 Fix the representation

Preserve the exact Base64 string and verify the decoded checksum before testing. Define field boundaries explicitly rather than repeatedly changing them in response to partial output.

### 24.2 Derive candidate keys

For an independently motivated candidate, test raw bytes at valid key lengths and any supported derivation. Hash-based, padded, or truncated variants can be useful implementation hypotheses; expensive KDF searches should use justified salt and parameter ranges.

### 24.3 Test authenticated layouts

Prioritize layouts that provide a strong success criterion: for example, GCM with a 12-byte prefix nonce and final 16-byte tag, SIV with a 16-byte synthetic IV, or a specified ChaCha/Poly1305 representation. Record associated data and exact serialization.

### 24.4 Test CBC and stream-like modes

For CBC, test the proposed IV boundary and inspect later blocks even when the external IV is unavailable. Evaluate padding only after decryption. For CTR, record counter structure and byte order. CFB segment size is also part of the construction.

### 24.5 Validate the complete output

Evaluate the entire result against the expected encoding or format. A successful parser or authentication check is stronger than isolated recognizable words. Retain exact inputs and parameters so the result can be reproduced.

### 24.6 Record search limits

An exhausted finite family is a useful result. It does not exhaust arbitrary compositions, all passwords, all KDF settings, or all cipher implementations.

---

## 25. Information From Additional Ciphertexts

A second sample from the same generating process can provide evidence unavailable from one blob.

| Observation | Possible interpretation |
|---|---|
| Repeated prefix | Fixed header, reused IV/nonce, deterministic metadata, or duplicate content |
| Repeated aligned blocks | ECB, matching CBC prefixes under a fixed IV, or duplicated data |
| Lengths consistently varying in 16-byte steps | Support for padded block encryption or fixed-size framing |
| Lengths varying byte-for-byte | Support for stream-like encryption or variable-length authenticated output |
| All outputs exactly 96 bytes | Fixed-size objects, records, or padded slots |
| Confirmed same stream key and nonce | Ciphertext XOR cancels the repeated keystream |

A shared prefix alone does not prove nonce reuse. Comparisons require consistent provenance, field alignment, and a supported common-construction hypothesis.

---

## 26. Analysis Limits

The quantitative results cover this exact 96-byte sample, a 50,000-sample statistical calibration, specified format checks, and **74,004 simple-transform trials**. These transform trials overlap and are not independent evidence.

No complete ciphertext-only search can exhaust unrestricted keys, algorithms, custom encodings, or parameter choices. Compatibility tables list possible constructions; they do not imply that every listed construction has been exhaustively attacked.

The bytes alone do not determine:

1. Whether the payload is an encrypted message.
2. The cipher, mode, or key size.
3. IV, nonce, salt, or tag placement.
4. The KDF or its parameters.
5. Whether compression or another layer precedes encryption.
6. Whether external parameters or associated data are required.
7. Whether a key is random or derived from a weak password.
8. The intended plaintext encoding, length, or format.

---

## 27. Final Conclusions

| Conclusion | Assessment |
|---|---|
| Valid canonical standard Base64 | Directly verified |
| Decoded length is 96 bytes | Directly verified |
| Tested low-order statistics are compatible with uniform bytes | Supported by finite-sample calibration |
| Source entropy or encryption status is established | Not determined |
| Full-payload repeating XOR of 7-bit text, periods 1–94 | Excluded exactly |
| Tested simple transforms recover printable text | No |
| Specific compression and container representations | Excluded within stated boundaries |
| AES-CBC, AES-GCM, and other listed constructions fit | Yes |
| One cipher or mode has been identified | No |
| First 16 bytes are an IV or final 16 bytes are a tag | Not determined |
| A verified plaintext has been recovered | No |

The artifact is a well-defined binary target with several exact exclusions and many compatible cryptographic interpretations. There is no demonstrated ciphertext-only weakness. Progress toward recovery depends on specifying a candidate construction and key material, obtaining related samples, or identifying structure in the generating process.

---

## Appendix A — Exact Decoded Blocks

```text
B1 = 6918328cf8b29c06c943528f0085a805
B2 = 9bb180abb3057e1aa9f93c5da26c2dc7
B3 = c9488db29a8e406f058f244fa6b8c0bc
B4 = 64fd860471c787d72074d8c86c3a7fc4
B5 = 5a79c4f93dd45d2ef25963dd7ee6a53c
B6 = c6ccc740d3e6cb6a518aaf8047618e24
```

---

## Appendix B — Measurement Conventions

Byte offsets are zero-based decimal unless explicitly marked hexadecimal. Bits are read MSB-first for the runs count. Printable ASCII means bytes `0x20`–`0x7e`; transform scoring additionally allows tab, LF, and CR.

The random-sample calibration used seed `20260911` with NumPy 2.5.1 on Python 3.14.7. The reported intervals describe the simulated null distribution. The lag p-value includes selection over byte lags 1–32.

Decompression attempts were limited to 1 MiB of output, with a 64 MiB memory limit for the LZMA decoder. No tested suffix reached end-of-stream. The XZ recompression result uses the default container format.

Empirical entropy, index of coincidence, and χ² are descriptive statistics of this sample. They do not directly estimate cryptographic security or establish the entropy of the generating process.
