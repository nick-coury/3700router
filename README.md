# 3700_bridge

## Approach

1. Requirements   

We were asked to write the code to emulate a BGP Router. BGP routers are similar to network bridges in the way that they both have forwarding tables and they receive and send messages. One big difference between routers and bridges is that routers need to aggregate their forwarding table. 

### Overall Requirements 
- Accept route update messages from the BGP neighbors, and forward updates as appropriate
- Accept route revocation messages from the BGP neighbors, and forward revocations as appropriate
- Forward data packets towards their correct destination
- Return error messages in cases where a data packet cannot be delivered
- Coalesce forwarding table entries for networks that are adjacent and on the same port
- Serialize your routing table cache so that it can be checked for correctness

2. High Level Code Structure 

``` python


```

3. Implementation 

At a high level, here was our approach:



## Challenges 

Originally we took a different approach. Our first implementation 

## Program Features

The program is well documented: every choice is commented with an explanation of what is going on and the data structures/variable types are explicitly stated


## Testing 

