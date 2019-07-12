# Introduction to Auctions and Keepers in Multi Collateral Dai

The Multi Collateral Dai system within the MakerDAO Protocol is a smart contract platform on Ethereum that backs and stabilizes the value of our stablecoin, Dai. It does this through a dynamic system of Collateralized Debt Positions (CDPs), autonomous feedback mechanisms, and appropriately incentivized external actors.

In this document, we explain the auction mechanisms within the system, as well as special types of external actors, called keepers, that bid on the auctions.

## Auctions

When everything in the system is going well, value accrues through [stability fees](https://github.com/makerdao/community/blob/master/faqs/stability-fee.md) collected from the CDPs. Whenever the net income from stability fees reaches a certain limit, that surplus is used to buy MKR and burn it, thereby reducing the amount of MKR in circulation. This is done through a **Buy and Burn Auction**.

The system protects against debt creation by overcollaterization. Under ideal circumstances and with the right risk parameters, the debt for an individual CDP can be covered by the collateral deposited in that CDP. If the value of that collateral drops to the point where a CDP no longer has the required collateralization ratio, then the system automatically liquidates the CDP and sells off the collateral until the outstanding debt in the CDP, along with a liquidation penalty, is covered. This is done through a **Collateral Auction**.

If, for example, the collateral price drops sharply or no one wants to buy the collateral, then there may be debt in the liquidated CDP that system needs to take care of.

The first course of action is to cover this debt by income from the stability fees if there is any surplus to cover this. If there is not, then the system initiates a so-called Debt Auction, where the winning bidder pays Dai to cover the outstanding debt against receiving an amount of newly minted MKR - whereby the amount of MKR in circulation is increased.

**To summarize, we have three types of auctions:**

-   **Buy and Burn Auction:** Winning bidder pays MKR for surplus Dai from stability fees. The MKR received is burnt, thereby reducing the amount of MKR in circulation
    
-   **Collateral Auction:** Winning bidder pay Dai for Collateral from liquidated CDP. The Dai received is used to cover the outstanding debt in the liquidated CDP
    
-   **Debt Auction:** Winning bidder pays Dai for MKR to cover outstanding debt that Collateral auctions haven’t been able to cover. MKR is minted by the system, thereby increasing the amount in circulation

The actors that bid on these auctions are called **Keepers**.

## Keepers

A keeper is an independent (typically automated) actor that is incentivized by profit opportunities to contribute to decentralized systems. A keeper can be a human, but, in general, the name refers to programs/bots that automatically monitor and interact with CDPs and the Dai Credit System on behalf of a real-world user.

In the context of Multi Collateral Dai, keepers participate in the Debt Auctions and Collateral Auctions when CDPs are liquidated. Keepers can also perform other functions, including trading Dai to **profit from the expected long-term convergence towards the Target Price**.
  
We will now go into more detail about how the auctions work.

## The Auction Parameters and Mechanisms

Various considerations have been taken into account in the design of the auction mechanisms. For example, **from a systems point of view, it is desirable to complete the auction as soon as possible to keep the system in a steady state, so the auction mechanism shall incentivize bidders to get their bid in first on an auction**. Another consideration is that the auctions are executed on-chain, which means that the number of required transactions (and associated fees) are minimized.

  

### Auctions Glossary

**Risk parameters**. In general, the following parameters are used across all of the auction types:

-   `beg` - minimum bid increase, for example 3%
    
-   `ttl` - bid duration, for example 6 hours. The auction ends if no new bid has been placed during this time.
    
-   `tau` - auction duration, for example 24 hours. The auction ends after this period under all circumstances.
    

The values of the risk parameters are determined by MKR Governance per auction type. Note that **there are different types of Collateral auctions per type of collateral used in the system**.

**Auction and bid information**. It is always possible to see the following information for an active auction:

-   `lot` - amount that is up for auction/sale
    
-   `bid` - current highest bid
    
-   `guy` - highest bidder
    
-   `tic` - bid expiry date/time (empty if there has been zero bids)
    
-   `end` - auction expiry date/time
    

### Bid Increments During an Auction:

During auctions, bid amounts will increase by a percentage with each new bid. This is the `beg` at work. For example, the `beg` could be set to 3%, meaning if the current bidder has placed a bid of 100 Dai, then the next bid must be at least 103 Dai. Overall, the purpose of the bid increment system is to incentivize early bidding and make the auction process move quickly.

### How do bidders place bids during an auction?

As bidders place bids, the **mechanism operates like a typical transfer of tokens from a user to a smart contract would**. It allows a bidder to send a number of either DAI or MKR tokens from their address to the system/specific auction. If one bid is beat by another, the original bidder’s amount will be refunded back to that bidder’s address. It’s important to note, however, that once a bid is submitted, there is no way to cancel it. **The only possible way to get that bid amount returned is if another bidder out bids it**.

**Now, let’s review the mechanisms of the three different auction types.**

## Buy and Burn Auction (Surplus Dai Auction)

**Summary:** Buy and Burn Auction, or Surplus Auctions, are used to auction off a fixed amount of the surplus Dai in the system for MKR. This surplus Dai will normally come from Stability Fees that are accumulated in a CDP). In this auction, bidders compete with increasing amounts of MKR. Once the auction has ended, the Dai auctioned off is sent to the winning bidder. The system burns the MKR received from the winning bid.

