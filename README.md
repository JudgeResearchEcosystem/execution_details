# Execution Details
This repo is meant for Judge Research's execution partners.  For right now it consists mostly of this readme and some examples, but we will drop functions in here as they become common across discussions with our various execution partners.  The goal is to make ingesting, interpreting and executing upon our signals super fast & easy.

## Basic Workflow

1.  Judge Research sends out signals as JSONs, starting with a 15 minute signal for a long & short ETH-USD and BTC-USD signal.
    - Each execution partner gets those JSONs at the same time.
    - Ideally you share with us an endpoint for us to send the JSONs, and another for us to ping trade & PnL info.  
2.  We give more lattitude about execution details than the typical alpha-generator -> executing partner relationship.  We encourage you to layer on whatever microstructure or execution logic makes the most sense for your in-house capabilities & spare cycles.
3.  The JSON is structured so we do *not* need knowledge about current open positions. 
    
## Getting Going

1.  Our primary live signal-sending server has a elastic IP address so you can whitelist it.
2.  You'll likely need to look at the **example JSON** and then have a conversation or two with us so you can be sure yo uare interpreting things correctly.  

## An Important Note on Assessing the Signal

1.  80% or more of the time we do not generate a 





## Other Resources

- Our [wiki](https://judgeresearch.notion.site/The-Judge-Research-Wiki-37d2ae0159254928b483f01fec87b576) lays out the larger picture, and walks you through how to use our SDK & API to contribute features. 
