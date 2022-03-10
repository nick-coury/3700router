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

- Separate functionality for each message type
    - Handshake message: 
        - When our router starts up, it needs to send a handshake messages to each of its neighbor routers.
    - Update message:
        - These messages tell your router how to forward data packets to a specific destination on the internet.
        - Aggregate entries in the forward table.
    - Withdraw message:
        - These messages tell your router that a route in its forward table is no longer valid.
        - Remove the entry from the forward table and reaggregate. 
    - Data message:
        - These messages get routed to their destination according to the forward table.
        - Data messages have no impact on the router, they are simple messages that get routed. 
    - Dump and Table messages:   
        - These messages are useful for testing and is a way to output your entire forward table. 
        - The convention is to recieve a dump message and to respond with a table message.   



## Challenges 

Originally we took a different approach. Our first implementation used a dictionary of dictionaries as our forwarding table which worked for the first few sets of tests. This dictionary of dictionaries would store the peer as the key, and hold the rest of the information in 
the value section, which was another dictionary. However, when we started recieving multiple messages that would have the same peer in the routing table, certain entries would be overridden and our forwarding table would be broken. Thus we needed a new way to store data in
our forwarding table. We came up with keeping a list of dictionaries this time with the peer as an element of the dictionary instead of as the key to the data. This would allow us to hold multiple entries that had the same peer which was something our old design prevented
us from doing. Because we changed the way our routing table worked, we then needed to re-work every part of our code that referenced the forwarding table before making any further progress. 

The other part of the program that we struggled a little bit with was the longest ip matching. It took us a while to figure out how we should go about doing this, and tried a number of other methods that failed before landing on our implimentation. Our implimentation
involves taking the network and the netmask, converting them into binary, and then "and"-ing the two together. Then take the destination ip and the same netmask to do the same thing, if they are equal, then its a matching ip. In the case there is more than one
matching ip, we choose the longest one. 

## Program Features

The program is well documented: every choice is commented with an explanation of what is going on and the data structures/variable types are explicitly stated. Additionally, our code has a couple helper methods that handle forward table mutation, binary and decimal ip conversions, and netmasking. 

Our code is also well abstracted making further development easier. In the case that any message type is changed or new types are added, our code has a framework to support it.


## Testing 

Testing first began with writing code to open the Unix domain sockets, and listen to all of them using select() or poll(). 

Once our sockets were correctly opened, we implement basic support for "update" and "data" messages. We verified that messages were sent correctly by reading through the program logs and manually tracking the message paths. 

At this stage we began implementing support for "dump" and "table" messages to make future testing easier. By adding this support we can dump the forward table after each update message to make sure our update method works. 

Next we focused on increasing the complexity of forwarding messages. We tested the added complexity via dump messages and by reviewing our logs manually. 

Testing the withdraw and no route functionality went hand in hand with testing our update method. If we could effectively remove and add entries to the forward table, then our dump messages would reflect the correct forward table. 

Testing for peering networks was tested the same way we tested the increased complexity of forward messages. In actuallity, this was not a very big change resulting in minor tweaks to the code. 

The last step of testing was testing our aggregation and deaggregation. We tested this functionality by adding strategic print statements to the function and underlying helper functions. We also, once again, used dump messages to tell if aggregation worked. 



