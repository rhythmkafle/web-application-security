Authentication vulnerabilities are critical vulnerabilities. They can allow attackers to gain sensitive data and permissions to functionalities. They may also expose other possible attack surfaces. 

### Types
There are three types of authentication:
- Something you `know`, like password
- Something you `have`, like a token
- Something you `are`, like biometrics

---
# Vulnerabilities in password-based login
In password-based logins, users either register for an account themselves, or they are assigned an account by an administrator. The only way to verify the actual user is by the fact that someone has access to that password. 

## Bruteforce attacks
These attacks are typically automated using wordlists of usernames and passwords. Websites who don't implement sufficient rate limiting protection is vulnerable to such attacks.

### Enumerating Usernames
Usernames are especially easy to guess, when it confirms to a recognizable pattern. For example, most emails are of the pattern `firstname.lastname@gmail.com`. Sometimes, the privileged accounts are conveniently named `admin` or `administrator`.
Sometimes, usernames can also be reflected in HTTP response.  Usernames may also be reflected in error messages.

### Bruteforcing Passwords
If we can know the pattern in which password is set in, we can create a wordlist of similar pattern and then attempt a bruteforce.

## Flawed Bruteforce Protection
Brute-force attack will compromise an account if it gets chance to try many many passwords. Rate limiting can be implemented using mainly 2 methods:
- Locking the account that the remote user is trying to access if they make too many incorrect attempts
- Blocking the remote user's IP address if they make too many incorrect attempts

However, there might be some flaws in the protection. 
In some cases, our IP gets blocked for too many incorrect attempts, but the ban gets resets once we login to our own account. So, we can just login to our accounts every few times. 


## Account locking
If we meet the suspicious criteria, usually after a number of failed attempts, our account may be locked. 
We might be able to enumerate usernames even by the response code which says account locked.
