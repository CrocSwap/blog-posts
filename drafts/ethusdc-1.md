# Impermanent Loss and JIT Liquidity in the Uniswap ETH/USDC 0.3% Pool

This is the second in a series of posts by [0xfbifemboy](https://twitter.com/0xfbifemboy) on the performance of concentrated liquidity.

## Introduction

One of the most compelling and enduring primitives of decentralized finance has been the constant function market maker, starting with the release of Uniswap V1 in late 2018 on the Ethereum mainnet. Since then, new versions of Uniswap as well as innumerable forks have captured billions of dollars' worth of liquidity. For example, Uniswap V3, which introduced the concept of 'concentrated liquidity,' currently accounts for over [$4b of liquidity](https://info.uniswap.org/#/) on Ethereum mainnet alone.

However, the profitability of being a liquidity provider (LP) on Uniswap has recently been called into question. For example, a study by Loesch *et al.,* [Impermanent Loss in Uniswap v3,](https://arxiv.org/abs/2111.09192) estimated that about half of LPs would have been better off holding assets directly rather than providing liquidity. These results were widely popularized by an [article](https://rekt.news/uniswap-v3-lp-rekt/) on Rekt, and since then there has been a substantial amount of public dialogue about the losses that LPs face due to the phenomenon of impermanent loss (IL).

At a basic level, Uniswap LPs profit a fixed trading fee associated with each liquidity pool, typically 0.01%, 0.05%, or 0.3% of the value of each swap. However, when the price of the two assets begins to diverge, LPs experience IL associated with that divergence, as LPs are essentially continuously selling the appreciating asset of the pair for more of the depreciating asset. Therefore, LPs are in profit overall if the trading fees captured outweigh the IL to which they are subject. These considerations apply to both the *xy = k* model of Uniswap V1/V2 as well as the concentrated liquidity model of Uniswap V3. In Uniswap V3, LPs may capture more trading fees whenever their liquidity is in range; however, if the price moves permanently out of the range of their liquidity, their IL is correspondingly exacerbated. Additionally, Uniswap V3 comes with the phenomenon of "just-in-time (JIT) liquidity," which further reduces the fee accrual of 'normal' liquidity providers.

Liquidity provisioning is crucial to decentralized finance; however, if providing liquidity is generally unprofitable, then eventually DeFi will experience an enormous liquidity crisis. It is therefore tremendously important to understand how we can improve upon Uniswap V2/V3 and arrive at a model of truly sustainable liquidity provisioning. To that end, we have performed analyses of Uniswap V3 pool data which are complementary to and extend the results of Loesch *et al.*

In a [previous post](https://crocswap.medium.com/a-study-on-apes-profits-vs-impermanent-losses-56667e4029e6), we studied the fee profits vs. impermanent losses of APE/ETH and APE/USDC liquidity providers in the first 24 hours after ApeCoin's launch, for which the data suggested that liquidity providers set overly narrow ranges for their concentrated liquidity. However, a number of complex and speculative dynamics may be at play for liquidity provisioning on a newly launched, highly anticipated token. What do LP dynamics look like in a more mature liquidity pool, where a wealth of prior data exists for liquidity providers to use in formulating expectations of future volatility and price movement? There is no better place to look than the Uniswap V3 pool with the highest TVL, namely, the ETH/USDC 0.3% fee tier liquidity pool.

Our results for the ETH/USDC 0.3% pool suggest that LPs, especially smaller LPs, are constrained in the degree to which they can actively manage their concentrated liquidity positions on Uniswap V3. As a result, their positions tend to drift out of range, suffering from IL and failing to capture any trading fees. Novel decentralized exchanges that allow for simpler, more gas-efficient, active liquidity management may therefore serve as a step toward a long-term model of sustainable DEX liquidity.

## Basic statistics

As previously mentioned, approximately $4.4 billion of liquidity is currently accessible through Uniswap V3 on Ethereum mainnet. However, most pools are very small, with a handful of top liquidity pools capturing the lion's share of that value:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/uniswap_mainnet_pools.png)

To begin exploring the wealth of on-chain data, we retrieved the price and liquidity history of the pool with the highest TVL (ETH/USDC 0.3%). Because the state of the pool is always known, by keeping track of when new liquidity positions are minted, updated, or burned, we can straightforwardly interrogate the data to answer a number of basic questions.

For example, at the time of this post's writing, the ETH/USDC 0.3% pool on mainnet has around 3.4k open liquidity positions. Taking a look at the size of these positions (as measured by the market value of added collateral at the time(s) liquidity was minted or added), it appears that the majority of liquidity positions are worth 3 to 5 figures:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_position_sizes_mainnetonly.png)

In fact, out of these 3.5k positions, the vast majority (close to 80%) were minted with less than 10k of liquidity. However, the bulk of the liquidity in the pool (in dollar terms) resides in the remaining 20% of positions, some of which are individually worth millions of dollars each.

## Positions going out of range

Notably, as of the time of this post's writing, only around 2k of these 3.5k open liquidity positions are actually in range—meaning that 41% of all Uniswap V3 positions on the largest liquidity pool are earning no trading fees whatsoever! More generally, we can ask: at each timepoint since the pool was created, what proportion of liquidity positions are active (*i.e.,* the price of ETH is inside the active range specified by the position's minter)? The plot below shows the time course of this position over the last eight months.

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_positions_active_mainnetonly.png)

Immediately, we notice that in the first week of the pool's existence, the majority of liquidity positions are in range; however, over time, the proportion of in-range positions seems to fall dramatically, stabilizing at slightly above 50%. This result is naively quite unexpected, as liquidity which is completely out of range is not generating any fees for its owner, and the capital locked in those positions could be far better used elsewhere.

One natural extension of this analysis is to see how the proportion of in-range positions varies with the price of ETH. In the plot below, these two time series have been overlaid on the same plot (with an arbitrary scaling factor of 0.01 applied to the ETH price for plotting):

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/eth_price_overlay.png)

