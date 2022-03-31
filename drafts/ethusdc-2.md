# Theory vs. Practice: A Deeper Look at ETH/USDC Liquidity Dynamics

This is the third in a series of posts by [0xfbifemboy](https://twitter.com/0xfbifemboy) on the performance of concentrated liquidity.

## Introduction

In a previous post, we examined the dynamics of ETH/USDC liquidity provisioning on Uniswap V3. Focusing on the liquidity pool with a 0.3% swap fee, which is the highest TVL pool on Uniswap, we found that positions tended to go out of range very frequently. Correlations between liquidity position size and age suggested that large liquidity providers (LPs) were potentially more sophisticated active managers of liquidity, whereas passive users of concentrated liquidity allowed their old positions to drift out of range for months at a time. Additionally, we identified the presence of substantial just-in-time (JIT) liquidity in the pool, which extracted over $1m in total profit.

Many questions remained unanswered. Although the ETH/USDC 0.3% liquidity pool is the highest TVL pool on Uniswap V3 and therefore highly informative, providers of ETH/USDC liquidity do not operate within that single pool alone. They may also choose to provide liquidity at other fee tiers, on other blockchains where Uniswap is deployed, or on other exchanges entirely. As such, to gain a clearer picture of ETH/USDC liquidity dynamics through time, it is imperative that we carefully examine how LPs behave across pools rather than limiting our analysis within a single pool.

Along these lines, we attempted to perform a cross-venue analysis of ETH/USDC liquidity on the Uniswap 0.3% and 0.05% pools on Ethereum mainnet as well as the 0.05% pool on Polygon. First, because Uniswap pools have fixed fee rates, users on Ethereum mainnet have a choice of whether to supply liquidity to the 0.3% or 0.05% ETH/USDC liquidity pool (or both). By looking to more traditional order books in analogy as well as direct computation of the fee generation efficiency of the two pools, we can measure to what extent user behavior conforms to or deviates from expected rational behavior. Second, we compare liquidity dynamics on Ethereum mainnet, where transaction fees are often high, to Polygon, where transaction fees are typically a small fraction of a dollar. Although such a comparison is far from definitive, it is nevertheless somewhat informative with regard to determining what extent high transaction fees are a direct limiting factor to active management of liquidity positions.

We find that liquidity in the 0.05% fee tier is more concentrated around the current ETH price. To some extent, this is consistent with rational behavior where users with higher expectations of price volatility set larger ranges and charge higher fees (*i.e.,* add liquidity to the 0.3% pool) as compensation for impermanent loss. However, we find that the liquidity distribution in the 0.05% pool is relatively dominated by a small number of large players. Additionally, historical data suggests that the 0.05% pool has outperformed the fee accrual of the 0.3% pool. Together, these suggest that LPs, particularly smaller or less knowledgeable ones, may be underestimating the benefits of supplying liquidity to the 0.05% pool. Furthermore, liquidity positions in the Polygon deployment of Uniswap continue to tend to drift out of range, suggesting that while lower gas costs may be important for promoting active liquidity management, user expertise (or lack thereof) may be an stronger limiting factor.

## Providing liquidity across fee tiers

As previously mentioned, the Uniswap V3 liquidity for the ETH/USDC token pair is split between two different pools, one charging a 0.3% swap fee and another charging a 0.05% swap fee. As we can see, although the TVL of the 0.3% fee tier is higher, the volume of swaps executed via the 0.05% fee tier liquidity is much greater:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/uniswap_mainnet_pools_24hvolume.png)

It is crucial to first understand the reasons why liquidity is fractured across multiple fee tiers. Suppose, for example, that we knew the price of ETH would be equal to $3,000 plus or minus 10% for perpetuity. In that case, every liquidity provider would set the range of their position from $2,700 to $3,300, and much like with stablecoin/stablecoin pairs, we would see a single, very low fee tier dominate (as all else held equal, someone providing liquidity at a slightly lower fee will have all swaps routed through them first, hence favoring a 'race to the bottom' for fees charged).

