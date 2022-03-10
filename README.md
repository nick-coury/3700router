# 3700_bridge

## Approach

1. Requirements   

We were asked to write the code to emulate a BGP Router. BGP routers are similar to network bridges in the way that they both have forwarding tables and they receive and send messages. One big difference between routers and bridges is that routers need to aggregate their forwarding table. 

1.2 Overall Requirements 
- Accept route update messages from the BGP neighbors, and forward updates as appropriate
- Accept route revocation messages from the BGP neighbors, and forward revocations as appropriate
- Forward data packets towards their correct destination
- Return error messages in cases where a data packet cannot be delivered
- Coalesce forwarding table entries for networks that are adjacent and on the same port
- Serialize your routing table cache so that it can be checked for correctness

2. High Level Code Structure 

``` python
def run(self):
    while True:
        socks = select.select(self.sockets.values(), [], [], 0.1)[0]
        
        for conn in socks:
            k, addr = conn.recvfrom(65535)
            srcif = None

            for sock in self.sockets:
                if self.sockets[sock] == conn:
                    srcif = sock
                    break
                
            # a specific message for the current socket in the list of sockets 
            msg = k.decode('utf-8')
            msg_dict = json.loads(msg)

            # Choose what to do with the received messaged based on the type
            # if an update message, call update
            if(msg_dict["type"] == "update"):
                self.update(msg_dict, srcif)
            # if a data message, call data   
            elif(msg_dict["type"] == "data"):
                self.data(msg_dict, srcif)
            # if a dump message, call dump
            elif(msg_dict["type"] == "dump"):
                self.dump(msg_dict)
            # if a withdraw message, call withdraw 
            elif(msg_dict["type"] == "withdraw"):
                self.withdraw(msg_dict, srcif) 

```

3. Implementation 

At a high level, here was our approach:



## Challenges 

Originally we took a different approach. Our first implementation used a dictionary of dictionaries 

## Program Features

The program is well documented: every choice is commented with an explanation of what is going on and the data structures/variable types are explicitly stated


## Testing 