**High-level Mechanism Process:**

-   MKR holders (Maker Governance voters) specify the amount of surplus allowed in the system. The Surplus auction is triggered when the system has a surplus of Dai above the amount decided.
    
	-  To determine whether the system has a net surplus, the income and debt in the system must be added together. Any user can do this by sending the heal transaction to the system contract called Vow.
    
	-   Provided there is a net surplus, the buy and burn auction is triggered when any user send the flop transaction to the Vow contract

-   When the auction begins, a fixed amount (lot) of Dai is put up for sale. Bidders then bid with MKR in increments greater than the minimum bid increase amount. The auction officially ends when the bid duration **ends (ttl) without another bid has been place OR when auction duration (tau) has been reached**. Once the auction ends, the MKR received for the surplus Dai is then sent to be burnt thereby contracting the systems MKR supply.
    

## Collateral Auction (Collateral Sale)

**Summary:** Collateral Auctions are used to try and cover debt in CDPs that are being liquidated because the value of the CDP collateral has fallen below a certain limit decided by the MKR Governors.

**High-level Mechanism Process**

For each type of collateral, the MKR holders specify a risk parameter called the liquidation ratio. This ratio determines the amount of overcollaterization a CDP requires to stay in safe mode. For example, if the **liquidation ratio is 150%**, then the value of the collateral must always be one and a half times the value of the Dai generated. If the value of the collateral falls below the liquidation ratio, then the CDP becomes unsafe and is liquidated by the system. The system then takes over the collateral and auctions it off to cover both the debt in the CDP and a liquidation penalty.

-   The Auction is triggered when a CDP is liquidated.
    
	-   Any user can liquidate a CDP that is unsafe by sending the `bite` transaction identifying the CDP. This will launch a collateral auction.
    
	-   If the amount of collateral in the CDP being “bitten” is less than the lot size for the auction then there will be one auction for all collateral in the CDP.
    
	-   If the amount of collateral in the CDP being “bitten” is larger than the lot size for the auction then an auction will launched with the full lot size of collateral, and the CDP can be “bitten” again to launch another auction until all collateral in the CDP is up for bidding in collateral auctions.

An important aspect of a Collateral Auction is that the **expiry and bid expiry parameters** are dependent on the specific type of collateral, where the more liquid collateral types have shorter expiry times and **vice-versa**.

Once the auction begins, the first bidder will make a bid with an amount of DAI that will cover the outstanding debt associated with the collateral amount (lot). If and when there is a bid that covers the outstanding debt, the auction will turn into a reverse auction, where bidder bids on accepting smaller parts of the collateral for the fixed amount of Dai that covers the outstanding debt. The Auction ends when the bid duration (ttl) has passed OR when the auction duration (tau) has been reached. Again, this process is to encourage bidders to bid early. Once the auction is over, the winning bidder gets the collateral sent to their address, and the Dai received for the collateral is then transferred to the system.

## Debt Auction (MKR minting)

**Summary:** Debt auctions are used to recapitalize the system by auctioning off MKR for a fixed amount of Dai. In this process, bidders compete with their willingness to accept decreasing amounts of MKR for the Dai they will pay.

**High-level Mechanism Process:**

Debt auctions are triggered when the system has a significant amount Dai debt that has passed a given debt limit

-   The MKR Governors specifies a limit for how much debt it is ok to have in the system. The Debt auction is triggered when the system has a debt in Dai below the limit that has been decided.
  
	-   In order to determine whether the system has a net debt the income and debt in the system must be added together. Any user can do this by sending the `heal` transaction to the system contract named Vow.
    
	-   Provided there is a sufficiently sized net debt, the debt auction is triggered when any user send the `flop` transaction to the Vow contract
    

This is a reverse style auction where bidders make bids on how little MKR they are willing to accept for the Dai amount they have to pay when the auction settlement occurs. To further that point, the end result of this kind of auction is a fixed amount (lot) of Dai that the winning bidder must pay. The Auction ends when the bid duration (ttl) has passed. Again keep in mind that the tau is the max duration that an auction can occur in the case of no bidders making bids. Once the auction is over, the Dai received for the MKR that the system mints, is transferred to the system balance to reduce the original debt in the system.

## In Closing

Hopefully you now have an understanding of the different types of auctions in Multi Collateral Dai, and how they work, and perhaps you also want to get ready to participate as a Keeper in these auctions.

The expectation is that almost all interactions with the auction contracts will happen via automated Keeper bots.  

To support this the Maker Foundation will release a Python Keeper API. More information about this API will follow once it is released, so stay tuned for more information in this area.

  
Meanwhile, you are always welcome to ask questions in the #keeper channel in the Maker Rocket Chat at [https://chat.makerdao.com/channel/keeper](https://chat.makerdao.com/channel/keeper)