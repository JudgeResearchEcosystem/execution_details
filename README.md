# Execution Details
This repo is meant for Judge Research's execution partners.  The goal is to make it easy to take in, assess and execute our signals.  This repo will grow over time.

## Basic Workflow

1.  Every 15 minutes, we send a JSON to an endpoint you specify.  You receive that JSON at the same time as our other executing partners.
2.  We do not need to know the current status of any orders, or other such informtaion.  We never custody any assets or execute any orders ourselves.  We have designed the JSON so the communication flow is purely from our AI -> your servers, other than: 
3.  Once every few days we ping an endpoint you give us for PnL & trade records.  
    
## Getting Started

1.  Our primary live signal-sending server has a elastic IP address so you can whitelist it.
2.  You'll likely need to look at the **example JSON** and then have a conversation or two with us so you can be sure you are interpreting things correctly.  

## Interpreting Our Signal

At the highest level, the JSON is designed so the AI can send you signals without having to know your current price read or positions.  This means you'll have to interpret the JSON a bit more than a straight-up buy/sell/close order.   

See the [example signal](https://github.com/JudgeResearchEcosystem/execution_details/blob/main/example_json_and_exec_details.md) page to learn more.

## Very Important Points on Assessing the Signal

- Roughly 80% of the time our signals are not strong enough to suggest taking a new position. We include, however, a measure of certainty in every signal, 'meas_cert' so in a matter of a few days you can have a reliably large sample from which to infer the quality of our signals.  

- You can also use 'meas_cert' to help size your positions.
  
- It is important to emphasize that the 'targ_price' field included in every signal should *not* be interpreted as a representation of our signals.  Even when our AI has low certainty about the directional movement, it may still show a large 'targ_price.'  

- The most statistically appropriate way to measure the signal's accuracy is to compare the 'meas_cert' field you get at the beginning of every 15 minute block to the value of (close - open) / (high - low) at the end of the candlestick.  

- For complex reasons, asessing the signal this way will bias your interpretation of the signals' accuracy downards.  However, because you get 'meas_cert' 96 times a day, you can assess the signal faster in a matter of days; as opposed to the weeks or more than a month in might take to get to large sample properties among the subset of signals that suggest taking a new position.

- We have included an [easy function](https://github.com/JudgeResearchEcosystem/execution_details/blob/main/wind_perc_chart) for you to call to compare 'meas_cert' to the observed directional change in the market. 


## Other Resources
- removed wiki
