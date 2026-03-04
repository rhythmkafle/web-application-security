It is also known as shell injection. It allows an attacker to execute OS commands on the server that is running an application, and typically compromise the application and its data.

## Injecting OS Commands
In this example, a shopping application lets the user view whether an item is in stock in a particular store. This information is accessed via a URL:
```
https://insecure-website.com/stockStatus?productID=381&storeID=29
```

To provide the stock information, the application must query various legacy systems. For historical reasons, the functionality is implemented by calling out to a shell command with the product and store IDs as argument.

```
stockreport.pl 381 29
```

This command outputs the stock status for the specified item, which is returned to the user. The application implements no defenses against OS command injection, so an attacker can submit the following input to execute an arbitrary command:
```
& echo aiwefwlguh &
```

If this was successful, following error msg would be displayed:
```
Error - productID was not provided
aiwefwlguh
29: command not found
```

Placing the `&` command separator after the injected command is useful as it separates our command from whatever that follows the injection point. 

## Useful Commands
|Purpose of command|Linux|Windows|
|---|---|---|
|Name of current user|`whoami`|`whoami`|
|Operating system|`uname -a`|`ver`|
|Network configuration|`ifconfig`|`ipconfig /all`|
|Network connections|`netstat -an`|`netstat -an`|
|Running processes|`ps -ef`|`tasklist`|

## Blind OS Command Injection vulnerabilities
Imagine a website that lets users submit feedback about the site. The user enters their email address and feedback message. The server-side application then generates an email to a site administrator containing the feedback.
To do this, it might call the `mail` program with the submitted details:
```
mail -s "This site is great" -aFrom:peter@normal-user.net feedback@vulnerable-website.com
```

The output from `mail` command, is not returned in the application's responses, so using the `echo`payload won't work.

### Detecting Blind OS command injection using time delays
You can use an injected command to trigger a time delay, enabling you to confirm the Command Injection vulnerability.
The `ping` command is a good way to do this, as it lets you specify the number of ICMP packets to send.
```
& ping -c 10 127.0.0.1 &
```

