Here's the CIP I've been working on for a leverage-based saturation and pledge benenfit. I believe this proposal could make Cardano much more decentralized and end the MPO vs SPO debate. Tell me what you think. The Google sheets is just temporary for this post. The spreadsheet file will be part of the final pull request.

## Summary
Modify the rewards calculation equation and protocol stake pool fee parameters to incentivize delegation to low leverage pools.

## Abstract
The current incentives and fee structure is unfair to small pools and does nothing to limit high leverage among the largest entities staking on Cardano. As a result, Cardano has become more centralized as multiple pool operators (MPOs) gain more stake by creating additional pools without any restrictions. Modifying the rewards calculation equation to create a dynamic saturation point based on leverage will allow for better distribution of stake across the network. A modification of the pledge benefit portion of the rewards calculation equation will incentivize delegation to low leverage pools. Finally, a change to the current protocol stake pool fee parameters will ensure a level playing field to encourage fair competition among stake pools. This proposal increases the sybil attack resistance of the Cardano blockchain, removes any advantage or incentive for entities to operate multiple stake pools, and may reduce the large influence of centralized exchanges on block production.

## Motivation
It has been evident for quite some time that the current incentives and fee structure of the Cardano blockchain are lacking. Cardano’s Nakamoto Coefficient is in steady decline and the emergence of initial stake pool offerings (ISPOs) on the network has only accelerated the path to centralization. The inadequacy of the current protocol stems not from one’s ability to operate multiple pools, but from the lack of a mechanism to limit the leverage of the entities that participate in staking. Leverage is defined as the ratio between an entity’s stake and pledge. An entity having high leverage means its total stake is much higher than its pledge. This is bad in that it reduces the amount of stake that other entities can use for block production.

The current protocol does not consider leverage in its rewards calculation, rather it focuses on the parameter k which defines the optimal number of pools on the network. Unfortunately, having k number of pools does not enforce or incentivize decentralization, as this parameter cannot control who operates the top k pools. This is evident, as we see that many of the top k pools on Cardano are in fact operated by MPOs, making the “effective” k much lower than the k specified in the protocol. These groups of stake pools operated by single entities push many pools out of the top k pools leaving them to essentially be abandoned by delegators. To further the woes of small pool operators, the current stake pool fee parameters are unfair resulting in further abandonment. Specifically, the minimum fixed fee of 340 ADA prevents small pools from offering competitive rewards to delegators. This problem is described in detail in [CIP 23].

As a response to the increasing number of abandoned stake pools, both IOG and the Cardano Foundation have begun initiatives to delegate large amounts of stake to small pools to help them achieve more delegation. Unfortunately, these efforts have largely been ineffectual to the current state of the network. Stake pools without ISPO’s and large social media personalities are simply unable to accumulate delegation under the current protocol. A stake pool operator’s sole purpose is for block production on the Cardano blockchain. There should be no prerequisites other than having a competitive pledge and the skills necessary to run a stake pool properly. With the current protocol, many stake pools with competitive pledge amounts ranging from 50,000 ADA to 250,000 ADA and highly skilled operators are unable to accumulate delegation. The fact that an investment worth more than a house is unable to produce consistent rewards via block production is a testament to the inadequacy of the current protocol.

## Specification
This proposal is a modification of the rewards calculation equation, the function poolReward, in section 10.8 Rewards Distribution Calculation of “A Formal Specification of the Cardano Ledger”.

maxPool = (R / (1 + a0)) * (o + (s * a0 * ((o - (s * ((z0 - o) / z0))) / z0)))

where: maxPool = maximum rewards for a stake pool, R = ((reserve * rho) + fees) * (1 - tau), o = min(poolstake / totalstake, z0) = z0 for fully saturated pool, s = pledge / totalstake, and z0 = 1 / k. Current protocol parameters: k = 500, rho = 0.003, a0 = 0.3, and tau = 0.2

