Whatever problem you're having, first make sure you are using the [latest PHPMailer](https://github.com/PHPMailer/PHPMailer).

# Which kind of encryption should I use?
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
# SMTP Error: Could not connect to SMTP host.
This is often reported as a PHPMailer problem, but it's almost always down to local DNS failure, firewall blocking or other issue on your local network. It means that PHPMailer is unable to contact the SMTP server you have specified in the `Host` property, but doesn't say exactly why. It can also be caused by not having the `openssl` extension loaded (See above).

You can test your connectivity by running some commands on your server (you will need `dnsutils` and `telnet` packages installed on linux). First check DNS is working:

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

Next try to telnet to the host on the port you need (this example is using TLS for gmail):
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

(Enter `quit` to get out of that). If port 587 doens't work, you can try port 465 or port 25, and use whichever one works - though bear in mind that port 25 doesn't usually support encryption (see above).

**If these connection attempts fail, PHPMailer will not work either**. So go fix your network, then try again. If you are not in control of your own firewall or DNS, you probably need to raise a support ticket with your ISP to fix this (it's very common for them to block or divert port 25 outbound). If they won't fix it, you need to replace your ISP.

Back in PHPMailer, you can get verbose feedback on the connection and the whole SMTP conversation by setting:
```php
$mail->SMTPDebug = 4;
```