# Starklane NFT Bridge

A seamless bridge for NFT transfers between Ethereum Layer 1 and StarkNet Layer 2.

## Introduction

As the StarkNet ecosystem prepares for rapid growth in the upcoming months, it is essential to develop key infrastructures that will support this business development phase. The **Starklane NFT Bridge** is one such infrastructure.

The **Starklane NFT Bridge** aims to provide a seamless, secure, and efficient solution for transferring all types of NFT tokens between Ethereum Layer 1 (L1) and StarkNet Layer 2 (L2).

This README outlines the strategy, methodology, benefits, technical overview, and potential challenges for **Starklane**. The bridge has the ambition to support not only ERC-721 but also ERC-1155 tokens and accommodate new token standards in the future. This project will be created in collaboration with the StarkNet team.

## Benefits

1. **Reduced gas fees**: Transferring tokens from L1 to L2 significantly reduces gas fees for users, making token transactions more cost-effective on L2.
2. **Faster transactions**: StarkNet's L2 solution results in faster token transactions due to increased network throughput.
3. **User-friendly interface**: The Starklane NFT Bridge provides an easy-to-use interface, simplifying the token transfer process for users.
4. **Scalability**: The bridge solution enables greater scalability for token markets over time, accommodating larger volumes of users and transactions.

## Strategy

The primary objectives of the Starklane NFT Bridge are to address the challenges faced by two specific personas:

1. **StarkNet NFT Native Users** who want to bridge their assets to Ethereum L1.
2. **Ethereum L1 NFT Native Users** who seek to bridge their assets to StarkNet L2.

## Source

Current GitHub Test repository in Cairo 0.1:

