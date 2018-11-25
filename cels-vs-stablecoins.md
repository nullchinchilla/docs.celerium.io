# PRPoSW vs stablecoins

## Motivation

Currently popular cryptocurrencies such as Bitcoin and Ethereum tend to have really volatile prices, making them poor stores of value and units of account. The obvious problem this causes is the difficulty of offering goods and services denominated in, say, BTC, but a more significant problem is that such an unstable currency makes it impossible to have a functioning financial ecosystem --- any financial instruments denominated in BTC would be incredibly volatile, and people buying, say, BTC-denominated bonds would be more speculating on the future value of BTC than providing liquidity to somebody else.

Thus, it's clear that a decentralized currency with a stable value will fulfill a very significant need in the market. This space is usually filled with **stablecoins**: cryptocurrencies pegged to real-world assets like USD. Why does Celerium not try to make cel a stablecoin in this fashion?

## Existing approaches

Existing approaches at creating a stable-value cryptocurrency focus on _pegging_ it to a real-world asset, generally the USD. In fact, **stablecoins** are often defined simply as on-chain assets pegged to real-world ones. Let's examine some existing approaches to creating such pegged stablecoins:

### Asset-backed currency board

This includes examples like **Tether** and **TrueUSD**. We simply have a centralized entity acting as a currency board, taking deposits in fiat and issuing blockchain-traded IOUs.

The most obvious advantage is that with a trustworthy, reliable currency board, exchange rates will be extremely stable. No sort of "black swan" economic shock would ever break the peg as long as the currency board stays solvent, which is not hard.

The biggest problem, of course, is counterparty risk: if the currency board is compromised, dishonest, or shut down by governments, the currency will instantly collapse. HKD will not retain value if the HKMA disappears off the face of the planet. Furthermore, it gives currency boards like Tether's operating company a huge pile of cash --- it would be very hard to resist the temptation to invest or speculate with the backings and make the system secretly fractional-reserve, as Tether is often suspected of doing, further increasing counterparty risk.

### Collateralized debt

The most notable stablecoin in this category is **MakerDAO**. These stablecoins are backed with _on-chain_ assets: Dai, for example, is backed with ETH. An on-chain smart contract manages currency supply, rather than a centralized bank, and each stablecoin represents a debt of a loan backed by their cryptoassets. 

The central idea is that the currency is over-collateralized: each coin is issued only against a debt much more than its face value in cryptoassets. For example, a new stablecoin pegged to the USD may be issued only when a user deposits $2 worth of ETH. This hedges against risks in the collateral crashing, at the cost of making it difficult to see why anybody would want a stablecoin: the answer typically is that this coin can be used as a tool to margin-trade on the underlying cryptoasset. To maintain stability, whenever the price falls below the peg, collateral would be used to buy up stablecoins, and if the stablecoin is in danger of become undercollateralized all the collateral would be split up between the users.

There are several fairly salient problems with this approach. For example, scalability could be a concern: the competitiveness of the stablecoin against centralized margin-trading solutions is questionable, and this may cause problems related to a restricted money supply. In addition, there is little downward pressure on the trading price between the face value \(say $1\) and the collateral value \($2\), which may cause much more volatility than a "proper" currency-board peg. Finally, the stability of the coin during black-swan crashes of backing collateral is extremely questionable.

\(A straightforward translation to the world of fiat currencies would be a central bank trying to maintain a peg to the USD, but having foreign exchange reserves only in unrelated things like gold and stocks. It does not inspire nearly as much confidence as a currency board with 1:1 USD reserves\) 

### Seigniorage shares

The best example of this kind of system is **Basis**. Essentially, there are two currencies: discounted bonds, and pegged coins. An automatic central bank prints coins to redeem bondholders' bonds when the price of the coin is higher than the peg, or drops "helicopter money" when there are no outstanding bondholders \(in Basis's case, "shares" determine who get the helicopter money\). When the price of the coin is too low, bonds are auctioned in exchange for coins \(which are then destroyed\), entitling the bondholder to exactly 1 coin the next time money supply expands. It is claimed that this approach, with no explicit reserves, can be used to target any exchange rate with any commodity.

One problem with this approach is that the implict backing for the issued currency is "how much dollars we can get by auctioning an unlimited amount of bonds". This value is _not_ infinite, and is likely to be less than the amount of currency issued whenever confidence in the currency falls, causing a death spiral as the market loses confidence that the peg will ever recover and their bonds will ever be redeemed, discounting them exponentially until it is no longer possible to defend the peg by selling bonds.

Seigniorage shares are most likely _fundamentally_ flawed, and rely on indefinite, unsustainable adoption growth to stay stable. At least to me, this becomes blindingly obvious when transplanted to the case of a fiat currency trying to peg to another one. If systems like Basis worked, central banks would never need capital controls, foreign exchange reserves, etc to defend a peg. All they have to do is sell bonds, denominated in their own currency, redeemable in their own currency whenever the peg recovers, and be free to pursue whatever monetary policy they want! This flagrantly violates the "impossible trinity" and intuitively sounds hopelessly circular.

