# Sign in With Stellar

## Preamble

```
SEP: To Be Assigned
Title: Sign in with Stellar
Author: Mahmoud Alkhraishi <@mkhraisha>
Track: Informational
Status: Draft
Created: 2024-07-03
Updated: 2024-07-03
Version: 0.1.0
Discussion: N/A
```

## Simple Summary

Sign in with Stellar provides a standard way for users to sign in to websites using their Stellar Account. This is a profile of the [SEP-10](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0010.md) Authentication intended to provide a simple and secure way for users to sign in to websites using their Stellar account without the need for a backend server.

## Dependencies

SEP-10 is a required dependency for this SEP.

## Motivation

[Sign in with Ethereum](https://login.xyz/) provides an easy way to sign in to websites using your Ethereum account. [SEP-10](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0010.md) attempts to provide a similar ability for Stellar users, however because it does not have a simple predefined flow for a wallet to access a DAPP, it is not as simple as Ethereum. This SEP attempts to provide a simple and secure way for users to sign in to websites using their Stellar account.

## Abstract

Sign in with Stellar enables a wallet to sign a challenge from a website to prove that the user owns a Stellar account. This is done by the website providing a challenge to the wallet, the wallet signing the challenge with the user's Stellar account, and the wallet returning the signature to the website. The website can then verify the signature to prove that the user owns the Stellar account. A backend server is not required for this flow, as the website can verify the signature directly. It also removes the requirements for an invalid transaction to be created, as the signature can be verified directly. It also provides a simple library to make it easy for wallets and websites to implement this flow.

## Specification

This section outlines the steps required for implementing the "Sign in with Stellar" process, including detailed format specifications for the challenge and response, as well as examples.

### 1. Challenge Generation

The website generates a random challenge. The challenge must be a cryptographically secure random string, ensuring it is unique for each request.

**Challenge Format:**

```json
{
  "challenge": "random_challenge_string",
  "timestamp": "ISO_8601_timestamp",
  "domain": "example.com"
}
```

- `challenge`: A cryptographically secure random string.
- `timestamp`: The current timestamp in ISO 8601 format.
- `domain`: The domain of the website issuing the challenge.

### 2. Sending Challenge to Wallet

The website sends the challenge to the wallet. This can be done through various methods such as a QR code, deep link, or directly if the wallet is integrated into the website.

### 3. Wallet Signs the Challenge

The wallet signs the challenge with the user's Stellar account. The signature is generated using the private key of the user's Stellar account.

**Signing Process:**

1. The wallet receives the challenge JSON.
2. The wallet uses the private key to sign the `challenge` field.

**Response Format:**

```json
{
  "public_key": "G...ABC", // User's Stellar public key
  "signature": "base64_encoded_signature"
}
```

- `public_key`: The public key of the user's Stellar account.
- `signature`: The signature of the `challenge` field, encoded in base64.

### 4. Returning Signature to Website

The wallet returns the signature to the website. This can be done through a callback URL, or if the wallet is integrated into the website, it can be returned directly.

### 5. Verifying the Signature

The website verifies the signature to prove that the user owns the Stellar account.

**Verification Steps:**

1. The website retrieves the user's public key from the response.
2. The website verifies the signature using the public key and the original `challenge` field.

**Verification Example (Pseudo-code):**

```python
import stellar_sdk

def verify_signature(public_key, challenge, signature):
    try:
        keypair = stellar_sdk.Keypair.from_public_key(public_key)
        return keypair.verify(challenge.encode(), base64.b64decode(signature))
    except Exception as e:
        return False

public_key = "G...ABC"
challenge = "random_challenge_string"
signature = "base64_encoded_signature"

if verify_signature(public_key, challenge, signature):
    print("Signature is valid.")
else:
    print("Signature is invalid.")
```

### Error Handling

- **Invalid Signature**: If the signature verification fails, the website should reject the login attempt and prompt the user to try again.
- **Expired Challenge**: If the challenge is too old (e.g., more than 5 minutes), the website should reject it and issue a new challenge.

### Security Considerations

- **Replay Attacks**: Ensure that each challenge is used only once by invalidating the challenge after it is used.
- **Challenge Expiry**: Implement a time limit for the challenge validity to prevent reuse of old challenges.
- **Transport Security**: Use HTTPS to secure the communication between the website and the wallet.

## Design Rationale

This section provides the reasoning behind the design choices made for the "Sign in with Stellar" process and discusses the alternatives considered.

### Simplicity and Security

The design prioritizes both simplicity and security by using a straightforward challenge-response mechanism. This approach avoids the complexity of requiring a backend server to verify transactions, as the website can directly verify the signature.

### Compliance with SEP-10

Implementers of this profile should be compliant with SEP-10. By adhering to the SEP-10 standard, this proposal ensures compatibility with existing systems and simplifies the implementation for developers already familiar with SEP-10.

### Alternatives Considered

1. **Using Full Transaction Signing**:

   - **Pros**: Fully aligns with SEP-10's original design.
   - **Cons**: More complex to implement and requires creating and managing transactions, which can be cumbersome for simple authentication purposes.

2. **OAuth-like Flow with Backend Server**:

   - **Pros**: Familiar approach for many developers.
   - **Cons**: Requires additional infrastructure (backend server) and introduces more points of failure and security considerations.

3. **Direct Public Key Exchange**:
   - **Pros**: Simplest possible method.
   - **Cons**: Lacks the security of a challenge-response mechanism and is vulnerable to man-in-the-middle attacks.

### Chosen Design

The chosen design strikes a balance by providing a secure and verifiable method without requiring a backend server. By using a challenge-response mechanism, it ensures that the authentication process is both secure and straightforward to implement.

### Key Consideration

Implementers should use standardized messages and ensure that the challenge is cryptographically secure. This will facilitate easier adoption and implementation while maintaining high security standards.

## Backwards Compatibility

This section discusses the compatibility of the "Sign in with Stellar" process with existing systems and standards.

### Compatibility with Existing Systems

The "Sign in with Stellar" process is designed to be fully compatible with existing Stellar accounts and infrastructure. Since it builds on the SEP-10 standard, any systems already implementing SEP-10 can adopt this proposal with minimal changes.

### Minimal Impact on Wallets

Wallets that already support SEP-10 can implement this proposal with minimal additional development. The main requirement is the ability to handle and sign a cryptographic challenge, which is a basic feature of most cryptocurrency wallets.

### No Breaking Changes

This proposal does not introduce any breaking changes to the Stellar network or existing SEP-10 implementations. It is an additional profile of SEP-10 designed to simplify and streamline the authentication process for web applications.

### Incremental Adoption

Adopting this proposal can be done incrementally. Websites and wallets can start supporting the "Sign in with Stellar" process without disrupting existing functionality or requiring immediate widespread adoption.

### Summary

The "Sign in with Stellar" proposal ensures backward compatibility by building on existing standards and requiring no changes to Stellar accounts or network infrastructure. It provides a seamless and incremental upgrade path for wallets and web applications.

## References

- **SEP-10: Stellar Web Authentication**: [https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0010.md](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0010.md)
- **Sign in with Ethereum**: [https://login.xyz/](https://login.xyz/)

## Changelog

- `0.1.0`: Initial draft version of the "Sign in with Stellar" proposal.