This proposal modifies the poolReward function by removing the current saturation mechanism which uses z0 and k and replaces it with a new mechanism that uses a new parameter max_leverage. max_leverage defines the maximum amount of leverage a stake pool can have before becoming saturated. Because z0 is removed completely and this CIP aims to incentivize delegation to low leverage pools, the portion of the poolReward function that determines the pledge benefit is simplified so that it is not a function of poolstake. This makes the total pledge benefit the same regardless of the size a pool which means the larger a pool is the less the pledge benefit influences rewards.

An expression called satpoolratio is used for implementing the saturation point of a stake pool which is determined by a pool’s pledge and the value of max_leverage:

satpoolratio = (pledge * max_leverage) / totalstake

where max_leverage is any positive integer.

The rewards calculation equation proposed:

maxPool = (R / (1 + a0)) * (o +(s * a0))

where: maxPool = maximum rewards for a stake pool, R = ((reserve * rho) + fees) * (1 - tau), o = min(poolstake / totalstake, satpoolratio) = satpoolratio for a fully saturated pool, and s = pledge / totalstake. Proposed value of parameter max_leverage: 50. Parameters rho, a0, and tao remain unchanged.

This modification to the rewards calculation equation removes one protocol parameter k and adds one new protocol parameter max_leverage. The proposed value for max_leverage is 50. However, this may be too aggressive. Community consensus should be reached so that this parameter is set thoughtfully. For implementation, max_leverage can be set high at first and then decreased incrementally to give delegators time to redelegate.

In order for this proposal to work properly with pools with low pledge, the protocol stake pool fee parameters must also be changed.

```
| Name of the Parameter   | New Parameter (Y/N)  | Deleted Parameter (Y/N) | Proposed Value   | Summary Rationale for Change |
|-----------------------  |--------------------  |------------------------ |---------------   | ---------------------------- |
| minPoolCost             | N                    | Y                       | N/A              | See Rationale section.       |
|-----------------------  |--------------------  |------------------------ |---------------   | ---------------------------- |
| minPoolRate             | Y                    | N                       | .02              | See Rationale section.       |
|-----------------------  |--------------------  |------------------------ |---------------   | ---------------------------- |
```

This proposal removes the parameter minPoolCost from the protocol. A new parameter minPoolRate as described in [CIP 23] is added. The proposed value for minPoolRate in this proposal is 0.02. Any stake pool with a margin registered lower than minPoolRate will have its margin set to minPoolRate by the protocol when calculating delegator rewards. If minPoolCost cannot be removed, then it should be set to zero.

## Rationale
The current protocol allows for staking entities to achieve high leverage without any restrictions. As a result, many of the largest entities staking on Cardano operate with very high leverage leaving less opportunity for entities with smaller delegation. These high leverage “multipools” are made possible by operators splitting their pledge and operating multiple pools. The act of “pool splitting” bypasses the saturation mechanism of the current protocol. This mechanism is not influenced by an entity’s pledge or leverage. Instead, it is influenced by parameter k which is an arbitrary number representing an optimal number of pools on the network. As stated before, an optimal number of pools does not have any influence on the decentralization of the network, as it cannot control who runs these pools.

This proposal aims to reduce the amount of leverage large entities can achieve as well as remove any incentive or advantage an entity might have operating multiple pools. This effect is achieved by removing the current saturation mechanism and replacing it with an improved mechanism which is leverage-based. A new parameter called max_leverage is introduced which sets the maximum leverage a stake pool can obtain before becoming saturated. As a result, this new saturation mechanism effectively sets a maximum leverage an entity can possess before its rewards become significantly reduced due to saturation. Furthermore, “pool splitting” is addressed, as this new mechanism removes any incentive or advantage for an entity to operate more than a single pool. If an entity decides to split their pledge and operate two pools, those pools will saturate at half the size of the original pool and only offer half the rewards.

The current protocol’s pledge benefit is also flawed in that its effects on pool rewards increase with pool size. This mechanism effectively makes it slightly more profitable to delegate to larger pools. Combine this flaw with the unfair minimum fixed fee and the inconsistency of rewards offered by small stake pools and it is clear that the current protocol does not provide a level playing field for stake pools.