However, suppose that people have different opinions on what might happen to the price of ETH in the future. Some might expect it to be relatively stable; however, some others might expect it to be highly volatile (higher standard deviation within each time period) or to have a directional bias (increasing or decreasing). In this case, liquidity providers who expect higher volatility or directional bias than usual will prefer to set the ranges of their position higher, so they can capture the associated trading fees. They will also demand a higher fee for providing liquidity, as higher expectations of volatility or directional movement lead directly to higher expectations of impermanent losses.

The case for fee tiers can also be arrived at via analogy to classic limit order books, where a traditional market maker controls the spread in order to manage inventory imbalances and to compensate for expected adverse selection (refer to classic models such as [Avellaneda-Stoikov](https://www.math.nyu.edu/~avellane/HighFrequencyTrading.pdf) for details). Because the bid-ask spread is analogous to the swap fee charged by a liquidity pool, this is exactly analogous to a liquidity provider with higher expectations of volatility or directional drift preferring both a larger position range and a higher fee tier.

To what extent does this theoretical explanation hold true in practice? One test is to examine the distribution of liquidity across prices of ETH. If the above reasoning is accurate, then we would expect to see liquidity in the 0.05% pool more concentrated around the current market price of ETH than that of the 0.3% pool. This appears to hold true in practice:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_mainnet_density.png)

In the above plot, the vertical red line represents the current ETH price. As expected, while both liquidity pools have distributions of liquidity which are roughly centered around the current price, the liquidity in the 0.05% pool is typically concentrated to a much narrower range than the liquidity in the 0.3% pool. The practical ramification of this observation is that while most swaps will be routed through the 0.05% pool, in conditions of abnormally high market volatility and movement, especially if large swaps are made or if the price substantially increases or decreases, the 0.3% fee liquidity will begin to receive an increasing share of the trading fees
. 
Beyond the concentration of liquidity at a a tighter range, the size (*i.e.,* market value) of individual liquidity positions also appears to be slightly higher in the median case for liquidity positions minted to the 0.05% pool:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_mainnet_position_values_comparison.png)

This is an interesting observation in light of the fact that the 0.05% pool has a much lower TVL than the 0.3% pool, as it potentially suggests that liquidity provisioning in the 0.05% pool is relatively more dominated by large players. That being said, however, even large players should prefer to spread out their liquidity over several smaller positions, to avoid sharp threshold effects where sizable chunks of their liquidity become completely active or inactive as the instantaneous price crosses a boundary point. The existence of these boundary effects is clearer when examining the liquidity distribution on the 0.05% pool alone:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_mainnet_density_rest.png)

There is a very clear dropoff in available liquidity when the price of ETH falls under $2,500 or rises abouve $3,000, largely attributable to the fact that a single liquidity position accounts for the a large proportion of the liquidity in the rectangular 'chunk' in the middle of the distribution. (The sharp 'spikes' within this chunk are artifacts resulting from the nonlinear transformation between liquidity tick space and ETH prices in USD.) In fact, this is the single largest open liquidity position in the 0.05% liquidity pool, ID [209743](https://etherscan.io/nft/0xc36442b4a4522e871399cd717abdd847ab11fe88/209743):

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_large_position.png)

This single liquidity position alone was minted with, and still contains, nearly $60 million in value:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_large_position_mint.png)

