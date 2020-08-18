
# Vaultie DID Method Specification
v0.0.1, Dmitry Semenovskiy, Vaultie Inc.
## Introduction
Vaultie aims to create a fully verifiable blockchain-based digital signatures that are both legally binding and link to real identities. At its core Vaultie uses Ethereum blockchain to anchor the signature, conforming to W3C standards (https://w3c-dvcg.github.io/lds-merkle-proof-2019/) and IPFS as the data storage.
### Ethereum
Ethereum is an open source, public, blockchain-based distributed computing platform and operating system featuring Smart Contracts functionality. There is an ability to anchor data using an Ethereum (or other EVM-compatible) blockchain transaction's input.
### IPFS
The InterPlanetary File System (IPFS) is a protocol and peer-to-peer network for storing and sharing data in a distributed file system. IPFS uses content-addressing to uniquely identify each file in a global decentralized file-system. Due to its content-addressing nature it is not possible to change the file contents without changing its address, thus we can consider files stored on IPFS immutable.
## Overview
Vaultie DID method uses IPFS as a decentralised storage for DID Documents. An Ethereum transaction, that does not require any additional Smart Contracts, provides a mapping from a DID to an IPFS hash address of the corrosponding DID Document. This enables DID Documents on IPFS to be effectively addressed via their DIDs. While this method requires additional step in order to lookup DID Document, the method is much more cost effective than using Smart Contracts and Ethereum's expensive storage.
## Specification
### Method DID Format
Vaultie DIDs are identifiable by their did\:vaultie: method string and conform to the [Generic DID Scheme](https://w3c-ccg.github.io/did-spec/#the-generic-did-scheme).
### DID Creation
The creation of a DID follows a few steps:
1. Generate at least 32 bytes of entropy
2. From the entropy, generate an Ethereum key pair
3. From the Ethereum key pair, take the Keccak-256 hash of the public key.
4. Take the last 40 bytes of the hash from step 3.
The result from step 4 is your DID and Ethereum wallet address.
### DID Registration/Anchoring
DID Anchoring refers to creating the mapping from DID to IPFS address on the blockchain by sending a transaction with the input data containing Keccak-256 hash of your DID: `did:vaultie:<ethereum_address>` and hexadecimal representation of DID Document's IPFS address.
This process anchors the DID on the blockchain and makes its DID Document accessable with only this DID.
### DID Document Resolution
DID Document resolution is achieved by querying the transactions by DID's unique identifier part (Ethereum wallet address) and filtering them according to the method of creation:
1. The transaction's `from` and `to` should match. In other words, the transaction must be sent to yourself.
2. Order matters. To effectively determine the latest version of DID Document start with newer transactions.
3. The transaction's input data must begin with Keccak-256 hash of `did:vaultie:<ethereum_address>`
4. Keccak-256 hash of DID and IPFS address must be separated.
5. Hex to Base58 encoding should be applied to the resulting string.
### DID Document Updating and Deleting
IPFS addresses are base58 representation of hashes of their content, so an updated DID Document will also have a new IPFS address.
Updates of DID Documents are performed via the same method as creation, except for the new IPFS address.
Deletion of DID Document is performed by updating transaction using an all-0 byte string as IPFS address.
## Key Management
### Key Recovery
Writing to Ethereum blockchain requires control over the private key used to anchor the DID, so any key recovery method which applies to Ethereum keys can be applied here. This can involve seed phrases stored on an analog medium, USB cold-storage wallets, and etc.
### Key Revocation
Due to decentralized nature of blockchain, no party can revoke the keys controlling DID of another party in Ethereum ecosystem.
## Privacy and Security Considerations
### Key Control
As mentioned in the Key Recovery section, the entity which controls the private key which anchored the DID also effectively controls the DID Document which the DID resolves to. Thus great care should be taken to ensure that the private key is kept private. Methods for ensuring key privacy are outside the scope of this document.
### DID Document Public Profile
The DID Document anchored with the registry contract can contain any content, though it is recommended that it conforms to the [W3C DID Document Specificaiton](https://w3c-ccg.github.io/did-spec/#did-documents). As registered DIDs can be resolved by anyone, care should be taken to only update the registry to resolve to DID Documents which DO NOT expose any sensitive personal information, or information which you may not wish to be public.
### IPFS and Canonicity
IPFS allows any entity to store content publically. A common misconception is that anyone can edit content, however the content-addressing nature of the storage platform means that this new content will have a different address to the original. Therefore, while any entity can copy and modify an anchored DID Document, they cannot change the document that a DID resolves to via the blockchain unless they control the private key which is associated with it.
