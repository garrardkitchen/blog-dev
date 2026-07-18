---
aliases: [/blog/digital-certificates/]
title: "Digital Certificates"
date: 2020-06-08T12:57:32+01:00
description: "How hashes, encryption, digital signatures, X.509 certificates and signed JWTs fit together—and where their responsibilities differ."
summary: "A practical guide to the boundaries between encoding, hashing, encryption, signing and certificates, with modern OpenSSL examples and an accurate explanation of JWT verification."
tags: [engineering, security, cryptography, openssl, "digital certificate", "digital signature", jwt, x509]
---

Digital certificates become confusing when several different ideas are compressed into one vague notion of “security”. Hashing, encryption, signing and certificates are related, but they solve different problems. A JWT adds another layer of terminology while still relying on the same underlying primitives.

The cleanest way through the subject is to keep four questions separate:

| Question | Mechanism |
|---|---|
| Did these bytes change? | A cryptographic hash can detect a change, provided the expected hash is obtained securely. |
| Can anyone else read these bytes? | Encryption provides confidentiality. |
| Did the holder of a particular private key approve these bytes? | A digital signature provides integrity and proof of possession of that private key. |
| Why should I associate that public key with this server or organisation? | A certificate and a trust-validation process bind a public key to an identity or other attributes. |

None of those answers automatically supplies the others. Encryption does not prove who created a message. A signature does not hide it. A certificate does not make every use of its key trustworthy.

## Start with the building blocks

### Encoding is not encryption

Base64 converts bytes into text that is easier to carry through text-oriented systems. Base64url is a URL-safe variation used by JWTs. Both are reversible without a key, so neither provides confidentiality or integrity.

If you paste a signed JWT into a decoder, its header and payload are normally readable. That is expected behaviour, not a security flaw. A signed JWT is protected against undetected modification; it is not encrypted.

### A hash is a fingerprint of some bytes

A cryptographic hash function accepts an input of any practical size and produces a fixed-size digest. SHA-256, for example, produces 256 bits. Change the input and the digest should change unpredictably.

Hashes are one-way in the practical sense: there is no decryption key that recovers the input. They are not guaranteed to be unique, because infinitely many possible inputs map to a finite set of digests, but a modern cryptographic hash is designed to make finding a collision computationally infeasible.

A bare hash does not authenticate anything. An attacker who can replace a file can usually replace an adjacent hash as well. The expected digest must arrive through a trusted channel, be protected by a MAC, or be digitally signed.

### Encryption protects confidentiality

Symmetric encryption uses the same secret key to encrypt and decrypt. It is efficient and is the normal choice for protecting bulk data. Modern applications should use authenticated encryption such as AES-GCM or ChaCha20-Poly1305, which detects tampering as well as hiding the plaintext.

Asymmetric cryptography uses a related public/private key pair. Its operations depend on the algorithm: a key pair may support signatures, key agreement, encryption, or some combination. “Encrypt with either key and decrypt with the other” is not a safe general model of public-key cryptography.

Real protocols rarely encrypt a large message directly with RSA. They normally use a hybrid design: asymmetric cryptography establishes or protects a random symmetric key, and authenticated symmetric encryption protects the data. TLS is the familiar example. Application code should normally use a reviewed protocol or a high-level cryptographic API rather than inventing this construction.

### A digital signature protects integrity and establishes key possession

To sign a message, the sender uses a private key and a signature algorithm. The recipient uses the corresponding public key to verify the signature.

Conceptually, many signature schemes hash the message and apply a private-key operation to an encoded form of that digest. The exact construction matters, however, so it is better to think in terms of a named signature algorithm—such as RSA-PSS with SHA-256 or ECDSA with SHA-256—than “encrypting a hash with the private key”.

A successful verification establishes two things:

1. The signed bytes have not changed since the signature was created.
2. Whoever created the signature possessed the private key corresponding to the verification key.

It does **not**, by itself, establish who owns that key. That missing association is the problem certificates address.

## What an X.509 certificate contains

An X.509 certificate is a signed data structure. Among other fields, it contains:

- the subject public key;
- an issuer and subject;
- a serial number;
- a validity interval;
- extensions that constrain how the certificate may be used;
- the issuer's signature over the certificate.

For a typical public HTTPS site, the server presents its leaf certificate and usually the intermediate certificates needed to build a path towards a trusted root. The client then performs certificate-path validation. That includes checking signatures, validity dates, constraints and intended usage. For HTTPS it also verifies that the requested DNS name is covered by the certificate's Subject Alternative Name extension. Revocation checking depends on the client and its policy.

The root certificate is trusted because it is already in the client's trust store, not because it signs itself. A self-signature alone creates no external trust.

This leads to an important distinction:

- A **self-signed certificate** can be useful in development or in a system where its exact certificate or public key is distributed through a trusted channel.
- A **publicly trusted certificate** is useful when clients need to build a chain to a certificate authority they already trust.

Private systems do not always need a public certificate authority. Pinning a particular certificate, running a private PKI, or registering a public certificate against an application identity can all be valid trust models. The relying party still needs an explicit reason to trust the binding.

## A practical signing example with OpenSSL

The following commands demonstrate signatures and certificates. They are learning aids, not a complete key-management design. They assume OpenSSL 3 and a shell in which `openssl` is available.

First, create an encrypted RSA private key:

```shell
openssl genpkey \
  -algorithm RSA \
  -aes-256-cbc \
  -pkeyopt rsa_keygen_bits:3072 \
  -out signer-private.pem
```

OpenSSL prompts for a passphrase. In production, private keys should be generated, stored and used in an appropriate secret store, key vault, hardware security module or platform keystore. Do not commit this file or its passphrase to source control.