As Preston Byrne [more abrasively puts it,](https://prestonbyrne.com/2017/10/13/basecoin-bitshares-2-electric-boogaloo/)

> \[...\] the structure their “white paper” puts forward, which can be summarised as "_I create a coin, which will always revert to a market price of $1 per coin regardless of quantity demanded, backed by bonds which I issue, which are denominated in the coins I just created, and redeemed for coins I will later create"_ – is **recursive**_,_ or the **financial equivalent of a perpetual motion machine**.

## Can stablecoins work?

### Pegging to an asset without holding that asset

Stablecoins appear to involve a fundamentally centralization/stability tradeoff: currency boards are stable if you trust the central bank to do its job and not be shut down, but other solutions have uncertain pegs, with seigniorage shares having an especially suspect stabilization mechanism. 

A decentralized-issuance stablecoin can be reduced to the problem of directing a central bank to hold a 1:1 peg with USD without holding any USD --- since holding real USD is impossible on-chain --- under realistic market conditions. The difficulty of doing so in the world of fiat currency, despite many governments wishing to maintain pegs without sound reserves in the right currency, suggests that in the world of decentralized currencies it may be impractical too.

### Incentive problems with price feeds

Perhaps a more severe issue lies in the reliance of any stablecoin on a reliable, uncorruptible feed of exchange rates with an off-chain asset: Dai, for example, has a pool of oracles posting the ETH/USD exchange rate. This oracle, for obvious reasons, is very difficult to decentralize. Generally, coinholders either appoint a number of semi-trusted oracles in a dPoS-like scheme, or vote for what they think is the latest exchange rate in an incentivized Schelling-point game. The problem with both of these approaches is that _incentives_ for a secure feed feed are very difficult to design.

For the delegated-oracle approach, consider the case where a USD-linked stablecoin's holders delegate votes to oracles. An attacker can set up an oracle consistently reporting that stablecoins are trading below the peg. All coinholders would then benefit from voting for this fake oracle, since if this oracle ends up taking control of the price feed, the automatic central bank will think it needs to reduce coin supply, driving up coin prices. The coinholders would kill their own currency with no need for external coercion or bribery.

For a naive Schelling-point approach, a classic $$P+\epsilon$$ [attack](https://blog.ethereum.org/2015/01/28/p-epsilon-attack/) will work very well. In addition, if the voting is based on coin weight, a Schelling-point system is likely to degenerate into a delegated-oracle approach, since individual users are likely to simply parrot the listed price of a major exchange. In this case, the attacker can set up a price feed that would change to a fake, below-peg exchange rate _only if a certain threshold of clients connect to it_. Clients would willingly switch to the attacker's feed, and the same problem will occur as in the delegated-oracle approach.

Some sort of incentive-compatible price feed might be possible, but I won't hold my breath for it: it seems like as long as whoever decides the price feed has an interest in the coin going either up or down, no matter how minuscule, all bets are off. Perfectly balancing incentives are difficult to come up with, especially since subjective valuation of losses and gains are typically asymmetrical and erratic \(for example, a scheme where stakeholders benefit 100% with probability 1% if the coin depreciates while benefiting 1% with certainty if the coin appreciates won't work\)

## Supply elasticity: a different approach

### Inelastic supply of coins

Why exactly do most cryptocurrencies have such a volatile price? The biggest reason is probably _completely inelastic supply of new coins_. Coins in a traditional cryptocurrency are mined according to a fixed schedule, independent of market conditions such as the demand for new coins. Thus, volatility in the demand for coins, due to factors such as adoption events, exchange crashes, etc causes large swings in the price of a coin.

![Shock in the demand for a traditional cryptocurrency](.gitbook/assets/pic1.png)

The picture above illustrates this: a shift in the demand curve causes a large shift in the equilibrium price.

### Elastic supply with Celerium

In contrast to usual cryptocurrencies, cels are _minted_ using a novel system known as **proportional-reward  proof-of-sequential-work** \(PRPoSW\). The details are radically different from usual cryptocurrency issuance methods, but the gist of it is that by using a primitive known as a non-interactive proof of sequential work, minters create cels at the fixed rate of 1 cel per 1 day of single-core computation on a current CPU. The entire system has absolutely no oracles, Schelling-point systems, or other centralized/gameable entities --- and if you're wondering how you can possibly calculate how fast a "current CPU" is without oracles or voting, please read the specs, as the method is rather clever.

Thus, assuming that "days of CPU time" has a stable value, minters in Celerium pay _constant per-unit costs_ when currency is created. Thus, they will supply an unlimited number of cels for any above-cost price, while supplying nothing for below-cost prices. This makes the supply of cels very elastic, giving much smaller price changes during demand shocks, especially upward ones, though it is not _perfectly_ elastic, considering the contribution of "old" cels to the supply during a downturn in demand where new cels are not in demand. We get a picture more like this:

![Shock in the demand for cels](.gitbook/assets/pic2.png)

It's important to note that this sort of elasticity is very common in real life currencies, both commodity and fiat. Central banks obviously print a lot more money during times of increased demand for currency, but even the supply of gold is not entirely inelastic: changes in demand for gold certainly do result in more or less gold production, as gold takes resources to extract and does not fall down from the sky. 

There is significant evidence that commodities with relatively elastic original supplies \(such as paper\) have much stabler prices than commodities with inelastic supply \(such as precious metals\), even if the "second-hand" market dominates. 

![Price of paper over time](.gitbook/assets/screenshot-from-2018-09-26-16-56-23.png)

![Price of platinum over time](.gitbook/assets/screenshot-from-2018-09-26-17-00-46.png)