To address the flawed pledge benefit, the rewards calculation equation has been simplified by removing the influence of pool size and the parameter k. The proposed pledge benefit is only influenced by pledge, totalstake, a0, and R. This means that the total ADA rewarded for the pledge benefit remains the same regardless of pool size. This effectively decreases the pledge benefit’s effect as a pool’s size increases. By making the pledge benefit’s effect inversely proportional to pool size, delegators will be incentivized to delegate to pools with low leverage. Moreover, “pool splitting” is also deterred by this new pledge benefit. If an entity decides to split their pledge and operate two pools, those pools will only offer half the total pledge benefit as the original.

The protocol stake pool fee parameters must also be changed to allow this proposal to work effectively. The current protocol operates with an unfair minimum fixed fee or minPoolCost of 340 ADA. This forces small pools to operate with significantly higher effective fees which prevents them from offering competitive rewards. This problem is described in greater detail in [CIP 23]. Unfortunately, this problem is only made worse with this proposal due to the leverage-based saturation mechanism.

To address the unfair protocol stake pool fee parameters, this proposal includes the complete removal of the minPoolCost in favor of a new parameter minPoolRate which sets a minimum marginal fee. This new parameter is described in [CIP 23]. The proposed value of minPoolRate in this proposal is 0.02.

This proposal aims specifically to reduce leverage across the network to make the Cardano blockchain considerably more decentralized. Given that majority of the entities representing the top 50% of stake operate with leverage higher than 50, we can expect that this CIP to increase Cardano’s Nakamoto Coefficient significantly. With this CIP implemented, many of the largest entities staking on Cardano will become over-saturated and delegations will overflow into smaller lower leverage entities. As creating more pools is not an option for multipools, delegators will have to redelegate to lower leverage entities in order to retain optimal staking rewards. Another result of implementing this CIP may be a reduced influence of centralized exchanges on block production. Due to the need of liquidity, centralized exchanges cannot have all their stake as pledge. In the current protocol, this is hardly an issue as pledge has little effect on rewards. However, with this proposal implemented, exchanges will have to pledge a large amount of stake to retain their rewards and influence on block production. Finally, the proposed changes also make Cardano more sybil attack resistant. The proposed saturation mechanism ensures that pools must have some amount of pledge to offer any rewards to delegators. With a pledge of zero, a pool is saturated at zero and receives no rewards. Similarly, a pool with very low pledge will be saturated at a very low stake and offer poor rewards. This makes sybil attacks via strong marketing and social influence much more difficult.

The main concern with this proposal is the effect it will have on low pledge pools that are not sybil pools or bad actors. As a result of the proposed leverage-based saturation point, pools with very low pledge will not be able to gain stake or high leverage. While their rewards will be competitive assuming the minPoolCost is removed from the protocol and that they are unsaturated, they will not be as consistent as pools with larger pledge that can gain higher stake. This problem could possibly be resolved using smart contracts that allow delegators to contribute to pledge. Another solution may be found in IOG’s [Conclave protocol].

## Backwards Compatibility
Due to the removal of parameters k and minPoolCost, this proposal is not backwards compatible. However, the rewards calculation equation is very similar to the calculation of current protocol, and no large difference in the performance of the rewards calculation is expected.

## Test Cases
See this [Google sheets] for a spreadsheet that can be used to test different values and compare the resulting rewards.

