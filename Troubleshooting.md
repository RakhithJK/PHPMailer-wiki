#Troubleshooting connection problems
Whatever problem you're having, first make sure you are using the [latest PHPMailer](https://github.com/PHPMailer/PHPMailer).

##"SMTP Error: Could not connect to SMTP host."
This is often reported as a PHPMailer problem, but it's almost always down to local DNS failure, firewall blocking or other issue on your local network. It means that PHPMailer is unable to contact the SMTP server you have specified in the `Host` property, but doesn't say exactly why. It can also be caused by not having the `openssl` extension loaded (See encryption notes below).

You can get verbose feedback on the connection and the whole SMTP conversation by setting `SMTPDebug = 4`.

Some techniques to diagnose the source of this error are discussed below.

##Including the wrong file
Not so long ago, PHPMailer changed the way that it loaded classes so that it was more compatible with composer, many frameworks, and the [PHP PSR-0 autoloading standard](http://www.php-fig.org/psr/psr-0/). Note that because we support PHP back to version 5.0, we cannot support the more recent [PSR-4 standard](http://www.php-fig.org/psr/psr-4/), nor can we use namespaces. Previously, PHPMailer loaded the SMTP class explicitly, and this causes problems if you want to provide your own implementation. You may have seen old scripts doing this:

```php
require 'class.phpmailer.php';
```

If you do only that, **SMTP sending will fail**. You need to either explicitly include the `class.smtp.php` file, or use the recommended approach of using the supplied autoloader, like this:

```php
require 'PHPMailerAutoload.php';
```

##DNS failures

These are often seen as connection timeouts, or "could not resolve host" or similar errors. Check your DNS is working by using the `dig` tool (from the `dnsutils` package on Debian/Ubuntu):

```shell
dig +short smtp.gmail.com
```
You will get something like this if your DNS is working:

```shell
gmail-smtp-msa.l.google.com.
173.194.67.108
173.194.67.109
```
If this fails, PHPMailer will not be able to send email because it won't be able to obtain the correct IP address to connect to. If perhaps you don't have a name in DNS, you can use an IP address directly as the hostname.

##Check it's there at all

Even a server with all services disabled will usually respond to simple pings, so if you know that your DNS is ok, check that the server is actually there:

```shell
ping smtp.gmail.com
```

You should see something like this (press ctrl-C to stop it):
```
PING gmail-smtp-msa.l.google.com (74.125.133.108): 56 data bytes
64 bytes from 74.125.133.108: icmp_seq=0 ttl=43 time=72.636 ms
64 bytes from 74.125.133.108: icmp_seq=1 ttl=43 time=68.841 ms
64 bytes from 74.125.133.108: icmp_seq=2 ttl=43 time=68.500 ms
```
##Check it's a mail server

It may be that some other service is running on the SMTP port you are trying to connect to. You can check this using the `telnet` tool, like this (connecting to gmail on its submission service port):
```
telnet smtp.gmail.com 587
```
This should give you something like this:

```
Trying 173.194.67.109...
Connected to gmail-smtp-msa.l.google.com.
Escape character is '^]'.
220 mx.google.com ESMTP ex2sm16805587wjd.30 - gsmtp
```

(Enter `quit` to get out of that). If port 587 doesn't work, you can try port 465 or port 25, and use whichever one works - though bear in mind that port 25 often doesn't support encryption (see encryption notes).

If it produces no output or something that doesn't start with `220`, then either your server is down or you've got the wrong server.

##Firewall redirection
Another thing to look out for here is that the name the mail server responds with should be related to the server you requested, as you can see in the above example - we asked for `smtp.gmail.com` and got `gmail-smtp-msa.l.google.com`, which looks like it's something to do with google - if instead you see something like the name of your ISP, then it could mean that your ISP's firewall is redirecting you transparently to their own mail servers, and you're likely to see authentication failures because you're logging into the wrong server. This is very likely to happen on port 25, but less likely to happen on ports 465 and 587, so it's yet another reason to use encryption!

##Authentication failures

If your authentication is failing, there are several likely causes:
* You have the wrong username or password
* Your connection is being diverted to a different server (as above)
* You have specified authentication without encryption

Generally you do not want to send a username or password over an unencrypted link. Some SMTP authentication schemes do add a minimal level of security (sending short hashes rather than clear text), but these provide only minimal protection, and so most servers do not allow authentication without encryption. Fix this by setting `SMTPSecure = 'tls'` and `Port = 587` as well as setting the `Username` and `Password` properties.

###Gmail and "Allow less secure apps"

Google have recently started imposing a different authentication mechanism for access to their apps, including gmail. This substitutes normal standards-based SMTP authentication for authorisation via OAuth2, and does not really improve security much at all - it simply means you have to enter your ID and password somewhere else (over the same protocols that they classify as insecure), and add complexity to your project to implement OAuth. This change breaks SMTP and IMAP access to gmail, and you may receive authentication failures (usually "5.7.1 Username and Password not accepted") from many email clients, including PHPMailer, Apple Mail, Outlook, Thunderbird and others.

To do this in PHP you'll need an OAuth2 client class - like [this one](http://www.phpclasses.org/package/7700-PHP-Authorize-and-access-APIs-using-OAuth.html) - to retrieve an authorisation token before making an SMTP connecton as normal. Google's docs on the subject are [here](https://developers.google.com/gmail/xoauth2_protocol).

In short, you *do* have to 'enable access for less secure apps', though it does not really make your app any less secure.

PHPMailer does not currently support this access mechanism, so if you get around to implementing this, please do submit a PR for it!

##Which kind of encryption should I use?
There's no doubt that you should use encryption at every opportunity, otherwise you're inviting all kinds of unpleasant possibilities for phishing, identity theft etc.

To use any kind of encryption you need the `openssl` PHP extension enabled. Check this by looking at the output of `phpinfo()` or `php -i` (look for an 'openssl' section), or `openssl` listed in the output of `php -m`, or run this line of code:
```php
<?php echo (extension_loaded('openssl')?'SSL loaded':'SSL not loaded')."\n"; ?>
```
As for what kind to use, the answer is generally simple: Don't use SSL on port 465, it's been [deprecated since 1998](http://en.wikipedia.org/wiki/SMTPS) and is only used by Microsoft products that didn't get the memo; use TLS on port 587 instead:

```php
$mail->SMTPSecure = 'tls';
$mail->Host = 'smtp.gmail.com';
$mail->Port = 587;
```
or more succinctly:

```php
$mail->Host = 'tls://smtp.gmail.com:587';
```

#It's still not working!
**If any of the above checks fail, PHPMailer will not work either**, and usually there's nothing that PHPMailer can do about it. So go fix your network, then try again. If you are not in control of your own firewall or DNS, you probably need to raise a support ticket with your ISP to fix this (it's very common for them to block or divert port 25 outbound). If they won't fix it, you need to replace your ISP.