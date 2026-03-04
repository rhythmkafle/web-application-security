## Lab: OS Command Injection, Simple Case
Go to the home page, and then click on `View Details`. After that, click on `Check Stock` at the bottom of the page.
Here, open the Networks tab, or burp suite, and see that a  POST request is being sent with
`productId=1&storeId=1`.
Modify the value to:
`productId=1&storeId=1|whoami`. Then send the request. The lab will be solved.

---
## Lab: Blind OS command injection with time delays
Needed to open Burp Suite for this one. Go to the `Submit Feedback` form. And then submit a feedback. We see that a POST request is being sent. In the POST request, change the value of email to `xyz||ping+-c+10+127.0.0.1||`.
Then send the request. The response will occur after 10 seconds, and the lab will be solved.

---
