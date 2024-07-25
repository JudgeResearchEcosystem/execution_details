# Example JSON Object And Exeution Details

This should make it clear how to interpret the JSON object we send to your API.  Alternatively, you may be looking at CSVs in a Dropbox folder, where each column name is a key in the JSON object, and each row is a new instance.

In our experience, an efficient workflow for people setting up for the first time is to read through these examples and then have a follow-up conversation to confirm interpretation & recollection of the below.

### High Level Goals

The design of our messages is a bit idiosyncratic.  It is designed so messages can (1) be sent without knowledge of the current state of your portfolio or previous orders, and (2) instructions are appropriately high-level while leaving execution details up to you.  

This necessitates a small script that sits on your servers and interprets the signal.  The goal of this repo is to make drafting that script very fast & easy.

## The Basics

Prices are expressed as percentages, so you may gain exposure on a set of assets based on our directional forecasts.  The signals are trained on Binance PERPs, but should be read as directional forecasts that apply across highly correlated assets.  

Look over the below JSON object.  close_only=False implies that our AI thinks the signal is strong enough that taking a new position is warranted.  So a large majority of signals will have close_only=**True**.  That means you'd only listen to this signal in terms of governing a decision to stay in or exit an already-open position.  If the cert_meas field is above .03 for a currently-open short, or below -.03 for a long, you should **exit**.  Otherwise, you should do nothing.  

The average hold time for a position is usually multi-hour, so the vast majority of signals you receive will end up being interpreted as 'do nothing.'

### Position-Exiting Logic

The price targets are mostly composed of a forecast of (high - low) over the coming 15 minute bar.  Therefore, **should** you use price targets, you should scale them by the square root of *t* for the mean hold time.  So if that is 4.5 hours, then sqrt(18) * targ_price would be an OK way to go.  However, target prices of course only use information up to time *t*; whereas the above exit rule allows future cert_meas values to speak as the market evolves.  

### Stop Losses

The stop price for now is a simple mirror image of the targ_price.  Accordingly, it is recommended you scale the *stop_price* field by a bit more than sqrt(18) for the time being.  Modeling of different stop-loss strategies do not have clear findings, other than that a small stop loss, around 1.25% will condense the win rate to just under 60% but create a distribution of trade profitability with much larger profits than losses.  However, it does so at the cost of overall profitability. Conversely, if smaller stops allow you to allow for greater leverage, than it may be worth it.  

### Use of Leverage

The close_only=False signals cluster heavily. They perform better when the clusters are large.  It is recommended that you wait 15 minutes after a new position is taken, and if the signal is still reading close_only=False in the same direction, you take another position.  

How you manage risk is up to you, but we note that it is highly capital inefficient to not use some leverage.  Roughly 70% of the time, you will *not* be in a position.

If you have faith in your stops being filled, our backtests are based on 5x leverage and multiply our stops by min(.026, (.0025 + stop_price * sqrt(18)). So the max per-trade drawdown should be 13%.  Of course to replicate this in the real world requires an assumption of certainty in your own execution tech.

## An Example With Some Notes

```
           {"close_only" : "False", #        # Does this signal govern only exiting current positions?
            "client_id" : "JR_1", #          # Planning ahead for multiple senders; can be ignored for now
            "time_block" : "", #             # beginning of discrete time period  
            "asset_flex" : "True", #         # OK to execute across highly correlated assets e.g. spot + perps if executing party wants
            "asset_base" : "BTC-USD", #      # Built so:  asset_base + asset_spec = e.g. "BTC-USDT-SPOT" 
            "asset_spec" : "T-SPOT", #       # specific instrument most used in modeling; "general" if inappropriate
              "strat_id" : "_BTC_and_ETH_1", # Built so:  client_id + strat_id = full strategy name 
             "close_all" : "True", #         # if close_all: close position in any asset that contravenes current order (e.g. short when order_side == "long")     
            "order_side" : "long", #         # long/short 
            "order_type" : "", #             # make/take; likely left blank & to our execution partners' discretion
            "order_size" : "1", #            # expressed as %
             "cur_price" : "", #             # string; price AI last observed; probably ~40 seconds out of date
            "targ_price" : "", #             # float; expressed as %, not exact price 
            "cert_meas" : "", #              # float; an unscaled measure of certainty (use this to infer signal accuracy)  
              "max_perc" : "", #             # float; don't enter if abs(cur_price - your quote) / cur_price > max_perc
              "wait_min" : "1",              # e.g.: if you're 2 minutes out & hit target, don't close even if u hit targ  
         "max_wait_fill" : "5", #            # don"t buy if not filled after this number of minutes
            "stop_price" : "", #             # for now will just mirror target price size
         "stop_wait_min" : "", #             # similar to 'wait_min' but for stops; unlikely ever used
              "max_hold" : "150", #          # in minutes if no subsequent live_send tells you to close, or no target hit, close down position
                   'ph_1': '', #             # placeholder fields so the jsons can keep the same length if we need additional fields to add.
                   'ph_2': '',               # currently ph_1 & ph_2 are occupied, but mainly of use to us, not you.  
                   'ph_3': '',
                   'ph_4': '',
                   'ph_5': '',
            }
```

## Explaining Non-Obvious Aspects of the Above

Hopefully the notes above explain most fields, but some additional points should be made:

- *'close_only'* - If False, take a new position w/ this JSON.  Most signals are set to 'True', that is, not strong enough to warrant taking a new position, but may effect any positions that are already open.  
- *'targ_price'* - The price targets are mostly composed of a forecast of (high - low) over the coming 15 minute bar.       
--  **Note:** *'targ_price'* should not be used to assess the signal's accuracy.
- *'stop_price'* - The stop price for now is a simple mirror image of the targ_price.    
-- You should adjust your target price limit orders as new jsons come in and give updated values 
- *'order_size'* - This will grow more sophisticated.  For now, it is left at 1.  You may apply your own position sizing judgements, if you would like.  Our our tests (backtests + live data reports) assume you are comfortable with a 5x leverage and stops of max size .024%.  Therefore, max drawdowns, assuming functioning stops, would be 12%.   
- *'cert_meas'* - Not a probability, in that it is not strictly bound btwn 0 - 1.  If min-max scaled on its own distribution, it is a good measure of certainty that can be used as one major ingredient in sizing positions.
-- Forecasts with *cert_meas* very close to zero will be about as directionally accurate as a coin toss, whereas larger values will enjoy win odds greater than 2-1.   
- *'wait_min'* - If your target price is hit, but with only *wait_min* minutes to go, leave open, in case the next forecast is in the same direciton.

These three fields will be the same in every field for a given asset:

- *'asset_flex'* - Can you use your discretion on how to gain exposure?  For instance, using perps & spot markets.  While probably the most idiosyncratic field, it will be set to True for at least the first two quarters of 2023.  We encourage you to layer on your own microstructure models to improve performance.  
- *'asset_base' + 'asset_spec'* - pasted together, these fields tell you what the main model was trained on.  They are separate so, in cases where 'asset_flex':True, you can use 'asset_base' in your JSON interpretation scripts.  For example, with 'asset_base':'ETH-USD' you might want to gain exposure via spot and perp markets   