With such a sizable amount of capital, it is perhaps odd that the liquidity provider ([0xEcaa8f3636270Ee917C5b08D6324722c2C4951c7](https://etherscan.io/address/0xEcaa8f3636270Ee917C5b08D6324722c2C4951c7)) does not have any other open liquidity positions, as the more capital a liquidity provider has, the more cost-efficient it is for them to carefully approximate their desired distribution of liquidity over ETH prices. While it is possible in principle to have a strong preference for a sharply thresholded distribution of liquidity, a smooth and continuous curve is far more plausible, suggesting a certain lack of care or sophistication on the part of this particular liquidity provider. Conceivably, were liquidity positions more easily manageable, the resulting distribution of liquidity across ETH prices would exhibit a smoother curve due to liquidity providers incorporating more sophisticated strategies with multiple, frequently changing positions.

## Optimizing returns across fee tiers

Another important element to consider is that the process of providing liquidity is a *competitive* one. We know that trades are initially routed through the pool at the lowest fee tier, but that as trading activity and prices fluctuate, higher fee tiers may receive greater or lesser shares of the total fees generated. Ultimately, the choice of which pool to provide liquidity to is subject to a competitive process; for example, if the TVL in one pool were extremely low, it would be very profitable to add liquidity there and capture an outsized portion of the fees.

Fee generation is a function of the liquidity distribution over ticks, price movements, swap volumes, and the actual fee rate charged. We can begin to get a grasp on these parameters by first examining the swap volume routed to each of the two Uniswap V3 ETH/USDC pools over time:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_swapvol_time.png)

Setting aside an initial period where the 0.3% pool received far higher volume than the 0.05% pool, likely due to launch-related factors with little long-term bearing, the 0.05% pool receives slightly higher swap volume than the 0.3% pool. This is not too surprising; after all, by default any sufficiently small trade will be completely routed through the 0.05% pool. However, this is not sufficient to make conclusions about the relative fee performance of the two pools. On one hand, the 0.05% pool has a lower swap fee than the 0.3% pool; on the other hand, the 0.05% pool has much lower TVL than the 0.3% pool, meaning that the marginal unit of added liquidity captures a higher fraction of the swap fees generated by the pool.

To precisely analyze this, we can consider the following question: Suppose that you had deposited 1 unit of ambient (unbounded) liquidity into each of the two liquidity pools on August 1st, 2021. (We exclude the first two months of the pools' existence, where the data are aberrant and not representative of long-term trends.) What would the growth of your accrued fees look like over time? This exact calculation is shown below:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_cumfees.png)

Surprisingly, the 0.3% pool clearly outperforms, with the accrued fees 21.0% higher than in the 0.05% pool over a period of approximately 8 months! Even though the 0.3% fee tier pool is already the highest TVL pool on Uniswap, this suggests that it 'should' have even higher TVL relative to the 0.05% pool.

One potential interpretation of this finding is that liquidity providers are underestimating the volatility of ETH prices. To attempt to distinguish the underlying reasons why this difference persists, we begin by asking the following question: Within every single hour-long timepoint, what is the *difference* between the fee accrual of the 0.3% pool and the 0.05% pool per liquidity unit? This statistic is plotted below:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_fees_diff.png)

We clearly see that while the two pools are typically relatively evenly matched in terms of fee generation, with the 0.3% pool outperforming a mere 52.8% of the time, there is a long tail of timepoints where the 0.3% pool generates substantially higher fees than the 0.05% pool. The timepoints where the 0.3% pool outperforms the 0.05% pool can be shown to correlate with a wide number of past and contemporaneous metrics; for example, within each hourly timepoint, a high variance in the distribution of ETH prices across different swaps (in tick space) is positively correlated with outperformance of the 0.3% pool:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_fees_tick_stdev.png)

Although minute, these differences inexorably add up over time into a large cumulative advantage.

These findings motivate a natural question: If we could perfectly move liquidity between the two pools each hour to maximize fee accrual, what would be the outperformance of this hypothetical liquidity position?

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_cumfees_opt.png)

At the end of this 8-month time period, perfect predictive ability to distinguish between the fee outperformance of each pool would outperform the 0.3% pool alone by 14.1% and the 0.05% pool alone by 38.0%, all else held equal.

Overall, our exploration of the liquidity dynamics across the 0.3% and 0.05% fee tiers suggests a wide gap between theory and practice. Were liquidity providers able to more skillfully and actively manage their liquidity positions, they might be able to reap substantially greater profits as well as provide superior trade execution for end users.

## Ethereum mainnet vs. Polygon