[https://github.com/TheArkProjekt/Starklane](https://github.com/TheArkProjekt/Starklane)

Current deployed version:

[Starklane](https://starklane.vercel.app/)

# Technical Overview

Below are explanations for different scenarios when bridging NFTs between L1 and L2.

## CASE 1 • Bridging L1 NFTs to L2

Moving a native L1 NFT to Layer 2 for the first time

1. User makes a transaction on the NFT contract to give their approval to the bridge contract to transfer their NFTs (via `approve()` or `setApprovalForAll()`).
2. User uses the `depositTokenFromL1ToL2()` function on the bridge contract which transfers their NFTs to an Escrow contract, and then sends a message to the L2 Bridge. The message includes the token URI that links the NFT metadata, the contract type and all the details needed to create the nft on L2
3. The L2 bridge contract handles the message with the function selector `handle_deposit_from_l1` and checks if there is a previously deployed contract using the mapping structure named `_l1_to_l2_addresses`.
4. If the L2 token contract does not exist, the Universal Deployer Contract is invoked to automatically deploy a default token contract with `deploy_contract()` and register the corresponding addresses in the `_l1_to_l2_addresses` mapping structure.
The deployed contract implements the `_permissioned_l2_mint` function and gives the L2 bridge contract the authority to mint a new token.
5. If the the token doesn’t exist on the L2 token contract, the L2 bridge contract mint a new token corresponding to the L1 token ID using the `_permissioned_l2_mint` function on the existing or newly deployed L2 token contract)
6. Alternatively if the token already exists and is owned by the L2 escrow contract the bridge will transfert the token to the user using the escrow contract.
7. If there is an L2 error, such as a messaging or contract issue, the operation can be cancelled and reverted. To ensure that the user's token is not stuck in the L1 contract, the user can initiate a cancellation procedure by calling a method on the L1 contract. After the waiting period of 5 days to avoid race conditions, the user can invoke a function on the L1 contract to complete the message cancellation and transfer the deposited NFTs back to their wallet.

### Main Functions

- `depositTokenFromL1ToL2(l1TokenAddress, l1OwnerAddress, l2OwnerAddress, tokenId)`: 
Used to deposit an NFT on the bridge from L1. The function includes four parameters:
    - `address: l1TokenAddress`: The address of the NFT token contract on L1.
    - `address: l1OwnerAddress`: The address of the NFT owner on L1.
    - `address: l2OwnerAddress`: The address of the NFT owner on L2.
    - `uint256: tokenId`: The unique identifier of the NFT to be deposited.
- `handle_deposit_from_l1`: 
used when receiving a message from L1
    - payload:
        - `l1_user_address`
        - `l2_user_address`
        - `l1_contract_address`
        - `l2_function_selector`
        - `token_uri`
        - `contract_type`

### Flow:
![BRIDGE_L1_FROM_L2.jpg]()

---

## CASE 2 • Returning L1 NFTs from L2

Returning a native L1 NFT, currently on Layer 2, back to its origin on Layer 1

1. User makes a transaction on the L2 NFT contract to give their approval to the bridge contract to transfer their NFTs (via `approve()` or `set_approval_for_all()`).
2. User uses the `return_l1_token_from_l2()` function on the L2 bridge contract,
if an existing association exist in the `_l1_to_l2_addresses` mapping structure then the function transfers their L2 NFT to the L2 escrow contract and then sends a message to L1 with the payload(l2UserAddress, l1UserAddress, l2ContractAddress, l1ContractAddress, tokenID, claimTime)
3. Later, the recipient User on L1 can access and consume the message using a `retrieveL1TokenFromL2()` function that will transfer the token on L1 user wallet from the escrow contract.

### Main Functions

- `return_l1_token_from_l2(l2_token_address, l2_owner_address, l1_owner_address, token_id)`: 
Used to deposit an NFT on the bridge from L1. The function includes three parameters:
    - `address: l2_token_address`: The address of the NFT token contract on L2.
    - `address: l1_owner_address`: The address of the NFT token owner on L1.
    - `address: l2_owner_address`: The address of the NFT owner on L2.
    - `uint256: token_id`: The unique identifier of the NFT to be deposited.
- `retrieveL1TokenFromL2` 
function that will trigger the consume message, used when receiving a message from L1
    - payload:
        - `l1_user_address`
        - `l2_user_address`
        - `l1_contract_address`
        - `l2_function_selector`
        - `token_uri`
        - `contract_type`

### Flow:

![RETURN_L1_FROM_L2.jpg]()

---

## CASE 3 • Bridging L2 NFTs to L1

Moving a native L2 NFT to Layer 1 for the first time.

1. User makes a transaction on the L2 NFT contract to give their approval to the bridge contract to transfer their NFTs (via `approve()` or `set_approval_for_all()`).
2. User uses the `deposit_token_from_l2_to_l1()` function on the bridge contract which transfers its NFT to an Escrow contract, and then sends a message to L1. The message includes the token URI that links the NFT metadata, the contract type and all the details needed to create the nft on L1, including a l1_claim_period_timestamp of 5 days
3. Later, the recipient User on L1 can access and consume the message using a `retrieveL2Token()` that will mint or transfer the token on L1.
4. If the matching contract doesn’t exist when retrieving the token the `retrieveL2Token()` will deploy the smart contract on L1. The deployed contract implements the `permissionedMint` function and gives the L1 bridge contract the authority to mint a new token.
5. If the the token doesn’t exist on the L1 token contract, the L1 bridge contract mint a new token corresponding to the L1 token

### Flow:

![BRIDGE_L2_TO_L1.jpg]()

### Main Functions

- `return_l2_token_from_l1(l1_token_address, l1_owner_address, token_id)`:
Used to deposit an NFT on the bridge from L1. The function includes three parameters:
    - `address: l1_token_address`: The address of the NFT token contract on L2.
    - `address: l1_owner_address`: The address of the NFT owner on L1.
    - `address: l2_owner_address`: The address of the NFT owner on L2.
    - `uint256: token_id`: The unique identifier of the NFT to be deposited.
- `retrieveL1TokenFromL2`:
function that will trigger the consume message, used when receiving a message from L1
    - payload:
        - `l2_user_address`
        - `l1_user_address`
        - `l1_contract_address`
        - `l2_contract_address`
        - `token_id`
        - `l2_function_selector`

---

## Case 4 Returning L2 NFTs from L1

Transferring a native L2 NFT, currently on Layer 1, back to its origin on Layer 2

1. User makes a transaction on the NFT contract to give their approval to the bridge contract to transfer their NFTs (via `approve()` or `setApprovalForAll()`).
2. User uses the `returnL2TokenFromL1()` function on the L1 bridge contract which triggers a Deposit event, transfers their NFTs
3. The bridge contract searches its registry to determine whether the L2 contract address is already known for the L1 contract (i.e., whether there has already been an L2-L1 bridge). Then the bridge contract sends a message to L2 and transfers the asset to an Escrow contract.
4. If the message parameter L2ContractAddress is present (enabling the StarkNet-side bridge to identify that the contract has already been deployed on L2, indicating a scenario where a token is being bridged for the second time).
5. The bridge contract invoked the Escrow contract to transfer the NFT back to the user.

### Main Functions

- `return_l2_token_from_l1(l1_token_address, l1_owner_address, token_id)`:
Used to deposit an NFT on the bridge from L1. The function includes three parameters:
    - `address: l1_token_address`: The address of the NFT token contract on L2.
    - `address: l1_owner_address`: The address of the NFT owner on L1.
    - `address: l2_owner_address`: The address of the NFT owner on L2.
    - `uint256: token_id`: The unique identifier of the NFT to be deposited.
- `retrieveL1TokenFromL2`:
function that will trigger the consume message, used when receiving a message from L1
    - payload:
        - `l2_user_address`
        - `l1_user_address`
        - `l1_contract_address`
        - `l2_contract_address`
        - `token_id`
        - `l2_function_selector`

### Flow

![RETURN_L2_FROM_L1.jpg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9cddba33-b2ea-4b3c-a7de-1833b0b05bae/RETURN_L2_FROM_L1.jpg)

---

## **Smart Contracts**

1. **L1 Bridge Contract**: Facilitates the transfer of tokens from L1 to L2.
2. **L2 Bridge Contract**: Facilitates the transfer of tokens from L2 to L1.
3. **Universal Deployer Contract**: Deploys corresponding token contracts on L2.

---

## **Smart Contract Interfaces**

### **Token Contracts (L2)**

#### Private Functions

- **`permissioned_mint(address tokenContract, uint256 tokenId, address owner)`**: A private function that can be called by the L2 bridge contract to mint a new token on L2. The function includes three parameters:
    - **`tokenContract`**: The address of the token contract on L2.
    - **`tokenId`**: The unique identifier of the NFT to be minted on L2.
    - **`owner`**: The address of the NFT owner on L2.
- **`permissioned_transfer(address tokenContract, uint256 tokenId, address from, address to)`**: A private function that can be called by the L2 bridge contract to transfer an NFT between two L2 addresses. The function includes four parameters:
    - **`tokenContract`**: The address of the token contract on L2.
    - **`tokenId`**: The unique identifier of the NFT to be transferred.
    - **`from`**: The address of the current NFT owner on L2.
    - **`to`**: The address of the new NFT owner on L2.

For the L1 -> L2 transfer, we can create a similar set of private functions for the token contracts on L1.

### **Token Contracts (L1)**

#### Private Functions

- **`_permissionedMint(address tokenContract, uint256 tokenId, address owner, claimTime)`**: A private function that can be called by the L1 bridge contract to mint a token on L1. The function includes four parameters:
    - **`tokenContract`**: The address of the token contract on L1.
    - **`tokenId`**: The unique identifier of the NFT to be transfered on L1.
    - **`owner`**: The address of the NFT owner on L1.
    - **`claimTime:`** The timestamp used to consume the message.
- **`_permissionedTransferFrom(address tokenContract, uint256 tokenId, address from, address to, claimTime)`**: A private function that can be called by the L1 bridge contract to transfer an NFT between two L1 addresses. The function includes five parameters:
    - **`tokenContract`**: The address of the token contract on L1.
    - **`tokenId`**: The unique identifier of the NFT to be transferred.
    - **`from`**: The address of the current NFT owner on L1.
    - **`to`**: The address of the new NFT owner on L1.
    - **`claimTime:`** The timestamp used to consume the message.

---

## NFT Smart Contract Ownership

Verifying ownership of deployed contracts on L1 and L2 is crucial to ensure the authenticity and security of the NFT bridge. Additionally, verifying ownership allows contract owners to extend the capabilities of their collection contracts by adding new features using the proxy pattern / replace class syscall. This pattern enables collection contracts to be upgraded and extended, improving the overall user experience and making them more adaptable to future needs. Several solutions can be implemented to achieve this goal, each with different verified statuses, such as deployed by the owner, claimed by the owner, or approved by the bridge governance delegate:

1. **Transaction-based verification**: By allowing & verifying the origin contract deployer address to initiate a transaction to create the corresponding contract on the L1 or the L2, the contracts become verified as "**deployed by the owner.**" This process ensures that only the original contract creator can create a replica of their contract on the other layer, establishing a trusted connection between the two contracts.
2. **Claim-based verification**: In cases where a corresponding contract has already been deployed, the original owner can claim their ownership using the address that deployed the origin contract. This method results in a "**claimed by the owner**" verified status, relying on the fact that the original contract creator still has control over the deploying address, which can be considered a strong indication of ownership.
3. **Committee-based verification**: If the original owner cannot prove their ownership using the deployer wallet, a small dedicated committee could be established to approve and transfer contract ownership, resulting in an "**approved by the bridge governance Committee**" verified status. The committee would evaluate evidence and claims provided by the owner to determine the legitimacy of their ownership. This approach adds a layer of human judgment and expertise to the process, which can help address complex or ambiguous ownership situations.

---

## Bridge Experience

User experience wise, Starklane has two objectives:

- **Develop a bridge with a great user experience**. We will create an easy-to-use interface for users to confidently transfer their assets. Trust is critical, and we will make sure our platform is reliable and secure for all users.
- **Set a new application quality standard in the StarkNet ecosystem**. Our solution aims to make apps and user experience in the StarkNet ecosystem better and more accessible to everyone. This will make it more attractive to a wider audience.

### Returning L2 NFTs from L1

More specifically we have identified one problem that need to be solved and that could be beneficial for the entire ecosystem:

The Starkware team discovered that transferring ERC-20 tokens from Starknet Layer 2 to Ethereum Layer 1 can be problematic for users. This issue primarily arises from the 12-hour wait time needed for Starknet Layer 2 to submit proof on Ethereum Layer 1. As a result, users must take an additional action after 12 hours to claim their tokens on Ethereum Layer 1, complicating the user experience.

To improve the user experience for both **Starkgate** and **Starklane**, we are exploring several ideas to address the time constraint related to proof submission:

1. The system predicts and pays the gas fees on behalf of the user, based on a 12-hour average variation. When users return to the interface to claim their tokens (around 12 hours later), they reimburse the spent gas fees to unlock their tokens. If the actual gas fees exceed the initially predicted amount, the transaction is canceled, and the user is notified.
2. Users link their Starknet L2 and Ethereum L1 wallets before initiating the transaction. They pay an estimated gas fee based on a 12-hour average variation. If the actual gas fees surpass the estimated amount, the transaction is canceled, the user is reimbursed, and notified.
3. Display the transaction status to bring visibility to the user (cf. inspired by block explorer)

---

## **Discussion Points**

1. **Governance**: Consideration of decentralized governance for the Starklane NFT Bridge, allowing the community to participate in decision-making processes especially in the contract claim part.
2. **Token economics**: The potential need for a native token to incentivize users and support the growth of the ecosystem.
3. **Third-party integrations**: Collaboration with other projects and platforms in the NFT space to enhance the bridge's functionality and **adoption**.
4. **Security audits**: Ensuring the safety of the bridge through rigorous audits and testing before deployment.

---

## **Potential issues**

1. **Token URI**: There are still some unresolved questions regarding the management of the token URI, with differing implementations between Cairo and Solidity. There is also the question of updating the token URI and how it will be reflected between Starknet and Ethereum.
2. **Upgradability**: The Starklane NFT Bridge must accommodate for future upgrades and improvements of Starknet to maintain its efficiency and security.
3. **Interoperability**: The bridge should support different token standards to cater to a wider range of tokens and use cases.
4. **Smart contract claim**: Collection owners can claim the smart contract with the wallet that deployed the contract, but issues may arise if people don't have ownership of the wallet that deployed the L1 contract.
5. **L2 > L1 messaging cancellation:** There’s no such mechanism