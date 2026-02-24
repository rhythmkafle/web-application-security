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
First, open Burp Suite, then try logging in with random username and password. Send the request to  `Intruder` tab. Add the username as the attacking parameter, then load the `usernames.txt` file. After that, Start attack.
![[Pasted image 20260222105232.png]]
We can see that too many attempts leads to ban. This means that we need to somehow bypass the rate limit protection. 
![[Pasted image 20260222105544.png]]
However, if we add the header `X-Forwarded-For` and set the value, then the rate limiting is bypassed. Also, we can see that the application takes longer time to verify the correct username against incorrect ones. 
So, we do the following steps for finding out the username:
- Change X-Forwarded-For value every time to bypass rate limiting
- Put a very long password so that we can find out what the correct username is by filtering through response time.
- Put the intruder in Pitchfork mode

When we do this, we find the correct username to be: `as`. Now, time to find the password.

Setup the intruder tab just like how we did for usernames. Then start attack.

The password with `302` response code is the password.
Login and solve the lab.

## Lab 6: Broken Brute-force protection, IP block
We notice that we get blocked every time we login 3 times incorrectly. But, if we login correctly, it seems that the ban is lifted.
First, make a `user.txt` file which will contain usernames such that every 3rd name is wiener:
```
carlos
carlos
wiener
...
```

Then create a password list where every 3rd password is peter:
```
12345
password
peter
...
```
This way, we can make sure that we solve the lab without being bruteforced.
Run burp intruder in `pitchfork` mode.

After doing this, we will get the password to the `carlos` user.
Login to solve the lab.

## Lab : Username enumeration via account lock

