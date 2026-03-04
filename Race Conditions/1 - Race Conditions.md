Race conditions are a common type of vulnerability closely related to business logic flaws. They occur when websites process requests concurrently without adequate safeguards. This can lead to multiple distinct threads interacting with the same data at the same time, resulting in a "collision" that causes unintended behavior.

![[Pasted image 20260303143115.png]]
The period of time during which a collision is possible is known as the "race window". This could be a fraction of a second between two interactions with the database.
The impact of race condition is heavily dependent on the application and the specific functionality in which it occurs.

# Limit Overrun race conditions
It lets you exceed some kind of limit imposed by the business logic of the application.
For eg: consider an online store that lets you enter a promotional code during checkout to get a one-time discount on your order. To apply this discount, the application may perform the following high-level steps:
1. Check that you haven't already used the code
2. Apply the discount to the order total
3. Update the record in the database to reflect the fact that you've now used this code


If we later attempt reuse of this code, the initial checks performed at the start of the process should prevent us:
![[Pasted image 20260303160304.png]]

Now, what happens if a user who has never applied this discount code before tries to apply it twice at the same time?
![[Pasted image 20260303160341.png]]

As we can see, the application transitions through a temporary sub-state; i.e. a state that it enters and then exits again before request processing is complete. In this case, the sub-state begins when the server starts processing the first request, and ends when it updates the database to indicate that you've already used this code. This introduces a small race window during which you can repeatedly claim the discount as many times as you want.


The variations of this attack includes:
- Redeeming a gift card multiple times
- Rating a product multiple times
- Withdrawing or transferring cash in excess of your balance account
- Reusing a single captcha solution
- Bypassing an anti-bruteforce rate limit

Limit overruns are a subtype of so-called "time-of-check to time-of-use" (TOCTOW) .flaws. 

## Detecting and exploiting limit overrun race conditions with Burp Repeater
The process of detecting and exploiting limit overrun race conditions is relatively simple.
- Identify a single-use or rate-limited endpoint that has some kind of security impact or other useful process
- Issue multiple requests to this endpoint in quick succession to see if you can overrun this limit

The race window is often just milliseconds and can be even shorter. Even if you send all requests at exactly the same time, in practice there are various uncontrollable and unpredictable external factors that affect when the server processes each request and in which order.
![[Pasted image 20260303165653.png]]

Burp suite automatically adjusts techniques to suit the HTTP version supported by the server:
- For HTTP/1, it uses the classic last-bytedata synchronization
- For HTTP/2, it uses the single-packet attack technique, first demonstrated by Portswigger reserch at black hat usa 2023

The single packet attack enables to completely neutralize interference from network jitter by using a single TCP packet to complete 20-30 requests simultaneously:
![[Pasted image 20260303171007.png]]
## Detecting and exploiting limit overrun race conditions with Turbo Intruder
We can download Turbo Intruder from the BApp store.
Turbo Intruder requires some proficiency in Python, but it is suited to more complex attacks, such as the ones that require multiple retries, staggered request timing, or an extremely large number of requests.

To use the single-packet attack in Turbo Intruder:
1. Ensure that the target supports HTTP/2. The single-packet attack is incompatible with HTTP/1.
2. Set the `engine=Engine.BURP2` and `ConcurrentConnections=1`
3. When queueing your requests, group them by assigning them to a named gate using the `gate` argument for the `engine.queue()` method. 
4. To send all the requests in a given group, open the respective gate with the `engine.openGate()` method.

`def queueRequests(target, wordlists): engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=1, engine=Engine.BURP2 ) # queue 20 requests in gate '1' for i in range(20): engine.queue(target.req, gate='1') # send all requests in gate '1' in parallel engine.openGate('1')`