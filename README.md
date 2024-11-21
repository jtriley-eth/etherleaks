# How To Leak a Secret 2, Electric Boogaloo

Ring signatures allow for a group of N members to generate a 1-of-N signature such
that no information about _which_ member of the group signed a given message.
However, this depends on public key infrastructure, fortunately Ethereum
provides public key infrastructure especially for Ethereum-related teams.
Unfortunately, ring signature support would require both an interface
implementation from wallet providers as well as other members of the group to
sign at least one message up front (explained [here](#ethereum-public-keys)),
requiring buy-in from the whole group.

As leaking secrets are generally against the interest of the opposition, buy-in
may not be feasible, thus we define a noir-circuit such that the signature may
be generated through existing wallet interfaces and ethereum addresses may be
used, rather than ethereum wallet public keys

## Implementation

The interface is as follows.

```rs
fn main(
    message_hash: pub [u8; 32],
    members: pub [Address; 8],
    signer_pub_x: [u8; 32],
    signer_pub_y: [u8; 32],
    signature: [u8; 64],
) {
    // -- snip
}
```

Note that the message hash and Ethereum address array are both public inputs,
as these should be publicly accessible. The signer's public key and signature,
however, are private inputs, to anonymize the signer.

Procedures:

1. Assert the message and signature correspond to the public key
2. Compute the signer's Ethereum address from the public key
3. Assert the signer's Ethereum address is in the members array

## Limitations

Current implementation has no linkability, ie no way to link two signatures to
the same singer like crypto-note.

## Ethereum Public Keys

On Ethereum, wallet addresses are actually the first 20 bytes of the keccak256
hash digest of secp256k1 ellptic curve points representing public keys. That is
to say that the Ethereum address itself leaks no information about the owner's
corresponding public key. If the address has signed at least one message,
however, the public key can be extracted from that signature, but not all
Ethereum wallets have signed a transaction or otherwise publicly available
message.

```py
# pseudocode
private_key = 0x...
message = 0x...

public_key = ec_mul(private_key, secp256k1_generator_point)

ethereum_address = keccak256(concat(public_key.x, public_key.y))

signature = sign(private_key, message)

extracted_public_key = ecrecover(signature, message)
```
