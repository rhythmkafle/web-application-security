## Lab 1 : Username enumeration via different responses
The candidate usernames and passwords are given for us to download. 
When we try entering username `test` and password `test`, we get `Invalid username` as response. Now we know what **NOT** to look for.

![[Pasted image 20260220150209.png]]

The burp intruder is really slow, so I used `ffuf` to do this. 
```bash
ffuf -X POST -u "https://0af700ac03953a839759f9f500a500bf.web-security-academy.net/login" -d "username=FUZZ&password=test" -w usernames.txt -fr "Invalid username"
```

Since we know that the wrong username produced "Invalid username", we filtered the response using `-fr`.

![[Pasted image 20260220150851.png]]

We have the username. Now its time for the password.
In the same manner we find out the password.
![[Pasted image 20260220151128.png]]

Login and solve the lab.


## Lab 4: Username enumeration via subtly different responses
In this lab, when giving random username and password, we get the response `Invalid username or password.` I believe simply filtering by response text won't solve this lab. So we will filter by response size instead.
We can see that almost all of them either have 1344 lines, or 1335 lines. And hence, we filtered both of them.

![[Pasted image 20260220151815.png]]

Now do the same for password.
![[Pasted image 20260220151924.png]]

Login and solve the lab.

## Lab 5: Username enumeration via response timing
