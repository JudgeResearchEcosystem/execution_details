# Execution Details
This repo is meant for Judge Research's execution partners.  The goal is to make it easy to take in, assess and execute our signals.  This repo will grow over time.

## Basic Workflow

1.  Every 15 minutes, we send a JSON to an endpoint you specify.  You receive that JSON at the same time as our other executing partners.
2.  We do not need to know the current status of any orders, or other such informtaion.  We never custody any assets or execute any orders ourselves.  We have designed the JSON so the communication flow is purely from our AI -> your servers, other than: 
3.  Once every few days we ping an endpoint you give us for PnL & trade records.  
    
## Getting Started

1.  Our primary live signal-sending server has a elastic IP address so you can whitelist it.
2.  You'll likely need to look at the **example JSON** and then have a conversation or two with us so you can be sure you are interpreting things correctly.  

## An Important Note on Assessing the Signal

- Roughly 80% of the time our signals are not strong enough to suggest taking a new position. We include, however, a measure of certainty in every signal, 'meas_cert' so in a matter of a few days you can have a reliably large sample from which to infer the quality of our signals.  

- You can also use 'meas_cert' to help size your positions.
  
- It is important to emphasize that the 'targ_price' field included in every signal should *not* be interpreted as a representation of our signals.  Even when our AI has low certainty about the directional movement, it may still show a large 'targ_price.'    

## Interpreting Our Signal

See the [example signal]() page to learn more 



## Other Resources

- Our [wiki](https://judgeresearch.notion.site/The-Judge-Research-Wiki-37d2ae0159254928b483f01fec87b576) lays out the larger picture, and walks you through how to use our SDK & API to contribute features. 
