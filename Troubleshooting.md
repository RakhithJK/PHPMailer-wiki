#Troubleshooting PHPMailer Problems
Whatever problem you're having, first make sure you are using the [latest PHPMailer](https://github.com/PHPMailer/PHPMailer). If you have based your code on an example you found somewhere other than here on GitHub, it's very probably outdated - base your code on the examples in [the examples folder](https://github.com/PHPMailer/PHPMailer/tree/master/examples). About 90% of [questions on Stack Overflow](http://stackoverflow.com/questions/tagged/phpmailer) make this mistake.

##Loading classes
###Including the wrong file
Not so long ago, PHPMailer changed the way that it loaded classes so that it was more compatible with composer, many frameworks, and the [PHP PSR-0 autoloading standard](http://www.php-fig.org/psr/psr-0/). Note that because we support PHP back to version 5.0, we cannot support the more recent [PSR-4 standard](http://www.php-fig.org/psr/psr-4/), nor can we use namespaces. Previously, PHPMailer loaded the SMTP class explicitly, and this causes problems if you want to provide your own implementation. You may have seen old scripts doing this:

```php
require 'class.phpmailer.php';
```

If you do only that, **SMTP sending will fail** with a `Class 'SMTP' not found` error. You need to either explicitly include the `class.smtp.php` file, or use the recommended approaches of using composer or the supplied autoloader, like this:

```php
require 'PHPMailerAutoload.php';
```
###Using composer
[Composer](https://getcomposer.org) saves a huge amount of work - handling package dependencies, updates and downloading, and generates a nice autoloader so you don't have to `require` classes yourself. Loading PHPMailer via composer is the preferred method of using PHPMailer in your project.

It's particularly important if you're using the XOAUTH2 classes sicne they have dependencies that are satisfied by composer. The dependencies are not included by default because they don't work on the older PHP versions that PHP supports, so you will find them in the 'suggest' section of PHPMailer's `composer.json` file. You should copy those dependencies to your own `composer.json`'s `require` section, then `composer install` to load them and add them to the autoloader.

If you don't do this, you're likely to see errors like this:

```php
Fatal error: Class 'League\OAuth2\Client\Provider\Google' not found in PHPMailer/get_oauth_token.php on line 24
```

To fix this either configure composer as described, or download this class and all its dependencies and load them manually yourself.

##Enabling debug output
If you're using SMTP (i.e. you're calling `isSMTP()`), you can get a detailed transcript of the SMTP conversation using the `SMTPDebug` property. The settings are as follows:

* 1: show client -> server messages only
* 2: show client -> server and server -> client messages - this is usually the setting you want
* 3: As 2, but also show details about the initial connection; only use this if you're having trouble connecting (e.g. connection timing out)
* 4: As 3, but also shows detailed low-level traffic. Only really useful for analysing protocol-level bugs, very verbose, probably not what you need.

Set this option by including a line like this in your script:

    $mail->SMTPDebug = 2;

The output format will adapt itself to command-line or HTML output, though you can override this using the `Debugformat` property.

##"SMTP Error: Could not connect to SMTP host."
This may also appear as **`SMTP connect() failed`** or **`Called Mail() without being connected`** in debug output. This is often reported as a PHPMailer problem, but it's almost always down to local DNS failure, firewall blocking or other issue on your local network. It means that PHPMailer is unable to contact the SMTP server you have specified in the `Host` property, but doesn't say exactly why. It can also be caused by not having the `openssl` extension loaded (See encryption notes below).

Some techniques to diagnose the source of this error are discussed below.

##Read the SMTP transcript
If you set `SMTPDebug = 2` or higher, you will see what the remote SMTP server says. Very often this will tell you exactly what is wrong - things like "Incorrect password", or sometimes a URL of a page to help you diagnose the problem. **Read what it says**. Google does this a lot - see below for info about their "Allow less secure apps" setting.

##DNS failures

These are often seen as connection timeouts, or "Temporary failure in name resolution", "could not resolve host", "getaddrinfo failed" or similar errors. Check your DNS is working by using the `dig` tool (from the `dnsutils` package on Debian/Ubuntu):

```shell
dig +short smtp.gmail.com
```
You will get something like this if your DNS is working:

```shell
gmail-smtp-msa.l.google.com.
173.194.67.108
173.194.67.109
```
If this fails, PHPMailer will not be able to send email because it won't be able to obtain the correct IP address to connect to. If perhaps you don't have a name in DNS, you can use an IP address directly as the hostname. To fix this you need to figure out why your DNS isn't working - perhaps you have not set up your resolvers?

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

##SELinux blocking
If you see an error like `SMTP -> ERROR: Failed to connect to server: Permission denied (13)`, you may be running into SELinux preventing PHP or the web server from sending email. This is particularly likely on RedHat / Fedora / Centos.
Using the `getsebool` command we can check if the httpd daemon is allowed to make a connection over the network and send an email:

    getsebool httpd_can_sendmail
    getsebool httpd_can_network_connect

This command will return a boolean on or off. If it's off, we can turn it on:

    sudo setsebool -P httpd_can_sendmail 1
    sudo setsebool -P httpd_can_network_connect 1

If you're running PHP-FPM via fastcgi, you may need to apply this to the fpm daemon rather than httpd.

##IPv6 blocking
Some service providers (including Digital Ocean) provide IPv6 connectivity for servers, but block outbound SMTP over IPv6 while allowing it on IPv4. This can be worked around by setting the `Host` property to an IPv4 address explicitly (the `gethostbyname` function only does IPv4 lookups):

    $mail->Host = gethostbyname('smtp.gmail.com');

##Authentication failures

If your authentication is failing, there are several likely causes:
* You have the wrong username or password
* Your connection is being diverted to a different server (as above)
* You have specified authentication without encryption

Generally you do not want to send a username or password over an unencrypted link. Some SMTP authentication schemes do add a minimal level of security (sending short hashes rather than clear text), but these provide only minimal protection, and so most servers do not allow authentication without encryption. Fix this by setting `SMTPSecure = 'tls'` and `Port = 587` as well as setting the `Username` and `Password` properties.

###Gmail, OAuth2 and "Allow less secure apps"

From December 2014, Google started imposing an authentication mechanism called [XOAUTH2](https://developers.google.com/gmail/xoauth2_protocol) based on [OAuth2](http://oauth.net/2/) for access to their apps, including Gmail. This change can break both SMTP and IMAP access to gmail, and you may receive authentication failures (often "5.7.14 Please log in via your web browser") from many email clients, including PHPMailer, Apple Mail, Outlook, Thunderbird and others. The error output may include a link to https://support.google.com/mail/bin/answer.py?answer=78754, which gives a list of possible remedies. There are two main solutions to this in PHPMailer:
* Enabling "[Allow less secure apps](https://support.google.com/accounts/answer/6010255)" will usually solve the problem for PHPMailer, and it does not really make your app significantly less secure. Reportedly, changing this setting may take an hour or more to take effect, so don't expect an immediate fix.
* PHPMailer added support for XOAUTH2 in version 5.2.11, though **you must be running PHP 5.4 or later** in order to use it. Documentation on how to set it up can be found on [this wiki page](https://github.com/PHPMailer/PHPMailer/wiki/Using-Gmail-with-XOAUTH2).

##Using encryption
There's no doubt that you should use encryption at every opportunity, otherwise you're inviting all kinds of unpleasant possibilities for phishing, identity theft etc.

To use any kind of encryption you need the [`openssl` PHP extension](http://php.net/manual/en/book.openssl.php) enabled. If you don't have it installed, or it's misconfigured, you're likely to have trouble at the `STARTTLS` phase of connections. Check this by looking at the output of `phpinfo()` or `php -i` (look for an 'openssl' section), or `openssl` listed in the output of `php -m`, or run this line of code:
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

Don't mix up these modes either; valid combinations are `tls` on port 587 (or possibly 25) and `ssl` on port 465. `ssl` on port 587 or `tls` on port 465 *will not work*.
###Opportunistic TLS
PHPMailer 5.2.10 introduced opportunistic TLS - if it sees that the server is advertising TLS encryption (after you have connected to the server), it enables encryption automatically, even if you have not set `SMTPSecure`. This *might* cause issues if the server is advertising TLS with an invalid certificate, but you can turn it off with `$mail->SMTPAutoTLS = false;`.

##PHP 5.6 certificate verification failure
In a change from earlier versions, PHP 5.6 verifies certificates on SSL connections. If the SSL config of the server you are connecting to is not correct, you will get an error like this:

```
Warning: stream_socket_enable_crypto(): SSL operation failed with code 1.
OpenSSL Error messages: error:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed
```

The correct fix for this is to replace the invalid, misconfigured or self-signed certificate with a good one. Failing that, you can allow **insecure** connections via the `SMTPOptions` property introduced in PHPMailer 5.2.10 (it's possible to do this by [subclassing the SMTP class](https://github.com/PHPMailer/PHPMailer/wiki/Overriding-the-SMTP-class) in earlier versions), though this is not recommended:

```php
$mail->SMTPOptions = array(
    'ssl' => array(
        'verify_peer' => false,
        'verify_peer_name' => false,
        'allow_self_signed' => true
    )
);
```

You can also change these settings globally in your php.ini, but that's a **really** bad idea; PHP 5.6 made this change for very good reasons.

Sometimes this behaviour is not quite so apparent; sometimes encryption failures may appear as the client issuing a `QUIT` immediately after trying to do a `STARTTLS`. If you see that happen, you should check the state of your certificates or verification settings.

##cURL error 60
You may see the error `cURL error 60: SSL certificate problem: unable to get local issuer certificate`. This may be because your CA file is out of date or missing. You can [download the latest CA cert file from curl](https://curl.haxx.se/ca/cacert.pem), install it somewhere accessible and point at it from your php.ini file with the `openssl.cafile` and `curl.cainfo` properties.

This error can also be caused if your PHP is using a libcurl compiled with libressl (a common option on homebrew) which has [a bug relating to this](https://github.com/libressl-portable/portable/issues/80) instead of the default openssl or OS X's built-in Secure Transport - running `curl -V` will tell you what yours is compiled with, like this:

    curl 7.48.0 (x86_64-apple-darwin15.4.0) libcurl/7.48.0 OpenSSL/1.0.2g zlib/1.2.5 libssh2/1.7.0 nghttp2/1.9.2

A standard OS X installation will use Secure Transport:

    curl 7.43.0 (x86_64-apple-darwin15.0) libcurl/7.43.0 SecureTransport zlib/1.2.5

##Testing SSL outside PHP
In order to eliminate PHP config or your code from encryption issues, you can use your local openssl installation to test the config directly using its built-in SMTP client, for example:

    openssl s_client -starttls smtp -crlf -connect smtp.gmail.com:587

You should expect a response like this:
```
CONNECTED(00000003)
depth=2 /C=US/O=GeoTrust Inc./CN=GeoTrust Global CA
verify error:num=20:unable to get local issuer certificate
verify return:0
---
Certificate chain
 0 s:/C=US/ST=California/L=Mountain View/O=Google Inc/CN=smtp.gmail.com
   i:/C=US/O=Google Inc/CN=Google Internet Authority G2
 1 s:/C=US/O=Google Inc/CN=Google Internet Authority G2
   i:/C=US/O=GeoTrust Inc./CN=GeoTrust Global CA
 2 s:/C=US/O=GeoTrust Inc./CN=GeoTrust Global CA
   i:/C=US/O=Equifax/OU=Equifax Secure Certificate Authority
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIEgDCCA2igAwIBAgIIQKPDG0sroxQwDQYJKoZIhvcNAQELBQAwSTELMAkGA1UE
BhMCVVMxEzARBgNVBAoTCkdvb2dsZSBJbmMxJTAjBgNVBAMTHEdvb2dsZSBJbnRl
cm5ldCBBdXRob3JpdHkgRzIwHhcNMTYwNDA3MDkwMzU5WhcNMTYwNjMwMDgyMDAw
WjBoMQswCQYDVQQGEwJVUzETMBEGA1UECAwKQ2FsaWZvcm5pYTEWMBQGA1UEBwwN
TW91bnRhaW4gVmlldzETMBEGA1UECgwKR29vZ2xlIEluYzEXMBUGA1UEAwwOc210
cC5nbWFpbC5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDNsHDL
zDdAFIunNFHuvBgE3ri8CinYqrPwh8LPhxNo7gnIxSOIASzlBa1xm4uBpInsWJLK
RxcqjQfGRRki558+ed5L2TrX3uoznEGAsoptatSPDuDaSttHjKX6ZOjbBEAxHp4r
ozplRFucGma7WkF1XR7htjdofWFCVN/u++Bhp1vJwO9RY2iwywjIVGqY4V9hYGHo
O0PKJSHTkIPHKZS1hSuM5f2P197cKrQVrFYx2dDMowJlCq8eEf1sp+38UXQEfqjo
BYNAj29ihiwmvYC/bN+6gZcn+vsR2w77p8tkLLzqY/vZ67Una6Qa+eV4Bl5Kmwwk
D1C1MDdoHTI8HDl9AgMBAAGjggFLMIIBRzAdBgNVHSUEFjAUBggrBgEFBQcDAQYI
KwYBBQUHAwIwGQYDVR0RBBIwEIIOc210cC5nbWFpbC5jb20waAYIKwYBBQUHAQEE
XDBaMCsGCCsGAQUFBzAChh9odHRwOi8vcGtpLmdvb2dsZS5jb20vR0lBRzIuY3J0
MCsGCCsGAQUFBzABhh9odHRwOi8vY2xpZW50czEuZ29vZ2xlLmNvbS9vY3NwMB0G
A1UdDgQWBBSEsJM0ANFUgqu5Qc3/VU+gllUUOjAMBgNVHRMBAf8EAjAAMB8GA1Ud
IwQYMBaAFErdBhYbvPZotXb1gba7Yhq6WoEvMCEGA1UdIAQaMBgwDAYKKwYBBAHW
eQIFATAIBgZngQwBAgIwMAYDVR0fBCkwJzAloCOgIYYfaHR0cDovL3BraS5nb29n
bGUuY29tL0dJQUcyLmNybDANBgkqhkiG9w0BAQsFAAOCAQEAhttyyIAjATMjXG03
kLgoKwHAZQ4ViSe2pt/DEMDUJNXBfJ+v6SI9wBE3QRHz6P/m5LkwoBeOrSiaNsiW
CrSZiBGFAj6/OBUUciHIPc/dKMYRFZ61wPArXD0VFJBtCV7cBSVvU3aW0YMPoufR
8UtjlOaTnm7pLqViGRy65EUwztznVe7eIi91X3pKPjg+TkoJmsbRes1ySmeQ06LV
1cWGd2HMOapOHK+cyQP2Uuo4ZAo5Hgiy9nnDRMmvShT2dKbIv19JyrfXPguZ/E7I
6z/Z/Fi7ilSrrpx/Frd8XwRCNQJPWfd2cV6NqGLwNR2qSCA0gJaWdIvJYqITw0lL
cAh6QQ==
-----END CERTIFICATE-----
subject=/C=US/ST=California/L=Mountain View/O=Google Inc/CN=smtp.gmail.com
issuer=/C=US/O=Google Inc/CN=Google Internet Authority G2
---
No client certificate CA names sent
---
SSL handshake has read 3494 bytes and written 491 bytes
---
New, TLSv1/SSLv3, Cipher is AES128-SHA
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : TLSv1
    Cipher    : AES128-SHA
    Session-ID: 936F1A0663F5CE73943C00650C2FB2B9612E1F9819D38A7CD853DB9130D0E5EE
    Session-ID-ctx:
    Master-Key: C092C10C71219E0BE8358CD432120D94CA39B01EDDA8A7007B08D7E86A74B6A16B14345610255063E1B0A2DB55D86635
    Key-Arg   : None
    Start Time: 1460541074
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
250 SMTPUTF8
```

(just type "QUIT" to get out of that). Notice that the verify return code is 0, which indicates successful verification. The `verify error:num=20:unable to get local issuer certificate` is not a problem. You can make the same kind of connection to your own server, or using different ports, though if you connect to port 465 you should skip the `-starttls smtp` option.

##"Could not instantiate mail function"

This means that your PHP installation is not configured to call the `mail()` function correctly (e.g. `sendmail_path` is not set correctly in your `php.ini`), or you have no local mail server installed and configured. To fix this you need to do one or more of these things:
* Install a local mail server (e.g. postfix).
* Ensure that your `sendmail_path` points at the sendmail binary (usually `/usr/sbin/sendmail`) in your `php.ini`. Note that on Ubuntu/Debian you may have multiple `.ini` files in `/etc/php5/mods-available` and possibly other locations.
* Use `isSendmail()` and set the path to the sendmail binary in PHPMailer (`$mail->Sendmail = '/usr/sbin/sendmail';`).
* Use `isSMTP()` and send directly using SMTP.

#It's still not working!
**If any of the above checks fail, PHPMailer will not work either**, and usually there's nothing that PHPMailer can do about it. So go fix your network, then try again. If you are not in control of your own firewall or DNS, you probably need to raise a support ticket with your ISP to fix this (it's very common for them to block or divert port 25 outbound). If they won't fix it, you need to replace your ISP.
_PS: BlueHost doesn't support smtp.gmail.com, they want you to use their smtp server. The work around would be to use email associated with BlueHost and their host address Or send using mail() function in this case._

#Where else to get help?
Several resources are worth checking:
* [The code examples](https://github.com/PHPMailer/PHPMailer/tree/master/examples) provided with PHPMailer. Base your code on these, not some ancient example from 2003.
* [The API docs](http://phpmailer.github.io/PHPMailer/).
* [The code itself](https://github.com/PHPMailer/PHPMailer/blob/master/class.phpmailer.php) - it's very well commented.
* [The issue tracker](https://github.com/PHPMailer/PHPMailer/issues) - it's very likely a problem similar to yours has happened before, so **search in there** _before_ opening a ticket. If you do create an issue, be sure to include your code, preferably the minimum necessary to reproduce or define the problem so that we have a chance to see what you're seeing - saying "It doesn't work" is not a bug report!
* [StackOverflow](http://stackoverflow.com/questions/tagged/phpmailer) - there are a ton of PHPMailer questions on there, the vast majority of which could be fixed by reading this page! **Search the questions** for the error message you're seeing **before** posting a question. If you post a question on SO, make sure you tag it as `PHPMailer` so that we will see it and **please don't** open an issue here as well. The issue tracker here is intended for actual bugs in PHPMailer, not problems with your server.