In December 2021, Uniswap was [deployed](https://www.coindesk.com/markets/2021/12/22/uniswap-launches-on-polygon-driving-matic-to-all-time-highs/) on Polygon. As Polygon is an EVM-based sidechain with much lower transaction fees than Ethereum mainnet, comparison of the behavior of liquidity providers between Uniswap pools of identical assets on Ethereum mainnet and Polygon may indicate to what extent liquidity providers are constrained in their actions due to high transaction fees on Ethereum mainnet.

There are, of course, many intrinsic differences between the mainnet and Polygon datasets:

* The Polygon deployment is only 2 months old, compared with nearly a year for Uniswap V3 on mainnet;
* The two chains plausibly have different userbases with different levels of wealth, behavioral tendencies, and preferences;
* The data for Polygon is confounded not only by the recency of deployment but also by highly volatile market conditions post-launch;
* Finally, the preexistence of Uniswap V3 liquidity pools on mainnet may crowd out liquidity provisioning which might otherwise have occurred on Polygon.

As such, it is challenging to perform these comparisons in a truly rigorous manner. Nevertheless, even informal analyses provide some clarity into future directions for improvement of DEX liquidity management. (Thankfully, because the structure of Uniswap event data is exactly identical across both chains, we were able to use the exact same data retrieval and analysis scripts, meaning that the data should be comparable at a basic, technical level.)

In general, we first observe that the Uniswap TVLs are much lower on Polygon than on Ethereum mainnet:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/uniswap_polygon_pools.png)

Interestingly, on Polygon, the 0.05% fee tier pool for ETH/USDC leads in both TVL and 24-hour volume traded, as opposed to Ethereum mainnet, where the 0.05% pool has higher volume but the 0.3% pool has higher TVL. The reason for this discrepancy is not immediately clear. One plausible explanation (among many) is that there is lower overall swap volume on Polygon Uniswap than on Ethereum mainnet:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_hourly_vol.png)

The relatively lower swap volume may originate in part from Polygon users being less wealthy and in part from the fact that preexisting liquidity on mainnet is much deeper. Because sufficiently large trades will naturally route through both lower and higher fee tier liquidity pools for superior execution, small swap sizes on Polygon might disproportionately favor the 0.05% pool for returns on liquidity provisioning, leading to the dominance of the 0.05% pool in both TVL and swap volume. However, in the absence of additional information, it is not clear to what extent this is an accurate characterization of the reasoning behind liquidity provider's choices. For simplicity, we will consider only the Polygon 0.05% pool in subsequent analyses.

In terms of total number of open liquidity positions over time, the Mainnet 0.3% pool remains the most popular by far over the entire time period studied. However, in the 3 months following the Polygon deployment of Uniswap, it rapidly caught up to and surpassed the Mainnet 0.05% pool by number of open positions:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_open_pos.png)

However, the liquidity positions on Polygon are generally much smaller than on Ethereum mainnet. As such, the TVL of open liquidity positions on Polygon is still far below that of either the 0.3% or 0.05% mainnet liquidity pools:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_open_pos_value.png)

If liquidity providers' ability to readjust out-of-range positions is limited by the fixed cost of gas on Ethereum mainnet, we should expect liquidity positions on Polygon Uniswap pools to be in range at a higher frequency than liquidity positions on Ethereum mainnet. Owners of large positions will be able to readjust their positions regardless, and so the proportion of total liquidity in range by dollar value may not be all that much higher on Polygon; however, if it is true that smaller LPs are gated by mainnet transaction fees, then it would be natural to predict a higher proportion of individual liquidity positions is in range on Polygon Uniswap than on Ethereum mainnet.

This is *weakly* supported by the data:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_positions_active.png)

Looking first at the proportion of individual positions in range at any given time (without weighting for position size), the Polygon 0.05% pool's liquidity positions are in range roughly as often as positions on the 0.3% mainnet pool and noticeably more than positions on the 0.05% mainnet pool. Several caveats are worth noting. First, there is clearly very high variability in this metric in the first several weeks immediately following deployment on Polygon. Second, because the Polygon deployment has only been active for two months, positions have not yet had as much time as the mainnet Uniswap pools to drift out of range, especially given the relatively range-bound price action of ETH in the first part of 2022.

