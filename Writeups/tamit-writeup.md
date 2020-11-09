---
title: "Traders @ MIT Writeup"
permalink: "comp/tamit"
---
# Traders @ MIT 2020
I competed (Traders @ MIT 2020)[http://traders.mit.edu/] with Olivia Jorasch. We won the competition, placing 2nd in HFT, 6th in Bonds and 2nd in Information Valuation.

## HFT
Here's a brief summary of the case:
* We write a C++ program to trade a fictional stock.
* There is some market micro-structure signal that indicate price movement, which you can exploit to make money.
* Otherwise, try market making.

The HFT signal is very hidden, and even now I am not sure how to exploit it. It seems like the best strategy is just to market make effectively. Here is the pseudocode for our MM bot (since our EC2 instance is terminated):
```
For every 12 order updates do:
    1. Find bid/ask without my bot.
    2. Cancel existing orders.
    3. fair = mid - (1 cent per 100 position)
    Sizes below are all 200
    4. If bid < fair - 0.12: put 2 buy orders at (bid + 0.01) and (bid - 0.04)
    5. If ask > fair + 0.12: put 2 sell orders at (ask - 0.01) and (ask + 0.04).
```
We changed 0.12 to 0.2 when market movement in the 12 updates was > 3 cents. Not sure if it helped. All the values above (12, 1 per 100, 200, 0.12) were minimaxed, meaning that we tried adjusting every parameter until the pnl stopped going up.

The bot did not do so well against all the bots ranked slightly below it, but it somehow pulled off wins against the top 2 ranked bots. I am guessing this is because we tuned our parameters to be slightly less aggressive in a competitive market, and the bots that placed just before us are all more aggressive market maker bots. However, the less-aggressive nature seemed to have helped in the final 10, where the market is more adverse.

## Bonds
Here's a brief summary of the case:
* We are trading risk-free bonds. Bond prices are given by *yield*. A high-yield bond is cheap, and a low-yield bond is expensive.
* There is an auction selling bonds at a high yield, and they are *always* good to buy. However, there are other teams bidding in these auctions, so you want to prioritize which ones to buy.
* There are customers appearing to different teams, buying bonds at a *very* low yield. It is *always* good to sell to customers.

Olivia and I thought hard about the case beforehand, thinking about what the fair value of a bond is, how to provide to customers and what the nash equilibrium of bidding in an auction is. Then, Olivia brought up the great point that all of these calculations are out of the window when teams have 2-3 minutes to trade 6 securities and place 6-7 bids, *on discord*. She was right.

Olivia and I split up into 2 roles: I buy/sell bonds in the market while Olivia place bids into the auction. Other teams did the same, and this is roughly the winning strategy:

For market trader:
1. After learning what bonds you acquired in the auction, sell them at 0 yield.
   1. Watch the order book: sometimes you need to sell for higher yield b/c teams are competing.
2. Look at your customer orders and buy bonds at 0 yield. If your customer is in the current year, you can buy all the way down to the customer yield.
3. **When there are 30 seconds left, cancel all bids.** This gives the bid placer enough time to place all bids.

For auction trader:
1. Start placing auction orders when the round starts, prioritizing A bonds that have customer orders.
2. Look at bond fills and pivot to bonds people are forgetting. We found that the current-year bond are most often forgotten.
3. At 30 seconds left, check your capital (the market trader may have sold bonds to get more capital), and place some final bids.

## IV
Here's a brief summary of the case:
* We have 4 symbols and 4 hedge funds, each trading 1 symbol, that will sell you their position for $200 / day.
* You can pay a 1-time fee to get this information 1 day in advance, which usually results in a better predictor for returns. The fee is done in an auction.
* There are 7 rounds. In each one, you will receive your pnl, all the price data as well as all the purchased HF position data.
* Find out which hedge funds are gold and which ones are not, then trade.

The #1 strategy is to buy B data and trade A & B using B's position. The #2 strategy (what we found) is to buy B data and trade only B. The reason you want to also trade A is that A & B are ETFs with an overlapping basket of stocks. As a concrete example, if you can predict that S&P goes up, NASDAQ will probably go up too. There was a position limit for each individual symbol, so trading A as well means extra profit.

You can probably win in this case by changing 2 lines in the starter code. However, the process of discovering these strategies is not easy.

The main challenge is that the first round of price data might have meant to throw you off: B positions had bad returns that year, while D seems to have performed well. My team, and I suspect other teams as well, followed this information and tried to build a bot based on D. It landed on its face.

Here is the rough approach we had to find this strategy. I have omitted all the dead ends and moments when I want to pull my hair out:
1. Make a Jupyter notebook with code to load in the price/positions data, which I am familiar with from previous experiences.
2. Find out how much return per $ a hedge fund is getting. Getting the hedge fund's pnl is not enough because C & D trade much larger volumes. (We also tried R^2 between hedge fund position and stock return, etc. They yield similar results)
   1. The key to the previous step is to make the code modular. That is, you should be able to change 2 lines to get all the new inputs into your machine.
3. We noticed that starting round 2, trading B has had the best return.
4. We also changed the tester code so it can test our trading bot against existing price & position data. The "buy what B buys" strategy outperformed every other strategy we had.

## Conclusion
Despite the numerous technical difficulties, it was fun -- thanks for organizing!