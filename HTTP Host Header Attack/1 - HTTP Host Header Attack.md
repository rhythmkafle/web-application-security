HTTP Host header is a mandatory request header as of HTTP/1.1. It specifies the domain name that the client wants to access.
For eg: when a user visits `https://portswigger.net/web-security`, their browser will compose a request containing a Host Header as follows:
```
GET /web-security HTTP/1.1
Host: portswigger.net
```

## Purpose of Host Header
The purpose of Host Header is to help identify with back-end component the client wants to communicate with. 
If requests didn't contain Host Headers, or if the Host header was malformed in some way, this could lead to issues when routing incoming requests to the intended application.

### Virtual Hosting
Multiple websites can have a single owner, and be hosted in a single web server. Although each of the distinct website will have different domain names, they all share a common IP address with the server.

## Routing Traffic via an intermediary
Another common scenario is when websites are hosted on distinct back-end servers, but all traffic between the client and servers is routed through an intermediary system. This could be a simple load balancer or a reverse proxy of some kind. This setup is especially prevalent in cases where clients access the website via a CDN.
In this case, even though the websites are hosted on separate backends, all of their domain names resolve to a single IP address of the intermediary component. This presents some challenges because the reverse proxy or load balancer needs to know the appropriate back-end to which it should route each request.

## How does HTTP Host Header solve this problem?
In both of these scenarios, the Host header is relied on to specify the intended recipient. A common analogy is the process of sending a letter to somebody who lives in an apartment building. The entire building has the same street address, but behind this street address there are many different apartments that each need to receive the correct mail somehow. One solution to this problem is simply to include the apartment number or the recipient's name in the address. In the case of HTTP messages, the Host header serves a similar purpose.
When a browser sends the request, the target URL will resolve to the IP address of a particular server. When this server receives the request, it refers to the Host header to determine the intended back-end and forwards the request accordingly.

## What is an HTTP Host Header attack?
HTTP Host Header attacks exploit vulnerable websites that handle the value of the Host Header in an unsafe way. If the server implicitly trusts the Host Header, and fails to escape it or validate it properly, an attacker may be able to use this input to inject harmful payloads that manipulate server-side behavior.
Off-the-shelf web applications typically don't know what domain they are deployed on unless it is manually specified in a configuration file during setup. When they need to know the current domain, for example, to generate an absolute URL included in an email, they may resort to retrieving the domain name from the Host Header:
```
<a href="https://_SERVER['HOST']/support">Contact support</a>
```
The header value may also be used in a variety of interactions between different systems of the website's infrastructure.
As the Host header is in fact user controllable, this practice can lead to a number of issues. If the input is not properly escaped or validated, the Host header is a potential vector for exploiting a range of other vulnerabilities, most notably:
- Web cache poisoning
- Business logic flaws in specific functionality
- Routing-based SSRF
- Classic server-side vulnerabilities, such as SQL injection

---
# How to identify and exploit HTTP Host Header Vulnerabilities?
## Supply an arbitrary Host Header
First check what happens if you provide an arbitrary, unrecognized domain name via the Host header. Sometimes, we will still be able to access the target website even when we supply an unexpected Host header. This could be for a number of reasons.
For eg: servers are sometimes configured with a default or fallback option in case they receive requests for domain names that they don't recognize. 
On the other hand, as the Host header is such a fundamental part of how the websites work, tampering with it often means you will be unable to reach the target application at all. The front-end server or load balancer that received your request may simply now know where to forward it, resulting in an `"Invalid Host Header"` error of some kind.

## Check for flawed validation
Instead of receiving `"Invalid Host header"` response, you might find that your request is blocked as a result of some kind of security measure. For example, some websites will validate whether the Host header matches the SNI from the TLS handshake. This doesn't necessarily mean that they're immune to Host header attacks.
Sometimes, the way the website parses the Host Header can reveal loopholes that can be used to bypass the validation. For eg: some parsing algorithms will omit the port from the host header, which means that only domain name is validated. If, somehow, we can supply a non-numeric port, we can leave the domain name untouched to ensure that we reach the website, we can potentially inject  payload via the port.

```
GET /example HTTP/1.1
Host: website.com:payload
```
Alternatively, we could take advantage of a less-secure subdomain that we have already compromised.
```
GET /example HTTP/1.1
Host: hacked-subdomain.vulnerable-website.com
```
Some sites will try to apply matching logic for arbitrary subdomains. In such cases, we may be able to bypass the validation entirely by registering and arbitrary domain name that ends with the same sequence of characters as the whitelisted ones:
```
GET /example HTTP/1.1
Host:nonvulnerable-website.com
```

## Send ambiguous requests
The code that validates the host and the code that does something vulnerable often resides in different application components or even on separate servers. By identifying and exploiting discrepancies in how they retrieve the Host Header, we may be able to issue an ambiguous request that appears to have a different host depending on which system is looking at it.

Few examples:
#### Inject duplicate host headers
Try adding a duplicate Host headers. This will often just result in your request getting blocked. However, as a browser almost never sends such request, the developers might never have anticipated such scenarios.
```
GET /example HTTP/1.1 Host: vulnerable-website.com 
Host: bad-stuff-here
```
Let's say that the front-end gives precedence to the first instance of the header, but the back-end prefers the final instance. In this scenario, the first header could ensure the request is being routed to the intended destination and the second header can be used to pass payload to the server side code.

### Supply an absolute URL
The ambiguity caused by supplying both an absolute URL and a Host header can also lead to discrepancies between different systems. Officially, the request line should be given precedence when routing the request but, in practice, this isn't always the case. You can potentially exploit these discrepancies in much the same way as duplicate Host headers.
```
GET https://vulnerable-website.com/ HTTP/1.1
Host: bad-stuff-here
```
Note that we may need to experiment with different protocols. Servers will sometimes behave differently depending on whether the request line contains an HTTP or an HTTPS URL.