Periods of sudden price movement, such as the timepoints at the beginning of September 2021, appear to be associated with sharp reductions in the proportion of active positions, as expected. In addition, the run-up in price from October to mid-November appears to be correlated with a consistent decline in the proportion of active positions, which gradually reversed as the price of ETH fell from its all-time high over the following 3 months. Taken together, these data would appear to suggest that a great deal of the variation in the proportion of active positions is coming the price drifting in and out of range of "old" liquidity positions.

Another natural question is to ask what proportion of liquidity *in dollar terms* is in-range at any given time. Because the gas cost of modifying a position is static regardless of size, it is intuitive to expect that smaller positions end up out of range more often than larger positions. Consequently, examining the proportion of active positions alone may understate the degree to which the volume of liquidity in the pool is utilized. The results of this calculation are shown below.

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_positions_active_weightadj_mainnetonly.png)

We performed this analysis by valuing liquidity at the market price of ETH at the time that liquidity was minted or added to a given position. This is not a perfect approximation, as the price of liquidity positions varies continuously over time; however, we believe it is a reasonable simplification to make.

It is readily apparent that weighting liquidity positions by the value of the liquidity contained within them results in much higher estimates for the proportion of liquidity in range, which for the ETH/USDC 0.3% pool appears to fluctuate between 60-80%. The "jumpy," discontinuous nature of the graph likely results from the instantaneous price moving across the boundaries of very large liquidity positions, as well as large LPers choosing to enter or exit the pool wholesale.

Although the situation appears to be improved, there are several important points to note. First, upwards of 20% or more of the liquidity in the pool typically remains entirely out of range, representing a poor usage of capital. Second, these results suggest that smaller liquidity providers are limited by the costs and complexity of active liquidity management, causing their positions to frequently stay out of range as opposed to larger LPers for whom the fixed cost of active management is far more worthwhile. Effectively addressing these barriers would allow for smaller players to more equitably share in the rewards of liquidity provisioning, rather than having LPing become a game dominated by large players.

To further understand why positions become or stay out of range, we can directly examine the relationship between different variables, such as a liquidity position's age or value, and the propensity for an currently open liquidity position to be in or out of range.

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_age_vs_activity.png)

The relationship between a position's age and its likelihood of being in range is admittedly fairly confounded by month-to-month variation in the price of ETH. For example, if the price increases and then decreases, then an older position minted at the original price may go out of range on the uptrend but end up back in range after the downtrend. However, if the price of ETH continues to steadily trend upward or downward over the course of multiple years, the number of out-of-range positions minted months or years ago will steadily accumulate.

However, the relationship with position size is considerably stronger:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_size_vs_activity.png)

