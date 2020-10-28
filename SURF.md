`SURF.Finance Audit by AegisDAO`

{ [twitter](twitter.com/aegis_eth) | [discord](discord.gg/dEeu8nA) | [telegram](t.me/aegis_public) }

`[APPROVED FOR RELEASE ON 10/27/2020]`

---

This audit focuses on the deployer's ability to act maliciously and what steps are taken by the code to prevent abuse.
Most smart contracts are permissioned, which means that there are certain functionalities only available to accounts granted special permission by the code.

# Contract Summary
- Tito.sol: The SURF farming rewards contract
    - [Modified from SushiSwap](https://raw.githubusercontent.com/sushiswap/sushiswap/1e4db47fa313f84cd242e17a4972ec1e9755609a/contracts/MasterChef.sol) [[diff]](https://www.diffchecker.com/DDEH36Mr)
- Governor.sol: The SURF governance contract
    - [Modified from Compound](https://raw.githubusercontent.com/compound-finance/compound-protocol/9bcff34a5c9c76d51e51bcb0ca1139588362ef96/contracts/Governance/GovernorAlpha.sol) [[diff]](https://www.diffchecker.com/r1cLJBhT)
- Timelock.sol: The SURF timelock contract
    - [Modified from Compound](https://raw.githubusercontent.com/compound-finance/compound-protocol/63c992c2ad8ad04bf3e513dca770d469569cbfea/contracts/Timelock.sol) [[diff]](https://www.diffchecker.com/j69Dw61V)
- SURF.sol: The SURF ERC20 token contract
    - [Modified from SushiSwap](https://raw.githubusercontent.com/sushiswap/sushiswap/4c8445af0c5b77055bcf4190c79f4a9956a8b3df/contracts/SushiToken.sol) [[diff]](https://www.diffchecker.com/BT0NM8m0)
- Whirlpool.sol: The SURF staking rewards contract

# Contract Overview
```
           ┌──────[parameter changes]────────┐      
           │                │                │      
           ▼                ▼                │      
    ┌────────────┐   ┌────────────┐   ┌────────────┐
    │  Tito.sol  │   │Whirpool.sol│   │Governor.sol│
    └────────────┘   └────────────┘   └────────────┘
           │                ▲                ▲      
   ┌───────┼───────┐        ║                │      
   ▼       ▼       ▼        ║                │      
┌─────┐ ┌─────┐ ┌─────┐     ║                │      
│Beach│ │Beach│ │Beach│     ║                │      
└─────┘ └─────┘ └─────┘     ║                │      
   ║       ║       ▲        ║                │         
   ║       ║       ╠════════╝                │         
   ╚══════[staking reward]                   │      
                   ║                         │          
                   ▼                         │      
              .─────────.                    │      
             ( SURF.sol  )─[governance votes]┘      
              `─────────'
```
# Permissioned Function Summary
## `Tito.sol`
|Function Name|Description|Justification|Risks to Users|Mitigation|Threat Impact|
|---|---|---|---|---|---|
|[*setWhirpoolContract()*](https://github.com/SURF-Finance/contracts/blob/master/Tito.sol#L868)|Allows the contract governor to update the address for the `Whirpool`|Contract upgrades|[**Deception**] Malicious upgrades|Community vigilance and review of proposed contract upgrades|High
|[*addPool()*](https://github.com/SURF-Finance/contracts/blob/master/Tito.sol#L873)|Allows the contract governor to add a token `Beach` to the platform, rewarding LP stakers of that token|New `Beaches`|[**Corruption**] 'Bogus' tokens being listed, diluting legitimate rewards|Community vigilance and review of proposed `Beach` additions|Medium
|[*setApr()*](https://github.com/SURF-Finance/contracts/blob/master/Tito.sol#L883)|Allows the contract governor to update a `Beach`'s APR, which can either decrease or increase rewards generated for stakers on that `Beach`|Dynamic response to economic demands|[**Corruption**] `Beach` APR being arbitrarily modified without proper justification|Community vigilance and review of proposed `Beach` APR modifications|Medium
|[*addToWhitelist()*](https://github.com/SURF-Finance/contracts/blob/master/Tito.sol#L891)|Allows the contrat governor to add an EOA (externally owned account/contract address) to the whitelist|AegisDAO offers a pooling service, but flash loans are a threat to the platform. EOAs are barred from interacting with the platform unless whitelisted by the contract governor, allowing the AegisDAO pool to operate and protect the platform from manipulation with flash loan attacks|[**Deception**] Flash loan capable contract is whitelisted|Community vigilance and review of proposed whitelist modifications|High
|[*removeFromWhitelist()*](https://github.com/SURF-Finance/contracts/blob/master/Tito.sol#L896)|Allows the contract governor toremove an EOA from the whitelist|A flaw or unintended issue could be discovered with a previously whitelisted EOA, so it should be possible to remove it|[**Corruption**] The AegisDAO pool could be removed from the whitelist, preventing all poolers from withdrawing their rewards|Community vigilance (by AegisDAO pool participants) and review of proposed whitelist modifications|Low

## `SURF.sol`
|Function Name|Description|Justification|Risks to Users|Mitigation|Threat Impact|
|---|---|---|---|---|---|
|[*setContractAddresses()*](https://github.com/SURF-Finance/contracts/blob/master/SURF.sol#L68)|Allows the contract governor to configure the `Tito`, `Whirlpool` and `SURF-ETH LP` token addressses|The Tito & Whirpool contract could be upgraded in the future|[**Deception**] Malicious upgrades|Community vigilance and review of proposed contract upgrades|High
|[*setTransferFee()*](https://github.com/SURF-Finance/contracts/blob/master/SURF.sol#L75)|Allows the contract governor to modify the `SURF` transfer fee|Dynamic response to economic demands|[**Corruption**] `SURF` transfer fee being arbitrarily modified without proper justification. Depending on distribution this could heavily reward those with large stakes in the `Whirpool`|Community vigilance and review of proposed `SURF` transfer fee modifications|Medium
|[*addToTransferWhitelist()*](https://github.com/SURF-Finance/contracts/blob/master/SURF.sol#L81)|Allows the contract governor to add a new contract to the transfer fee whitelist|New contracts could be added that would necessitate being whitelisted|[**Corruption**] Arbitrary addresses could be whitelisted from transfer fees without justification, tilting the economic design in their favor|Community vigilance and review of proposed `SURF` transfer fee whitelist modifications|Medium
|[*removeFromTransferWhitelist()*](https://github.com/SURF-Finance/contracts/blob/master/SURF.sol#L87)|Allows the contract governor to remove a contract from the transfer fee whitelist|A flaw or unintended issue could be discovered with a previously whitelisted contract, so it should be possible to remove it|None|Not necessary|None
|[*migrateLockedLPTokens()*](https://github.com/SURF-Finance/contracts/blob/master/SURF.sol#L94)|Allows the contract governor to migrate (permanently locked) `SURF-ETH LP` tokens to another address|New features could be added that would necessitate migrating `SURF-ETH LP` tokens|[**Corruption**] The destination of the `SURF-ETH LP` tokens could be an arbitrary address. [**Deception**] The destination of the `SURF-ETH LP` tokens could be a malicious contract|Community vigilance and review of proposed `SURF-ETH LP` token transfer destination|High

## `Whirpool.sol`
|Function Name|Description|Justification|Risks to Users|Mitigation|Threat Impact|
|---|---|---|---|---|---|
|[*setUnstakingFee()*](https://github.com/SURF-Finance/contracts/blob/master/Whirlpool.sol#L270)|Allows the contract governor to update the unstaking fee for the `Whirpool`|Dynamic response to economic demands|[**Corruption**] `Whirpool` unstaking fee being arbitrarily modified without proper justification. Depending on distribution this could heavily reward those with large stakes in the `Whirpool`|Community vigilance and review of proposed `Whirpool` unstaking fee modifications|Medium
|[*recoverERC20()*](https://github.com/SURF-Finance/contracts/blob/master/Whirlpool.sol#L279)|Allows the contract governor to recover ERC20 tokens that are NOT SURF or the SURF-ETH LP tokens|Users new to cryptocurrency frequently make mistakes, this function would allow the contract governor to recover tokens sent to the `Whirpool` on their behalf|None|Not necessary|None

# Concerns
## Can the developers steal my tokens?
No. The permissioned functions do not allow arbitrary modification of user balances and the scope of their functionality is limited to critical parameters.
## Can the developers mint their own tokens?
No. The only address allowed to call the `SURF` minting function is the `Tito` contract. The `Tito` contract only mints new tokens under certain parameters, primarily if the total supply is under 10M (the maxium amount of SURF that can exist). Because of this permission hierarchy, users are protected from the developer minting their own tokens because `Tito` will not allow it. Thanks `Tito` 🤙
## Can the developers unlock liquidity?
No. The `SURF-ETH LP` tokens are initially owned by the `SURF` contract. The tokens can be transferred if deemed necessary by the community later on. The liquidity is not permanently locked in the sense of projects like CORE because the developers would like to remain flexible when adopting new AMM tech such as Uniswap v3.

# Conclusions
This audit covers just one facet of risk when interacting with smart contracts.
This audit does NOT cover security risks including but not limited to logic or compiler bugs.
As always, you should do your own research and consult people you trust to inspect the code for you if you do not understand it.