```
total stake: 33.25b
a0: 0.3
R (rewards available in epoch): 30m
k (current): 500
max_leverage (proposed): 50

Effect of new saturation mechanism on rewards (saturated versus unsaturated stake pools):

| Pool Stake | Pledge | Leverage | Pledge Benefit (current) | Total Rewards (current) | APY (current) | Pledge Benefit (proposed) | Total Rewards (proposed) | APY (proposed) |
|------------|--------|----------|--------------------------|-------------------------|---------------|---------------------------|--------------------------|----------------|
| 70m        | 1m     | 70       | 208.38                   | 48791.37                | 5.2181%       | 208.21                    | 34910.35                 | 3.7068%        |
| 10m        | 1m     | 10       | 28.65                    | 6969.08                 | 5.2172%       | 208.21                    | 7148.64                  | 5.3551%        |
| 3m         | 1m     | 3        | 6.40                     | 2088.53                 | 5.2116%       | 208.21                    | 2290.34                  | 5.7291%        |
| 70m        | 100k   | 700      | 20.82                    | 48603.82                | 5.1975%       | 20.82                     | 3491.04                  | 0.3647%        |
| 10m        | 100k   | 100      | 3.10                     | 6943.53                 | 5.1976%       | 20.82                     | 3491.04                  | 2.5808%        |
| 3m         | 100k   | 30       | 0.91                     | 2083.04                 | 5.1975%       | 20.82                     | 2102.95                  | 5.2485%        |
| 70m        | 10k    | 7000     | 2.08                     | 48585.08                | 5.1954%       | 2.08                      | 349.10                   | 0.0364%        |
| 10m        | 10k    | 1000     | 0.31                     | 6940.74                 | 5.1954%       | 2.08                      | 349.10                   | 0.2552%        |
| 3m         | 10k    | 300      | 0.09                     | 2082.22                 | 5.1954%       | 2.08                      | 349.10                   | 0.8531%        |
| 60m        | 500k   | 120      | 93.85                    | 41736.42                | 5.2072%       | 104.11                    | 17455.18                 | 2.1461%        |
| 60m        | 250k   | 240      | 46.95                    | 41689.51                | 5.2012%       | 52.05                     | 8727.59                  | 1.0674%        |
| 30m        | 250k   | 120      | 23.38                    | 20844.66                | 5.2012%       | 52.05                     | 8727.59                  | 2.1461%        |

Effect of new pledge benefit on rewards (rewards up to saturation point):

| Pool Stake | Pledge | Leverage | Pledge Benefit (current) | Total Rewards (current) | APY (current) | Pledge Benefit (proposed) | Total Rewards (proposed) | APY (proposed) |
|------------|--------|----------|--------------------------|-------------------------|---------------|---------------------------|--------------------------|----------------|
| 50m        | 1m     | 50       | 155.77                   | 34857.91                | 5.2191%       | 208.21                    | 34910.35                 | 5.2272%        |
| 40m        | 1m     | 40       | 123.99                   | 27885.71                | 5.2190%       | 208.21                    | 27969.92                 | 5.2352%        |
| 30m        | 1m     | 30       | 92.21                    | 20913.50                | 5.2188%       | 208.21                    | 21029.50                 | 5.2485%        |
| 20m        | 1m     | 20       | 60.43                    | 13941.29                | 5.2184%       | 208.21                    | 14089.07                 | 5.2751%        |
| 10m        | 1m     | 10       | 28.65                    | 6969.08                 | 5.2172%       | 208.21                    | 7148.64                  | 5.3551%        |
| 1m         | 1m     | 1        | 0.05                     | 694.09                  | 5.1956%       | 208.21                    | 902.26                   | 6.8050%        |
```

The results above were calculated assuming these stake pools charge no fees. (0% margin and 0 fixed fee)

Definitions: Pool Stake - Total stake delegated to a stake pool in ADA. Pledge -  Amount of stake pledged by a stake pool in ADA. Leverage - The ratio between Pool Stake and Pledge. Pledge Benefit -  The reward a stake pool earns for their pledge in ADA in a single epoch. Total Rewards - The total reward a stake pool earns in ADA in a single epoch (includes pledge benefit). APY - Annual Percentage Yield, the expected annual return on investment for staking represented as a percentage of the original investment.

As you can see, this proposal successfully gives low leverage stake pools much higher rewards than high leverage stake pools. This is achieved by both the proposed saturation mechanism and pledge benefit. You can also see that “pool splitting” is no longer an effective way to maximize rewards.

[CIP 23]: <https://cips.cardano.org/cips/cip23/>
[Conclave protocol]: <https://iohk.io/en/research/library/papers/conclavea-collective-stake-pool-protocol/>
[Google sheets]: <https://docs.google.com/spreadsheets/d/183y_gxY2qqCIG0tSHv9rl6Ctew0X9U5Q/edit?usp=sharing&ouid=102080327491994835248&rtpof=true&sd=true>