In general, larger liquidity positions are far more likely to be in range, consistent with our prior observations that the proportion of total liquidity in range is far greater than the proportion of distinct positions in range.

## Just-in-time liquidity

Beyond positions moving out of range, another factor that reduces the profits of liquidity providers is the phenomenon of just-in-time liquidity, where liquidity is added and removed in the same block by a maximal extractable value (MEV) bot. The liquidity addition and removal are narrowly targeted to sandwich large swaps; although this actually improves execution price for the swapper, the provisioner of JIT liquidity captures a large fraction of the trading fees generated by the swap at the expense of preexisting liquidity providers. In the long run, the increasing proliferation of JIT liquidity may serve as a strong disincentive to provide liquidity to risky asset pairs.

As such, it is informative to examine the prevalence and impact on JIT liquidity. Although there is no way to precisely identify which liquidity positions originate from JIT MEV bots, one plausible heuristic is to simply examine liquidity positions which are minted and burned in the same block. One block on Ethereum mainnet corresponds to approximately 12 seconds, so the likelihood of manually generating non-JIT liquidity positions which fall under this classification is relatively low. (We also exclude positions where the minting transaction also burns a different liquidity position or vice-versa, to filter out complex liquidity management procedures from automated vaults.)

Using this criterion, we identify 957 JIT liquidity positions in the history of the ETH/USDC 0.3% pool (around 4.1% of all minted positions). The most recent JIT liquidity position captured in our dataset is:

