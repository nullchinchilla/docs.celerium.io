# Market simulation

## Basic idea

We want to be able to objectively reason about the stability of PrPoW. This requires an actual cryptocurrency market simulation, as opposed to a simplistic random-walk model. If we look at [the rather well-known section in Ethereum's whitepaper about stable-value cryptoassets](https://github.com/ethereum/wiki/wiki/Problems#10-stable-value-cryptoassets), we have a pretty good starting list of the actors we want to model:

* Short-term and long-term investors
* Speculative investors
* Irrational traders
* Traders with political agenda
* Media and adoption events

The way we model these sort of actors is by using a simplified market model, where there's a single market maker buying and selling coins at a small spread, continually adjusting the asking price to balance its currency reserves. Actors simulating different kinds of behavior then interact continually with the market maker.

## Actor behaviors

### Market maker

The market maker starts with 1 million coins and 1 million dollars, and initially trades 1 coin to 1 dollar at parity with a 1% spread. The mid-market rate is continually adjusted: every minute the MMR can go up or down 1% depending on whether the market maker has more coins or more dollars.

### HODLer

The HODLer continually buys 1 coin every minute. $sjkfjsdklfjsdf$

