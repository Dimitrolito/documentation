---
title: IP Royalty Vault
excerpt: ''
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
> ⏩ Skip the Read - 1 Minute Summary
>
> An IP Royalty Vault is a pool for all monetary inflows related to an IP Asset.
>
> Every IP Asset has 100,000,000 Royalty Tokens associated with it, where each token represents the right to 0.000001% of that IPA's revenue (*"Revenue Tokens"*) stored in the pool.
>
> Revenue Tokens are ERC-20 tokens used for payment (ex. USDC). These tokens must be whitelisted by the protocol to be used.

Each IP Asset has an IP Royalty Vault, which acts as a pool for all monetary inflows related to an IP Asset's commercial exploration or from minting licenses. Anyone who holds Royalty Tokens (defined below) has the right to claim their share of this pool.

## Token Terminology

1. Royalty Tokens - the IP Royalty Vault contract is also the ERC-20 contract for the Royalty Tokens of each IP Asset. This means the address of the IP Royalty Vault for an IP Asset is also the ERC-20 token address of the Royalty Tokens. Each IP Asset has 100,000,000 Royalty Tokens associated, where each token represents 0.000001% of those gains. The holders of these Royalty Tokens can claim the Revenue Tokens (defined below) that are in the associated IP Royalty Vault.
2. Revenue Tokens - these are the tokens used for payment (ie. ETH, USDC, etc). Royalty Tokens can be used to claim Revenue Tokens.
   1. An ERC-20 token must be whitelisted by governance in the [RoyaltyModule.sol contract](https://github.com/storyprotocol/protocol-core-v1/blob/e339f0671c9172a6699537285e32aa45d4c1b57b/contracts/modules/royalty/RoyaltyModule.sol#L50) to be used as a Revenue Token in the Royalty Module.

### Whitelisted Revenue Tokens

An ERC-20 token must be whitelisted by our protocol in the [RoyaltyModule.sol contract](https://github.com/storyprotocol/protocol-core-v1/blob/e339f0671c9172a6699537285e32aa45d4c1b57b/contracts/modules/royalty/RoyaltyModule.sol#L50) to be used as a Revenue Token. Here are the whitelisted tokens (you can mint tokens directly from the link):

| Token | Contract Address                             | Mint                                                                                                                        |
| :---- | :------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------- |
| SUSD  | `0xC0F6E387aC0B324Ec18EAcf22EE7271207dCE3d5` | <a href="https://odyssey.storyscan.xyz/address/0xC0F6E387aC0B324Ec18EAcf22EE7271207dCE3d5" target="_blank">Mint SUSD ↗️</a> |
| WIP   | `0x1516000000000000000000000000000000000000` |                                                                                                                             |

## How to obtain Royalty Tokens?

When an IP Asset receives revenue, it is deposited into its IP Royalty Vault. In order to claim revenue from this vault, you must have the associated Royalty Tokens. Once any address owns Royalty Tokens of a given IP Asset, it is entitled to that % (% of the total supply of Royalty Tokens owned) of any future Revenue Token (that is whitelisted) received & in the IP Royalty Vault.

Upon registering an IP Asset, the associated IP Account holds 100% of the Royalty Tokens. Because Royalty Tokens are ERC-20, they can be transferred like any other token. Thus, the IP Account could send them to someone else, or even put them up for sale on the secondary market.

## How Revenue Flows

This section will help explain how revenue flows from the time of payment to being claimed by the royalty token holder. For the purposes of explanation, we will use an example from the [Liquid Absolute Percentage (LAP)](doc:liquid-absolute-percentage), but it is the same for any royalty policy.

Imagine we have a scenario where IPA4 tips IPA3 1M USDC by calling `payRoyaltyOnBehalf`.

1. Revenue Tokens flow to the Royalty Module contract. This contract then splits up the tokens based on the **royalty stack** on the receiving IPA. In this case, IPA3 has a royalty stack of 15%, so 850k tokens flow to IP Royalty Vault 3, and 150k tokens flow to the LAP contract.

   ![](https://files.readme.io/be5dfdf9064320904ca27bc1f12a2475456064a19049b7d8fb2500d094746e1d-image.png)
2. The LAP contract separates the payment to the ancestors by calling `transferToVault`. In this case, IPA2 deserves 100k (10% of IPA3's earnings) and IPA1 deserves 50k (5% of IPA3's earnings).

   ![](https://files.readme.io/1ad3a4827aa302dd94bcf45ebca6749b68821fcfaadb6a85c9b70b9c8d3f4af5-image.png)
3. Now that the Revenue Tokens are in the IP Royalty Vaults, the associated Royalty Token holders can claim from the vaults. Remember, the Revenue Tokens get claimed to whoever holds the Royalty Tokens. In the most common case, they are in the IP Account since that's where they originate. To claim, you would call either `claimRevenueOnBehalfByTokenBatch` or `claimRevenueOnBehalfBySnapshotBatch`.

   ![](https://files.readme.io/c3523d5de4a3129f07eeceff5ff577178c3b3161b35fa2b75ed6e8ef98191872-image.png)

### External Royalty Policies: Rare Case

Revenue Tokens can also move from a vault to another vault via the functions `claimByTokenBatchAsSelf` or `claimBySnapshotBatchAsSelf` located in the `IpRoyaltyVault.sol` contract. For route 3 to be possible the vault that is claiming revenue tokens needs to own Royalty Tokens of the vault being claimed from. Route 3 can be particularly useful when used together with external royalty policies.

### Snapshots

One thing to note is that when an inflow to a vault occurs through any of the routes mentioned above the value is considered pending. Once the `snapshot`  function is called that pending value is recognized.

What `snapshot` does is similar to taking the current pending value in a vault and dividing it proportionally to the number of Royalty Tokens each address holds at the moment `snapshot` is called. Due to gas cost reasons, `snapshot` is not called automatically whenever a payment is made, therefore please be aware the user needs to be holding the Royalty Tokens at the time `snapshot` is called in order to have the right to claim its share of the existing pending value.