Extract the public key:

```shell
openssl pkey \
  -in signer-private.pem \
  -pubout \
  -out signer-public.pem
```

Create a file named `message.txt`, then sign its exact bytes with SHA-256 and RSA:

```shell
openssl dgst \
  -sha256 \
  -sign signer-private.pem \
  -out message.sig \
  message.txt
```

The signature is binary. It can be transported as Base64 when a text representation is required, but that encoding does not change its security properties.

Verify the signature with the public key:

```shell
openssl dgst \
  -sha256 \
  -verify signer-public.pem \
  -signature message.sig \
  message.txt
```

The result should be `Verified OK`. Change even one byte in `message.txt` and verification should fail.

Notice what this example has and has not proved. It shows that the signature matches the message and public key. It says nothing about whether `signer-public.pem` really belongs to the person or service you intended to trust. Receiving the message, signature and public key from the same untrusted source would not solve that problem.

## Put the public key in a certificate

We can create a short-lived, self-signed certificate around the same public key:

```shell
openssl req \
  -new \
  -x509 \
  -key signer-private.pem \
  -sha256 \
  -days 30 \
  -subj "/CN=signer.example.test" \
  -addext "basicConstraints=critical,CA:FALSE" \
  -addext "keyUsage=critical,digitalSignature" \
  -out signer-certificate.pem
```

Inspect it:

```shell
openssl x509 \
  -in signer-certificate.pem \
  -text \
  -noout
```

Extract its public key:

```shell
openssl x509 \
  -in signer-certificate.pem \
  -pubkey \
  -noout \
  -out certificate-public.pem
```

That extracted key can verify the original signature:

```shell
openssl dgst \
  -sha256 \
  -verify certificate-public.pem \
  -signature message.sig \
  message.txt
```

The certificate has made the key portable and attached metadata and usage constraints to it. It has not created trust from nowhere. Because the certificate is self-signed, a relying party must receive and trust it through some separate, secure process.

Nor is this certificate suitable for an HTTPS server as written. A TLS server certificate needs the appropriate extended key usage and a Subject Alternative Name containing the server's DNS name or IP address. Certificate profiles are use-case specific.

## How this relates to JWTs

A JSON Web Token is a compact way to carry claims. JWT describes the claims container; JSON Web Signature (JWS) describes the signed representation commonly used for access and identity tokens.

A compact, signed JWT has three Base64url-encoded parts:

```text
header.payload.signature
```

For JWS compact serialization, the signing input is exactly:

```text
BASE64URL(UTF8(protected header)) + "." + BASE64URL(payload bytes)
```

The signer applies the selected JWS algorithm to those bytes and Base64url-encodes the resulting signature. With `RS256`, that algorithm is RSASSA-PKCS1-v1_5 using SHA-256. The private key signs; the public key verifies.

The verifier does not decrypt the signature to recover the payload, and it never needs the private key. It:

1. parses the protected header;
2. selects a trusted public key according to the application's issuer and key-discovery rules;
3. verifies the signature over the original encoded header and payload;
4. validates the claims required by the application.

Step four is essential. A cryptographically valid token may still be unacceptable. A resource server will commonly validate:

- the algorithm against an explicit allow-list;
- the issuer (`iss`);
- the audience (`aud`);
- expiry (`exp`) and, where applicable, not-before (`nbf`);
- token type, scopes, roles or other application-specific claims.

The token's `alg` header describes how its signature was created; it must not be allowed to dictate verification policy by itself. Likewise, a key identifier such as `kid` helps choose among already trusted keys—it does not make an arbitrary key trustworthy.

A certificate may participate in JWT signing, but it is not required by the format. An identity provider might publish bare public keys as a JSON Web Key Set, publish an X.509 chain, or register a certificate against a client identity. The trust model belongs to the surrounding protocol and deployment, not to JWT alone.

Finally, because the header and payload are encoded rather than encrypted, never place secrets in an ordinary signed JWT. When confidentiality is required, use an appropriate encrypted-token design such as JWE or, more commonly, keep sensitive data on the server and put only a reference in the token.

## The boundaries worth remembering

- Base64 makes bytes printable. It provides no security.
- A hash detects a change only when the expected digest is itself trusted.
- Encryption hides data. Use authenticated encryption for application data.
- A signature protects signed bytes and proves possession of a private key. It does not hide the bytes.
- A certificate binds a public key to names and constraints. Trust comes from validating that binding against a trust anchor and a policy.
- A signed JWT is normally readable. Verification requires both a valid signature and valid claims.
- A public key is not confidential; a private key is. Protecting the private key is the foundation on which every later assertion rests.

The most useful question in any certificate design is therefore not “is it encrypted?” It is: **what security property do I need, which key establishes it, and why does the relying party trust that key?**

## References

- [RFC 5280: Internet X.509 Public Key Infrastructure Certificate and CRL Profile](https://www.rfc-editor.org/rfc/rfc5280)
- [RFC 7515: JSON Web Signature](https://www.rfc-editor.org/rfc/rfc7515)
- [RFC 7519: JSON Web Token](https://www.rfc-editor.org/rfc/rfc7519)
- [RFC 8725: JSON Web Token Best Current Practices](https://www.rfc-editor.org/rfc/rfc8725)
- [OpenSSL `dgst` documentation](https://docs.openssl.org/3.5/man1/openssl-dgst/)
- [OpenSSL `pkey` documentation](https://docs.openssl.org/3.5/man1/openssl-pkey/)
- [OpenSSL `req` documentation](https://docs.openssl.org/3.5/man1/openssl-req/)
- [OpenSSL `x509` documentation](https://docs.openssl.org/3.5/man1/openssl-x509/)
