---
title: "Options Trading"
date: 2019-10-11
permalink: /notes/2019/10/11/options-trading
--- 

Since I've become a little interested in options trading and would like to some methods of algotrading for options down the line, I thought it might be a good idea to create a brief overview of options trading.

People overcomplicate options trading, but the basics are simple.

An *option* gives the buyer an opportunity to buy a 100 shares of the selected stock within an expiration time for a *strike price* per stock at a *premium*.

There are 4 basic types of options: the long call, long put, covered call, and protected put.

A *long call* is an option that bets that the stock price will go up within the expiration date. It has high upside, but fixed loss potential.

For example, consider that IBM's stock is 100 right now and I buy a call for a 105 strike price at a premium of 2/stock.

- Case 1: IBM goes up to 110 and you get 5 of profit per share. Factoring in the premium of 200, you make 500-200 = 300
- Case 2: IBM goes up to 107 and you get 2 of profit per share. Factoring in the premium of 200, you break even at 200-200 = 0
- Case 3: IBM goes up, but only to 105 or below. Of course, there's no point in buying the stock now, so you lose your premium of 200.

These 3 scenarios are encapsulated in the following graphic:
![alt_text](https://www.investopedia.com/thmb/02k9_z5E7BkUwWLVwIWKJQk69Hc=/375x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/new_buy_calls-5bfd934d46e0fb0026d44d69)

*Case 1 is the unlimited portion section, Case 2 is the x-intercept, Case 3 is the curve below the x-axis*

A *long put* is an option that bets that the stock price will go down within the expiration date. It has high upside, but fixed loss potential.

For example, consider that IBM's stock is 100 right now and I buy a put for a 95 strike price at a premium of 2/stock.

- Case 1: IBM goes down to 95 and you get 5 of profit per share. Factoring in the premium of 200, you make 500-200 = 300
- Case 2: IBM goes down to 98 and you get 2 of profit per share. Factoring in the premium of 200, you break even at 200-200 = 0
- Case 3: IBM goes down, but only to 99 or above. Of course, there's no point in buying the stock now, so you lose your premium of 200.

These 3 scenarios are encapsulated in the following graphic:
![alt_text](https://www.investopedia.com/thmb/CSSmiTC2AnvPPaMshTqp-v7m0dw=/328x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/buy_puts_1-5bfd935146e0fb00264ccdb1)

*Case 1 is the unlimited portion section, Case 2 is the x-intercept, Case 3 is the curve below the x-axis*


A *covered call* involves buying a 100 shares of an option and selling a call option against those shares. By selling the call option, the seller receives the premium, capping the loss potential. However, if the stock reaches the strike price, the seller must sell the stock at the strike price, capping his or her upside.

![alt_text](https://www.investopedia.com/thmb/Y6oXuBl1qgLljXTX9kTA_L85iKc=/300x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/covered_calls-5bfd934f46e0fb00264cccb5)


A *protected put* is like an insurance policy. 
One owns a stock and wants protection in case it goes down. 
So, one purchases a put in the short term. 
For example, if I own 100 shares of IBM for 100 per share, I can buy a protected put at a strike price of 90. Hence, if the stock goes down to 80, then I have the right to sell at the strike price of 90. Hence, instead of losing 20/share, I only lose 10/share.