The difference between the mainnet and Polygon deployments is also quite modest after weighting liquidity positions by size:

![](https://github.com/CrocSwap/uniswap-analysis/blob/main/posts/img/ethusdc_positions_active_weightadj.png)

As with positions on Ethereum mainnet, a much higher fraction of total liquidity value on Polygon is in range as compared to the fraction of individual positions in range at any given time, suggesting that large positions are being actively managed to avoid staying out of range for prolonged periods of time. However, again, the dearth of data renders it difficult to draw strong conclusions.

Overall, the analysis of Polygon data is somewhat inconclusive. It does not strongly indicate that out-of-range positions would be readjusted were it not for gas fees on Ethereum mainnet; at the same time, especially given the difficulty of drawing direct comparisons between the Ethereum mainnet and Polygon deployments of Uniswap V3, it does not preclude that possibility. That being said, if gas fees are indeed not a strictly limiting factor to liquidity position readjustment, the Polygon data might then plausibly be interpreted to suggest that liquidity providers would benefit from the direct incorporation of tools, vaults, or strategic aids for active position management directly inside DEXes' dApps, as well as from more comprehensive support and education about liquidity provisioning in general.

Broadly speaking, it is reassuring to notice the same trends holding up across multiple chains, such as relatively comparable distributions of position sizes and swap volumes as well as the observation that the proportion of liquidity in range is generally higher than the proportion of liquidity *positions* in range. This consistency lends credibility to the overall validity of our analyses.

## Conclusion

By expanding the scope of our analysis from the ETH/USDC 0.3% liquidity on Ethereum mainnet to encompass both the 0.05% pool on mainnet as well as the 0.05% pool on Polygon, we we were able to generate a number of suggestive results. User behavior across the 0.3% and 0.05% mainnet pools, particularly the relative concentration of the liquidity on the 0.05% pool, suggests some degree of rationality in how users are allocating capital between the two fee tiers. However, we also observed that the fee generation of the 0.3% pool consistently outperformed the fee generation of the 0.05% pool over sufficiently long periods of time by a double-digit percent over 8 months. Although the fact that this difference was not much larger (*e.g.,* 2-3x) suggests the existence of market dynamics that prevent extreme divergence of TVLs, fees, and swap volumes, it also indicates the persistence of a fairly substantial market failure. In particular, it is supportive of the hypothesis that market participants are systematically underestimating the variability and directional drift of ETH prices, leading them to express a preference for more concentrated liquidity positions at lower fee tiers which ends up generating less fee growth on their position than expected.

One possible objection is that the profitability of a liquidity position is calculated from the difference of impermanent losses and accrued fees rather than fees alone. However, the comparative fee analysis was performed with respect to one unit of ambient (*i.e.,* unbounded) liquidity in each pool, so the results remain valid even when disregarding impermanent loss. Nevertheless, a powerful extension of these results would be to examine the historical profitability of liquidity positions directly.

We also studied the differences between liquidity positions on the Ethereum mainnet and Polygon deployments of Uniswap V3. Ultimately, after an initial "break-in" period, the propensity for liquidity to be out of range was relatively similar across both blockchains, suggesting that low transaction fees do not in and of themselves serve as a sufficient condition for liquidity providers to more actively manage their positions. The persistence of out-of-range liquidity on Polygon is consistent with a hypothesis that liquidity providers, particularly less well capitalized actors, would benefit substantially from novel DEX platforms that enable more compehensible and user-friendly liquidity management.

The discussion above motivates follow-up work which will form the basis of an upcoming article in this series. To more precisely analyze the phenomena we have observed, it becomes necessary to aggregate on a per-user or per-address level (much like the analysis of just-in-time liquidity in the first ETH/USDC analysis, but on a larger scale). This will allow us to examine the profitability of specific users, tendencies to reallocate capital over time or across different venues, and more. If you have found our posts interesting thus far, please stay tuned for future installments.

— 0xfbifemboy