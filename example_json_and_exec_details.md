# Example JSON And Exeution Details

This should make it clear how to interpret the JSON.  In our experience, an efficient workflow for people setting up for the first time is to read through these examples and then have a follow-up conversation to confirm interpretation & recollection of the below

### High Level Goals

The design of our messages is a bit idiosyncratic.  It is meant as a message that (1) can be sent without knowledge of the current state of your portfolio or previous orders, and (2) gives appropriately high-level instructions without making execution-level decisions for you.  

This necessitates a small interpretation script necessarily sitting on your servers.  The goal of this repo is to make that drafting that script very fast & easy.

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
             "cur_price" : "", #             # price AI last observed; probably ~40 seconds out of date
            "targ_price" : "", #             # expressed as %, not exact price 
            "cert_meas" : "", #              # an unscaled measure of certainty (use this to infer signal accuracy)  
              "max_perc" : "", #             # don't enter if abs(cur_price - your quote) / cur_price > max_perc
              "wait_min" : "1",              # e.g.: if you're 2 minutes out & hit target, don't close even if u hit targ  
         "max_wait_fill" : "5", #            # don"t buy if not filled after this number of minutes
            "stop_price" : "", #             # for now will just mirror target price size
         "stop_wait_min" : "", #             # similar to 'wait_min' but for stops; unlikely ever used
              "max_hold" : "150", #          # in minutes if no subsequent live_send tells you to close, or no target hit, close down position
                   'ph_1': '', #             # placeholder fields so the jsons can keep the same length if we need additional fields to add.
                   'ph_2': '',
                   'ph_3': '',
                   'ph_4': '',
                   'ph_5': '',
            }
```

## Explaining Non-Obvious Aspects of the Above

Hopefully the notes above explain most fields, but some additional points should be made:

- *'close_only'* - If False, take a new position w/ this JSON.  Most signals are set to 'True', that is, not strong enough to warrant taking a new position, but may effect any positions that are already open.  
--**Note**  Even when no previous position might be open, we still send signals out so you can use the 'meas_cert' and 'order_side' fields together to quickly get to large sample properties regarding this signal - as opposed to if you had to wait for the more rare 'close_only':False jsons to get to large sample size.  
- *'asset_flex'* - Can you use your discretion on how to gain exposure?  For instance, using perps & spot markets.  While probably the most idiosyncratic field, it will be set to True for at least the first two quarters of 2023.  We encourage you to layer on your own microstructure models to improve performance.  
- *'asset_base' + 'asset_spec'* - pasted together, these fields tell you what the main model was trained on.  They are separate so, in cases where 'asset_flex':True, you can use 'asset_base' in your JSON interpretation scripts.  For example, with 'asset_base':'ETH-USD' you might want to gain exposure via spot and perp markets   
- *'targ_price'* - The price targets are aggressively large.  Most position-exiting will be at the end of a time period, when the next json's sign flips.  
--  **Note:** *'targ_price'* should not be used to assess the signal's accuracy, other than when *close_only* = True.
-- You should adjust your target price limit orders as new jsons come in and give updated values 
- *'order_size'* - This will grow more sophisticated in the next several weeks.  For now, it is left at 1.  You may apply your own position sizing judgements, if you would like.  
- *'cert_meas'* - Not a probability, in that it is not strictly bound btwn 0 - 1.  Rather, it has a soft boundary around -1 and 1.  If min-max scaled on its own distribution, it is a good measure of certainty that can be used as one major ingredient in sizing positions.
-- Forecasts with *cert_meas* very close to zero will only be accurate ~57% of the time, whereas larger values will enjoy win odds greater than 2-1.   
- *'wait_min'* - If your target price is hit, but with only *wait_min* minutes to go, leave open, in case the next forecast is in the same direciton
