* Mint transaction: [0xaa23d43fb314f57d664bb55aa45c7dee965e583c7dac554d3c86c0149d3eb4af](https://etherscan.io/tx/0xaa23d43fb314f57d664bb55aa45c7dee965e583c7dac554d3c86c0149d3eb4af)
* Burn transaction: [0x3429f9efec0d362fd4b20de0db35488315d3c1c77e8a33c76c0ad013ac349792](https://etherscan.io/tx/0x3429f9efec0d362fd4b20de0db35488315d3c1c77e8a33c76c0ad013ac349792)

As you can see, both of these transactions occurred in block [14408704](https://etherscan.io/txs?block=14408704). By paging through the list of transactions in the block, we can see that the liquidity mint and burn transactions surround a separate transaction, highlighted below:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/eth_jit_etherscan.png)

The transaction sandwiched in between is [0x3ded4a5b908e88f2d3f80ecb021aaafd2167cec1da8ffd1eddd7ae9ecd7bd2f4](https://etherscan.io/tx/0x3ded4a5b908e88f2d3f80ecb021aaafd2167cec1da8ffd1eddd7ae9ecd7bd2f4), which includes, as expected, a very large swap of $365k USDC for 130 ETH:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/eth_jit_sandwiched_swap.png)

We would expect this swap to generate approximately $1k of fees if completely routed through the 0.3% ETH/USDC pool. To confirm this, we examine the transaction in which the liquitiy position was burned. The difference between the tokens "removed" and the tokens "collected" represents the accrued fees, which amount to approximately $1k, as expected.

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/eth_jit_burn.png)

To further validate this classification, we examine the extent to which the following assumptions hold: First, JIT positions should be very narrowly targeted, to allow the minter to capture as much of the trading fees generated by a target swap as possible. Second, JIT positions should be very *efficient* in generating fees relative to typical liquidity positions and with respect to the amount of time they remain active. Below, we show the distributions of the corresponding metrics (on a per-liquidity position basis).

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_fee_time_tickrange.png)

As expected, all of the single-block liquidity positions are targeted at a very narrow price range. Next, we study the distribution of fees captured by liquidity providers normalized by the duration of time that the liquidity position was active:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_fee_time_jit.png)

All of the single-block liquidity positions are extremely efficient at generating fees—no regular (non-JIT) positions even begin to come close. Together, these results suggest that our putative classification of single-block liquidity positions as JIT positions is accurate. Furthermore, the distribution of fees per active block shows the extent to which JIT positions are far more effective for fee generation than 'normal' liquidity provisioning; indeed, the distribution of JIT fee efficiency falls essentially completely outside the distribution of non-JIT fee efficiency.

We know from [prior research](https://mirror.xyz/totlsota.eth/hyu-U2Q4qp0hTxnjYdW1sACynZRS1uHOBVQ4CY-uEoc) that capturing MEV, which includes supplying JIT liquidity, is a highly competitive field. However, JIT liquidity has large barriers to entry; in particular, it requires the preexistence of a large amount of available capital as well as willingness to take on inventory risk by holding risky assets, and the number of profitable opportunities is naturally limited by the high gas fees associated with minting and burning positions.

Remarkably, even though the minting transactions originate from a variety of different EOAs, they are all routed through one of only *two* separate contracts, suggesting that these are the *only two* MEV bots which have ever successfully supplied JIT liquidity to the 0.3% ETH/USDC pool! These two contracts are:

* [0xa57Bd00134B2850B2a1c55860c9e9ea100fDd6CF](https://etherscan.io/address/0xa57Bd00134B2850B2a1c55860c9e9ea100fDd6CF), representing 87% of the JIT positions and 93% of fees accrued
* [0x9799b475dEc92Bd99bbdD943013325C36157f383](https://etherscan.io/address/0x9799b475dEc92Bd99bbdD943013325C36157f383), representing 13% of JIT positions and 7% of fees accrued

Between them, these two JIT bots have earned over $1.4 million in swap fees, not accounting for gas. 0xa57 in particular is quite [well known](https://twitter.com/wilburforce_/status/1418662736553291776?lang=en) for being a dominant supplier of JIT liquidity on Uniswap V3 in general, with $50 billion in total LP volume at the end of July 2021 (a figure which has likely grown many multiples since then). Although 0x979 is [no small fry](https://www.theblockcrypto.com/post/68791/dex-protocol-bancor-suffered-security-vulnerability-migrated-455k-worth-of-user-funds) themselves, looking at the number of JIT liquidity positions minted over time via each of the two addresses, we see that 0x979 simply *gave up* after November 2021:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_jit_by_addressmonth.png)

Perhaps thankfully, the proportion of total fees accruing to JIT liquidity continues to remain quite low: $1.2 million is only a small fraction of the total fees generated by the pool over its entire history. However, over time, the presence of JIT bots will continue to serve as an ongoing disincentive to the provisioning of non-JIT liquidity, whether passively or actively managed.

## Conclusion

The introduction of decentralized exchanges operating via a constant function market maker model and, later, incorporating concentrated liquidity have brought in tremendous amounts of capital to decentralized finance. These primitives serve as building blocks driving the progressive evolution of DeFi protocols and platforms. However, it has become clear that liquidity providers may suffer substantial losses due to impermanent loss, driven by high transaction fees, the complexity of active management, lack of education, and the growing presence of JIT liquidity. In the long run, these problems must be effectively addressed in order to foster a healthy ecosystem for liquidity providers.

In this post, we analyzed historical metrics for ETH/USDC liquidity on Uniswap, focusing on the highest TVL pool on Ethereum mainnet. We demonstrated that very large amounts of capital are inefficiently sidelined, with 40-50% of positions and 15-25% of total capital out of range at any given point in time. Our results suggest that the overall complexity of liquidity provisioning is a major impediment to more efficient, active management of liquidity positions. We also document the prevalence of JIT liquidity on Ethereum mainnet, which improves trade execution quality for swappers but cannibalizes returns for LPs. Although JIT liquidity only takes a small proportion of all fees generated, it is highly time- and capital-efficient, and the long-run equilibrium of unchecked JIT liquidity may be one where passively available liquidity is much less, if at all, accessible.

What are the precise barriers that prevent liquidity providers from active position management? Do these results generalize across assets or fee tiers? In subsequent posts in this series, we will continue to explore these questions in detail. For example, on Ethereum mainnet, where transaction fees are high, the costs of adjusting small positions often outweigh the immediate benefits. However, on Polygon, where transaction fees are fractions of a dollar, that barrier is no longer present. Comparing liquidity provisioning across Ethereum mainnet and Polygon may therefore help us answer these pressing questions. Similarly, the overall distribution of ETH/USDC liquidity across different fee tiers invites a comparison via analogy to depth of liquidity in traditional order books, which may give an indication of the degree to which liquidity providers are rationally allocating their capital. If you would like to know more about these topics, please continue to follow along as we release additional research updates.

Overall, our work suggests clear directions for future improvement in AMM and DEX design. Given the substantial losses incurred by LPs as they allow capital to sit idle, it is likely that novel DEXes that allow for cheaper or simpler liquidity management will be highly attractive to LPs.

— 0xfbifemboy