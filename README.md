# Execution Details
This repo is meant for Judge Research's execution partners.  Its goal is to make it as easy as is possible to take in, assess and execute our signals.    

Let's try to organize our conversations about execution around this (small) repo.  **Please read this read.me and the [example JSON](https://github.com/JudgeResearchEcosystem/execution_details/blob/main/example_json_and_exec_details.md) page before scheduling a call to answer questions about execution.**  

## Basic Workflow

1.  Every 1-3 minutes, we send a JSON to an endpoint you specify.  You receive that JSON at the same time as our other executing partners if you give us an API endpoint; 5-6 seconds later if you are using the dropbox folder.
2.  We do not need to know the current status of any orders, or other such informtaion.  We never custody any assets or execute any orders ourselves.  We have designed the JSON so the communication flow is purely from our AI -> your servers, other than: 
3.  Once a week we ask you send us your PnL & trade records.  
    
## Getting Started

1.  Let us know if you need an IP address to be whitelisted.
2.  You will need to look at the [example JSON](https://github.com/JudgeResearchEcosystem/execution_details/blob/main/example_json_and_exec_details.md) page in this repo and then have a conversation with us so you can be sure you are interpreting things correctly.
3.  On the day execution should start, we should schedule a short call for a final review of the logic you have implemented.  

## Interpreting Our Signal

At the highest level, the JSON is designed so the AI can send you signals without having to know your current price read or positions.  This means you will have to interpret the JSON a bit more than a straight-forward buy/sell/close order.   

See the [example JSON](https://github.com/JudgeResearchEcosystem/execution_details/blob/main/example_json_and_exec_details.md) page to learn more.

## Very Important Points on Assessing Signal Accuracy

- If you are at the stage of test capital, it is extremely inefficient to assess merely profit or profit-volatility ratios like Sharpes and Sortinos.  Rather, the signal accuracy is much more telling, and gets you to large sample qualities far faster.   

- This is because 80-90% of the time our signals are not strong enough to suggest taking a new position. We include, however, a measure of certainty in every signal, 'meas_cert' so in a matter of a few days you can have a reliably large sample from which to infer the quality of our signals.  

- Namely, you should calculate a rolling correlation - or any other metric, for that matter, as long as it is rolling - over a 240 minute window.  'meas_cert' is the portion of variance that is directional over the 15 minute forecast horizon:  that is, (close - open) / (high - low)
  
- It is important to emphasize that the 'targ_price' field included in every signal should *not* be interpreted as a representation of our signals.  Even when our AI has low certainty about the directional movement, it may still show a large 'targ_price.'  The targ_price field is derived primarily from an entirely different model of the asset(s)' volatility.   

- Asessing the signal accuracy will bias your interpretation of the signals' accuracy downards, because our strategy relies on combining forecasts.  However, because you get 'meas_cert' 24*60/3=480 times a day, you can assess the signal faster, in a matter of days; as opposed to the weeks it might take to get to large sample properties among the subset of signals that suggest taking a new position.

- We have included an [easy function](https://github.com/JudgeResearchEcosystem/execution_details/blob/main/wind_perc_chart) for you to call to compare 'meas_cert' to the observed directional change in the market. 


## Other Resources
- The wiki has been removed though may be rebuilt.
