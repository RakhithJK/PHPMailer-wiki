# Troubleshooting PHPMailer Problems

Whatever problem you're having, first **make sure you are using the [latest PHPMailer](https://github.com/PHPMailer/PHPMailer)**. If you have based your code on an example you found somewhere other than here on GitHub, it's very probably outdated - base your code on the examples in [the examples folder](https://github.com/PHPMailer/PHPMailer/tree/master/examples). About 90% of [questions on Stack Overflow](http://stackoverflow.com/questions/tagged/phpmailer) make this mistake, and is especially likely since the release of PHPMailer 6.0.

## Loading classes
### Using composer
[Composer](https://getcomposer.org) saves a huge amount of work - handling package dependencies, updates and downloading, and generates a nice autoloader so you don't have to `require` classes yourself. **Loading via composer is the preferred method of using PHPMailer in your project**. All you need to do is require the composer autoloader:

    require './vendor/autoload.php';

**This file will only exist if you have used composer to install PHPMailer**; it is not part of the PHPMailer package itself.

It's particularly important to use composer if you're using XOAUTH2 authentication since it requires many dependent classes that are satisfied by composer. The dependencies are not included by default because they are not needed by everyone and they don't work on the older PHP versions that PHPMailer supports, so you will find them in the 'suggest' section of PHPMailer's `composer.json` file. You should copy those dependencies to your own `composer.json`'s `require` section, then `composer update` to load them and add them to your autoloader.

If you don't do this, you're likely to see errors like this:

```php
Fatal error: Class 'League\OAuth2\Client\Provider\Google' not found in PHPMailer/get_oauth_token.php on line 24
```

To fix this either configure composer as described or download this class and all its dependencies and load them manually yourself.

### Using PHPMailer's own autoloader
**This only applies to the legacy 5.2 branch**. Not so long ago, PHPMailer changed the way that it loaded classes so that it was more compatible with composer, many frameworks, and the [PHP PSR-0 autoloading standard](http://www.php-fig.org/psr/psr-0/). Note that because 5.2 supports PHP back to version 5.0, we cannot support the more recent [PSR-4 standard](http://www.php-fig.org/psr/psr-4/), nor can we use namespaces. Previously, PHPMailer loaded the SMTP class explicitly, and this causes problems if you want to provide your own implementation. You may have seen old scripts doing this:

```php
require 'class.phpmailer.php';
```

If you do only that, **SMTP sending will fail** with a `Class 'SMTP' not found` error. You need to either explicitly include the `class.smtp.php` file (read the README for info on which files you need), or use the recommended approaches of using composer or the supplied autoloader, like this:

```php
require 'PHPMailerAutoload.php';
```
## Enabling debug output
If you're using SMTP (i.e. you're calling `isSMTP()`), you can get a detailed transcript of the SMTP conversation using the `SMTPDebug` property. The settings are as follows:

* `SMTP::DEBUG_OFF` (0): Normal production setting; no debug output.
* `SMTP::DEBUG_CLIENT` (1): show client -> server messages only. Don't use this - it's very unlikely to tell you anything useful.
* `SMTP::DEBUG_SERVER` (2): show client -> server and server -> client messages - this is usually the setting you want
* `SMTP::DEBUG_CONNECTION` (3): As 2, but also show details about the initial connection; only use this if you're having trouble connecting (e.g. connection timing out)
* `SMTP::DEBUG_LOWLEVEL` (4): As 3, but also shows detailed low-level traffic. Only really useful for analyzing protocol-level bugs, very verbose, probably not what you need.

Set this option by including a line like this in your script:

    $mail->SMTPDebug = SMTP::DEBUG_SERVER;

The output format will adapt itself to command-line or HTML output, though you can override this using the `Debugoutput` property. If you are using authentication, user IDs and passwords will be redacted in the debug output *except* when you use `SMTP::DEBUG_LOWLEVEL` (4).

## "SMTP Error: Could not connect to SMTP host."
This may also appear as **`SMTP connect() failed`**, **`Called Mail() without being connected`**, **`Network is unreachable (101)`** in debug output. This is often reported as a PHPMailer problem, but it's almost always down to local DNS failure, firewall blocking (for example as GoDaddy does), local anti-virus software, or another issue on your local network. It means that PHPMailer is unable to contact the SMTP server you have specified in the `Host` property, but doesn't say exactly why. It can also be caused by not having the `openssl` extension loaded (See encryption notes below).

Some techniques to diagnose the source of this error are discussed below.

### GoDaddy
Popular US hosting provider GoDaddy imposes very strict (to the point of becoming almost useless) constraints on sending an email. They block outbound SMTP to ports 25, 465 and 587 to all servers except their own. This problem is the subject of many frustrating [questions on Stack Overflow](http://stackoverflow.com/search?q=smtp+godaddy). If you find your script works on your local machine, but not when you upload it to GoDaddy, this will be what's happening to you. The solution is extremely poorly documented by GoDaddy: you **must** send through their servers, and also disable all security features, username, and password (great, huh?!), giving you this config for PHPMailer:

```php
$mail->isSMTP();
$mail->Host = 'localhost';
$mail->SMTPAuth = false;
$mail->SMTPAutoTLS = false; 
$mail->Port = 25; 
```

GoDaddy also refuses to send with a `From` address belonging to any aol, gmail, yahoo, hotmail, live, aim, or msn domain (see [their docs](https://www.godaddy.com/help/using-cdosys-to-send-email-from-your-windows-hosting-account-1073)). This is because all those domains deploy SPF and DKIM anti-forgery measures, and faking your from address is forgery.

## Read the SMTP transcript
If you set `SMTPDebug = SMTP::DEBUG_SERVER` or higher, you will see what the remote SMTP server says. Very often this will tell you exactly what is wrong - things like "Incorrect password", or sometimes a URL of a page to help you diagnose the problem. **Read what it says**. Google does this a lot - see below for info about their "Allow less secure apps" setting.

## DNS failures
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

## Check it's there at all
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
## Check it's a mail server

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

## Firewall redirection
Another thing to look out for here is that the name the mail server responds with should be related to the server you requested, as you can see in the above example - we asked for `smtp.gmail.com` and got `gmail-smtp-msa.l.google.com`, which looks like it's something to do with google - if instead you see something like the name of your ISP, then it could mean that your ISP's firewall is redirecting you transparently to their own mail servers, and you're likely to see authentication and TLS certificate verification failures (see below for more) because you're logging into the wrong server. This is very likely to happen on port 25, but less likely to happen on ports 465 and 587, so it's yet another reason to use encryption!
For GoDaddy Cpanel servers with WHM access helps disabling  "SMTP Restrictions" in "Home »Security Center »SMTP Restrictions"  (This feature prevents users from bypassing the mail server to send mail. It will allow only the MTA, mailman, and root to connect to remote SMTP servers.)

## SELinux blocking
If you see an error like `SMTP -> ERROR: Failed to connect to server: Permission denied (13)`, you may be running into SELinux preventing PHP or the web server from sending email. This is particularly likely on RedHat / Fedora / Centos.
Using the `getsebool` command we can check if the httpd daemon is allowed to make a connection over the network and send an email:

    getsebool httpd_can_sendmail
    getsebool httpd_can_network_connect

This command will return a boolean on or off. If it's off, we can turn it on:

    sudo setsebool -P httpd_can_sendmail 1
    sudo setsebool -P httpd_can_network_connect 1

If you're running PHP-FPM via fastcgi, you may need to apply this to the fpm daemon rather than httpd.

## IPv6 blocking
Some service providers (including Digital Ocean) provide IPv6 connectivity for servers but block outbound SMTP over IPv6 while allowing it on IPv4. This can be worked around by setting the `Host` property to an IPv4 address explicitly (the `gethostbyname` function only does IPv4 lookups):

    $mail->Host = gethostbyname('smtp.gmail.com');

The only issue with this approach is that you end up asking to connect to an explicit IPv4 address, which will usually cause you to fail certificate name checks. You can disable that (see `SMTPOptions` elsewhere in this doc), but that should be considered a poor workaround - the right solution is to fix your network.

Note: When using the Digital Ocean service check if your SMTP port is actually unlocked, as it is a US based company it contains a series of directives not to fall into spam, so you should ask for the unlock and follow steps to confirm with Digital Ocean the Purpose of sending your emails with PHPMailer.

## Authentication failures

If your authentication is failing, there are several likely causes:
* You have the wrong username or password
* Your connection is being diverted to a different server (as above)
* You have specified authentication without encryption

Generally, you do not want to send a username or password over an unencrypted link. Some SMTP authentication schemes do add a minimal level of security (sending short hashes rather than clear text), but these provide only minimal protection, and so most servers do not allow authentication without encryption. Fix this by setting `SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS` and `Port = 587` as well as setting the `Username` and `Password` properties.

### Gmail, OAuth2 and "Allow less secure apps"
From December 2014, Google started imposing an authentication mechanism called [XOAUTH2](https://developers.google.com/gmail/xoauth2_protocol) based on [OAuth2](http://oauth.net/2/) for access to their apps, including Gmail. This change can break both SMTP and IMAP access to Gmail, and you may receive authentication failures (often "5.7.14 Please log in via your web browser and then try again" or "Username and Password not accepted") from many email clients, including PHPMailer, Apple Mail, Outlook, Thunderbird and others. The error output may include a link to https://support.google.com/mail/bin/answer.py?answer=78754, which gives a list of possible remedies, or https://support.google.com/mail/?p=BadCredentials, which is largely unhelpful. There are two main solutions to this in PHPMailer:
* Gmail doesn't like unexpected or unfamiliar clients connecting to gmail accounts, so it may require you to log into your gmail account in your browser as usual (this will be mentioned in error output visible if you set `SMTPDebug = SMTP::DEBUG_SERVER`), or to visit the [unlock CAPTCHA page](https://www.google.com/accounts/DisplayUnlockCaptcha) mentioned in their support doc.
* Enabling "[Allow less secure apps](https://support.google.com/accounts/answer/6010255)" will usually solve the problem for PHPMailer, and it does not make your app significantly less secure. Reportedly, changing this setting may take an hour or more to take effect, so don't expect an immediate fix.
* PHPMailer added support for XOAUTH2 in version 5.2.11, though **you must be running PHP 5.5 or later** in order to use it. Documentation on how to set it up can be found on [this wiki page](https://github.com/PHPMailer/PHPMailer/wiki/Using-Gmail-with-XOAUTH2).

## Using encryption
You should use encryption at every opportunity, otherwise you're inviting all kinds of unpleasant possibilities for phishing, identity theft, eavesdropping, stolen credentials etc.

PHPMailer uses TLS encryption; TLS is simply the "new" (since 1998!) name for SSL. The two names are interchangeable.

The TLS / SSL config you use for email has nothing to do with any certificate you may use on your web site; you can still use encrypted email even if your site does not have a certificate.

### Check you have the openssl extension
To use any kind of encryption you need the [`openssl` PHP extension](http://php.net/manual/en/book.openssl.php) enabled. If you don't have it installed, or it's misconfigured, you're likely to have trouble at the `STARTTLS` phase of connections. Check this by looking at the output of `phpinfo()` or `php -i` (look for an 'openssl' section), or `openssl` listed in the output of `php -m`, or run this line of code:
```php
<?php echo (extension_loaded('openssl')?'SSL loaded':'SSL not loaded')."\n"; ?>
```

### Encryption flavours
There are two "flavours" of transport encryption available for email:
* "SMTPS", also referred to as "implicit" because it assumes that you're going to be using encryption right from the start of the connection. In PHPMailer this mode is selected by setting `SMTPSecure = PHPMailer::ENCRYPTION_SMTPS` (or `'ssl'`), and usually requires `Port = 465`.
* "SMTP+STARTTLS", also referred to as "explicit" because it initially connects _insecurely_ then explicitly asks for the connection to start using encryption. In PHPMailer this mode is selected by setting `SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS` (or `'tls'`), and usually requires `Port = 587` (defined in [RFC6409](https://tools.ietf.org/html/rfc6409)), though it can work on any port.

SMTPS on port 465 [deprecated in 1998](http://en.wikipedia.org/wiki/SMTPS) and was mostly only used by Microsoft; the standards recommended using SMTP+STARTTLS on port 587 instead. However, SMTPS on port 465 become a recommended solution again in 2018 in [RFC8314](https://tools.ietf.org/html/rfc8314).

```php
$mail->SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS;
$mail->Host = 'smtp.gmail.com';
$mail->Port = 587;
```
**or**
```php
$mail->SMTPSecure = PHPMailer::ENCRYPTION_SMTPS;
$mail->Host = 'smtp.gmail.com';
$mail->Port = 465;
```
**Don't mix up these modes**; `SMTPS` on port 587 or `SMTP_STARTTLS` on port 465 **will not work**.

### Opportunistic TLS
PHPMailer 5.2.10 introduced opportunistic TLS - if it sees that the server is advertising TLS encryption (after you have connected to the server), it enables encryption automatically, even if you have not set `SMTPSecure`. This *might* cause issues if the server is advertising TLS with an invalid certificate, but you can turn it off with `$mail->SMTPAutoTLS = false;`.

## Certificate verification failure
PHP versions since 5.6 verify certificates on SSL connections. If there's a problem relating to the certificate, you will get an error like this:

```
Warning: stream_socket_enable_crypto(): SSL operation failed with code 1.
OpenSSL Error messages: error:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed
```

You may not see this error; In implicit encryption mode (SMTPS) it may be hidden because there isn't a way for the channel to show messages - SMTP+STARTTLS is generally easier to debug because of this. In an SMTP transcript this will typically be shown as trying to send a `STARTTLS` command immediately followed by a `QUIT` command. It will also not be shown if you set `SMTPDebug = SMTP::DEBUG_CLIENT`; set it to at least `SMTP::DEBUG_SERVER` to see server responses.

There are three likely explanations and solutions for this error:

1. The server is publishing a bad, self-signed, or expired certificate - fix by replacing the certificate on your mail server. If you don't have access, ask whoever the admin is. You can run [some diagnostic tests](https://www.checktls.com) which will tell you whether the problem is at your end (if the tests pass) or the mail server's.
1. Your ISP is transparently redirecting your SMTP traffic to a different server - this is effectively a man-in-the-middle attack and is exactly the kind of thing that TLS is designed to protect you from. An appropriate fix here is to use your ISP's server explicitly - this is especially true for GoDaddy - but you may find this interferes with your ability to use some *from* addresses, especially if you're using common hosts like Gmail or Yahoo.
1. Your operating system or PHP configuration is using an out of date CA (certificate authority) certificate file, preventing it being able to verify a perfectly valid certificate from the server.

### Checking CA certificates
First of all find out where your PHP instance gets its CA certificates from:

    php -i | grep cafile
    openssl.cafile => /etc/ssl/cacert.pem => /etc/ssl/cacert.pem

Your location may be different, or not specified manually in your PHP config. If it's empty, it means it's relying on your OS default locations for CA certificates, and you'll need to consult your OS docs to find out where they are kept.

Ask openssl to test the connection using whatever CA cert bundle path you're using in the `CAfile` parameter:

    echo QUIT | openssl s_client -crlf -starttls smtp -CAfile /etc/ssl/cacert.pem -connect smtp.gmail.com:587

A successful result will look like this, where the `verify return` values are all `1`:

```
CONNECTED(00000003)
depth=2 OU = GlobalSign Root CA - R2, O = GlobalSign, CN = GlobalSign
verify return:1
depth=1 C = US, O = Google Trust Services, CN = Google Internet Authority G3
verify return:1
depth=0 C = US, ST = California, L = Mountain View, O = Google Inc, CN = smtp.gmail.com
verify return:1
---
Certificate chain
 0 s:/C=US/ST=California/L=Mountain View/O=Google Inc/CN=smtp.gmail.com
   i:/C=US/O=Google Trust Services/CN=Google Internet Authority G3
 1 s:/C=US/O=Google Trust Services/CN=Google Internet Authority G3
   i:/OU=GlobalSign Root CA - R2/O=GlobalSign/CN=GlobalSign
```
A bad result (suggesting you need to update or relocate your CA certs) would be:
```
CONNECTED(00000003)
depth=1 C = US, O = Google Trust Services, CN = Google Internet Authority G3
verify error:num=20:unable to get local issuer certificate
```

The OpenSSL output is not especially easy to read - you may find that [testssl.sh](https://testssl.sh) is better. Run it like this:

    ./testssl.sh --starttls smtp smtp.gmail.com:587

### Updating CA certificates
To update your CA certificates, make sure your operating system is fully up to date - CA certs are usually updated via OS updates. Alternatively, you can [download the latest CA cert file from curl](https://curl.haxx.se/ca/cacert.pem), install it somewhere accessible (for example `/etc/ssl/cacert.pem`) and point at it from the `openssl.cafile` and `curl.cainfo` directives in your php.ini file (this location will vary according to your OS and PHP config; where you need to put it is beyond the scope of PHPMailer!):

    openssl.cafile = /etc/ssl/cacert.pem
    curl.cainfo = /etc/ssl/cacert.pem

A highly recommended alternative is to use the [Certainty](https://packagist.org/packages/paragonie/certainty) package which ensures that you always have the latest CA cert bundle.

Failing that, you can allow **insecure** connections via the `SMTPOptions` property introduced in PHPMailer 5.2.10 (it's possible to do this by [subclassing the SMTP class](https://github.com/PHPMailer/PHPMailer/wiki/Overriding-the-SMTP-class) in earlier versions), though this is **not recommended** as it defeats much of the point of using a secure transport at all:

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

## cURL error 60
You may see the error `cURL error 60: SSL certificate problem: unable to get local issuer certificate`. This may be because your CA file is out of date or missing - see above for how to fix that.

This error can also be caused if your PHP is using a libcurl compiled with libressl (a common option on homebrew), which has [a bug relating to this](https://github.com/libressl-portable/portable/issues/80), instead of the default openssl or OS X's built-in Secure Transport - running `curl -V` will tell you what yours is compiled with, like this:

    curl 7.48.0 (x86_64-apple-darwin15.4.0) libcurl/7.48.0 OpenSSL/1.0.2g zlib/1.2.5 libssh2/1.7.0 nghttp2/1.9.2

A standard OS X installation will use Secure Transport:

    curl 7.43.0 (x86_64-apple-darwin15.0) libcurl/7.43.0 SecureTransport zlib/1.2.5

## Testing SSL outside PHP
In order to eliminate PHP config or your code from encryption issues, you can use your local openssl installation to test the config directly using its built-in SMTP client, for example to test an explicit SMTP+STARTTLS config:

    echo QUIT | openssl s_client -starttls smtp -crlf -connect smtp.gmail.com:587

or (if you're using implicit SSL on port 465)

    echo QUIT | openssl s_client -connect smtp.gmail.com:465

Of course this doesn't have to be gmail's server - you can substitute any mail server name in these commands.

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
...
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

(just type "QUIT" to get out of that). Notice that the verify return code is 0, which indicates successful verification. The `verify error:num=20:unable to get local issuer certificate` is not a problem.

## "Could not instantiate mail function"

This means that your PHP installation is not configured to call the `mail()` function correctly (e.g. `sendmail_path` is not set correctly in your `php.ini`), or you have no local mail server installed and configured. To fix this you need to do one or more of these things (in order of preference):
* Ideally, use `isSMTP()` and send directly using SMTP; it's faster, safer, and easier to debug than using `mail()`.
* Install a local mail server (e.g. postfix).
* Ensure that your `sendmail_path` points at the sendmail binary (usually `/usr/sbin/sendmail`) in your `php.ini`. Note that on Ubuntu/Debian you may have multiple `.ini` files in `/etc/php5/mods-available` and possibly other locations.
* Use `isSendmail()` and set the path to the sendmail binary in PHPMailer (`$mail->Sendmail = '/usr/sbin/sendmail';`). It's very unlikely this option is appropriate; evaluate other solutions first.

If you also see the message `More than one "from" person`, it's likely that your php.ini's `sendmail_path` property already contains a `-f` parameter, and so your code is trying to add a second envelope sender, which is not allowed.

# Addressing
It's important that you use valid email addresses. Every place that PHPMailer accepts an email address property, it expects an RFC821-format address, **not** an RFC822 one, for example `user@example.com`, **not** `Joe User <user@example.com>`. All the functions that accept an email address, like `addAddress` will return a boolean `true` if the address was accepted. Domain names containing non-ascii chars like `café.com` will use IDN 'punycode' format, which can't be evaluated properly until you ask PHPMailer to `send()`, so errors relating to them will appear later than for regular addresses.

# Messages end up in the spam folder
There is [a separate document about deliverability and spam filtering](https://github.com/PHPMailer/PHPMailer/wiki/Improving-delivery-rates,-avoiding-spam-filters).

# It's still not working!
**If any of the above checks fail, PHPMailer will not work either**, and usually there's nothing that PHPMailer can do about it. So go fix your network, then try again. If you are not in control of your own firewall or DNS, you probably need to raise a support ticket with your ISP to fix this (it's very common for them to block or divert port 25 outbound). If they won't fix it, you need to replace your ISP.

# Where else to get help?
Several resources are worth checking:
* [The code examples](https://github.com/PHPMailer/PHPMailer/tree/master/examples) provided with PHPMailer. Base your code on these, not some ancient example from 2003.
* [The API docs](http://phpmailer.github.io/PHPMailer/).
* [The code itself](https://github.com/PHPMailer/PHPMailer/blob/master/src/PHPMailer.php) - it's very well commented.
* [The issue tracker](https://github.com/PHPMailer/PHPMailer/issues) - it's very likely a problem similar to yours has happened before, so **search in there** _before_ opening a ticket. If you do create an issue, be sure to include your code, preferably the minimum necessary to reproduce or define the problem so that we have a chance to see what you're seeing - saying "It doesn't work" is not a bug report!
* [StackOverflow](http://stackoverflow.com/questions/tagged/phpmailer) - there are a ton of PHPMailer questions on there, the vast majority of which could be fixed by reading this page! **Search the questions** for the error message you're seeing **before** posting a question. If you post a question on SO, make sure you tag it as `PHPMailer` so that we will see it and **please don't** open an issue here as well. The issue tracker here is intended for actual bugs in PHPMailer, not problems with your server.# Troubleshooting PHPMailer Problems

Whatever problem you're having, first **make sure you are using the [latest PHPMailer](https://github.com/PHPMailer/PHPMailer)**. If you have based your code on an example you found somewhere other than here on GitHub, it's very probably outdated - base your code on the examples in [the examples folder](https://github.com/PHPMailer/PHPMailer/tree/master/examples). About 90% of [questions on Stack Overflow](http://stackoverflow.com/questions/tagged/phpmailer) make this mistake, and is especially likely since the release of PHPMailer 6.0.

## Loading classes
### Using composer
[Composer](https://getcomposer.org) saves a huge amount of work - handling package dependencies, updates and downloading, and generates a nice autoloader so you don't have to `require` classes yourself. **Loading via composer is the preferred method of using PHPMailer in your project**. All you need to do is require the composer autoloader:

    require './vendor/autoload.php';

**This file will only exist if you have used composer to install PHPMailer**; it is not part of the PHPMailer package itself.

It's particularly important to use composer if you're using XOAUTH2 authentication since it requires many dependent classes that are satisfied by composer. The dependencies are not included by default because they are not needed by everyone and they don't work on the older PHP versions that PHPMailer supports, so you will find them in the 'suggest' section of PHPMailer's `composer.json` file. You should copy those dependencies to your own `composer.json`'s `require` section, then `composer update` to load them and add them to your autoloader.

If you don't do this, you're likely to see errors like this:

```php
Fatal error: Class 'League\OAuth2\Client\Provider\Google' not found in PHPMailer/get_oauth_token.php on line 24
```

To fix this either configure composer as described or download this class and all its dependencies and load them manually yourself.

### Using PHPMailer's own autoloader
**This only applies to the legacy 5.2 branch**. Not so long ago, PHPMailer changed the way that it loaded classes so that it was more compatible with composer, many frameworks, and the [PHP PSR-0 autoloading standard](http://www.php-fig.org/psr/psr-0/). Note that because 5.2 supports PHP back to version 5.0, we cannot support the more recent [PSR-4 standard](http://www.php-fig.org/psr/psr-4/), nor can we use namespaces. Previously, PHPMailer loaded the SMTP class explicitly, and this causes problems if you want to provide your own implementation. You may have seen old scripts doing this:

```php
require 'class.phpmailer.php';
```

If you do only that, **SMTP sending will fail** with a `Class 'SMTP' not found` error. You need to either explicitly include the `class.smtp.php` file (read the README for info on which files you need), or use the recommended approaches of using composer or the supplied autoloader, like this:

```php
require 'PHPMailerAutoload.php';
```
## Enabling debug output
If you're using SMTP (i.e. you're calling `isSMTP()`), you can get a detailed transcript of the SMTP conversation using the `SMTPDebug` property. The settings are as follows:

* `SMTP::DEBUG_OFF` (0): Normal production setting; no debug output.
* `SMTP::DEBUG_CLIENT` (1): show client -> server messages only. Don't use this - it's very unlikely to tell you anything useful.
* `SMTP::DEBUG_SERVER` (2): show client -> server and server -> client messages - this is usually the setting you want
* `SMTP::DEBUG_CONNECTION` (3): As 2, but also show details about the initial connection; only use this if you're having trouble connecting (e.g. connection timing out)
* `SMTP::DEBUG_LOWLEVEL` (4): As 3, but also shows detailed low-level traffic. Only really useful for analyzing protocol-level bugs, very verbose, probably not what you need.

Set this option by including a line like this in your script:

    $mail->SMTPDebug = SMTP::DEBUG_SERVER;

The output format will adapt itself to command-line or HTML output, though you can override this using the `Debugoutput` property. If you are using authentication, user IDs and passwords will be redacted in the debug output *except* when you use `SMTP::DEBUG_LOWLEVEL` (4).

## "SMTP Error: Could not connect to SMTP host."
This may also appear as **`SMTP connect() failed`**, **`Called Mail() without being connected`**, **`Network is unreachable (101)`** in debug output. This is often reported as a PHPMailer problem, but it's almost always down to local DNS failure, firewall blocking (for example as GoDaddy does), local anti-virus software, or another issue on your local network. It means that PHPMailer is unable to contact the SMTP server you have specified in the `Host` property, but doesn't say exactly why. It can also be caused by not having the `openssl` extension loaded (See encryption notes below).

Some techniques to diagnose the source of this error are discussed below.

### GoDaddy
Popular US hosting provider GoDaddy imposes very strict (to the point of becoming almost useless) constraints on sending an email. They block outbound SMTP to ports 25, 465 and 587 to all servers except their own. This problem is the subject of many frustrating [questions on Stack Overflow](http://stackoverflow.com/search?q=smtp+godaddy). If you find your script works on your local machine, but not when you upload it to GoDaddy, this will be what's happening to you. The solution is extremely poorly documented by GoDaddy: you **must** send through their servers, and also disable all security features, username, and password (great, huh?!), giving you this config for PHPMailer:

```php
$mail->isSMTP();
$mail->Host = 'localhost';
$mail->SMTPAuth = false;
$mail->SMTPAutoTLS = false; 
$mail->Port = 25; 
```

GoDaddy also refuses to send with a `From` address belonging to any aol, gmail, yahoo, hotmail, live, aim, or msn domain (see [their docs](https://www.godaddy.com/help/using-cdosys-to-send-email-from-your-windows-hosting-account-1073)). This is because all those domains deploy SPF and DKIM anti-forgery measures, and faking your from address is forgery.

## Read the SMTP transcript
If you set `SMTPDebug = SMTP::DEBUG_SERVER` or higher, you will see what the remote SMTP server says. Very often this will tell you exactly what is wrong - things like "Incorrect password", or sometimes a URL of a page to help you diagnose the problem. **Read what it says**. Google does this a lot - see below for info about their "Allow less secure apps" setting.

## DNS failures
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

## Check it's there at all
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
## Check it's a mail server

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

## Firewall redirection
Another thing to look out for here is that the name the mail server responds with should be related to the server you requested, as you can see in the above example - we asked for `smtp.gmail.com` and got `gmail-smtp-msa.l.google.com`, which looks like it's something to do with google - if instead you see something like the name of your ISP, then it could mean that your ISP's firewall is redirecting you transparently to their own mail servers, and you're likely to see authentication and TLS certificate verification failures (see below for more) because you're logging into the wrong server. This is very likely to happen on port 25, but less likely to happen on ports 465 and 587, so it's yet another reason to use encryption!
For GoDaddy Cpanel servers with WHM access helps disabling  "SMTP Restrictions" in "Home »Security Center »SMTP Restrictions"  (This feature prevents users from bypassing the mail server to send mail. It will allow only the MTA, mailman, and root to connect to remote SMTP servers.)

## SELinux blocking
If you see an error like `SMTP -> ERROR: Failed to connect to server: Permission denied (13)`, you may be running into SELinux preventing PHP or the web server from sending email. This is particularly likely on RedHat / Fedora / Centos.
Using the `getsebool` command we can check if the httpd daemon is allowed to make a connection over the network and send an email:

    getsebool httpd_can_sendmail
    getsebool httpd_can_network_connect

This command will return a boolean on or off. If it's off, we can turn it on:

    sudo setsebool -P httpd_can_sendmail 1
    sudo setsebool -P httpd_can_network_connect 1

If you're running PHP-FPM via fastcgi, you may need to apply this to the fpm daemon rather than httpd.

## IPv6 blocking
Some service providers (including Digital Ocean) provide IPv6 connectivity for servers but block outbound SMTP over IPv6 while allowing it on IPv4. This can be worked around by setting the `Host` property to an IPv4 address explicitly (the `gethostbyname` function only does IPv4 lookups):

    $mail->Host = gethostbyname('smtp.gmail.com');

The only issue with this approach is that you end up asking to connect to an explicit IPv4 address, which will usually cause you to fail certificate name checks. You can disable that (see `SMTPOptions` elsewhere in this doc), but that should be considered a poor workaround - the right solution is to fix your network.

Note: When using the Digital Ocean service check if your SMTP port is actually unlocked, as it is a US based company it contains a series of directives not to fall into spam, so you should ask for the unlock and follow steps to confirm with Digital Ocean the Purpose of sending your emails with PHPMailer.

## Authentication failures

If your authentication is failing, there are several likely causes:
* You have the wrong username or password
* Your connection is being diverted to a different server (as above)
* You have specified authentication without encryption

Generally, you do not want to send a username or password over an unencrypted link. Some SMTP authentication schemes do add a minimal level of security (sending short hashes rather than clear text), but these provide only minimal protection, and so most servers do not allow authentication without encryption. Fix this by setting `SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS` and `Port = 587` as well as setting the `Username` and `Password` properties.

### Gmail, OAuth2 and "Allow less secure apps"
From December 2014, Google started imposing an authentication mechanism called [XOAUTH2](https://developers.google.com/gmail/xoauth2_protocol) based on [OAuth2](http://oauth.net/2/) for access to their apps, including Gmail. This change can break both SMTP and IMAP access to Gmail, and you may receive authentication failures (often "5.7.14 Please log in via your web browser and then try again" or "Username and Password not accepted") from many email clients, including PHPMailer, Apple Mail, Outlook, Thunderbird and others. The error output may include a link to https://support.google.com/mail/bin/answer.py?answer=78754, which gives a list of possible remedies, or https://support.google.com/mail/?p=BadCredentials, which is largely unhelpful. There are two main solutions to this in PHPMailer:
* Gmail doesn't like unexpected or unfamiliar clients connecting to gmail accounts, so it may require you to log into your gmail account in your browser as usual (this will be mentioned in error output visible if you set `SMTPDebug = SMTP::DEBUG_SERVER`), or to visit the [unlock CAPTCHA page](https://www.google.com/accounts/DisplayUnlockCaptcha) mentioned in their support doc.
* Enabling "[Allow less secure apps](https://support.google.com/accounts/answer/6010255)" will usually solve the problem for PHPMailer, and it does not make your app significantly less secure. Reportedly, changing this setting may take an hour or more to take effect, so don't expect an immediate fix.
* PHPMailer added support for XOAUTH2 in version 5.2.11, though **you must be running PHP 5.5 or later** in order to use it. Documentation on how to set it up can be found on [this wiki page](https://github.com/PHPMailer/PHPMailer/wiki/Using-Gmail-with-XOAUTH2).

## Using encryption
You should use encryption at every opportunity, otherwise you're inviting all kinds of unpleasant possibilities for phishing, identity theft, eavesdropping, stolen credentials etc.

PHPMailer uses TLS encryption; TLS is simply the "new" (since 1998!) name for SSL. The two names are interchangeable.

The TLS / SSL config you use for email has nothing to do with any certificate you may use on your web site; you can still use encrypted email even if your site does not have a certificate.

### Check you have the openssl extension
To use any kind of encryption you need the [`openssl` PHP extension](http://php.net/manual/en/book.openssl.php) enabled. If you don't have it installed, or it's misconfigured, you're likely to have trouble at the `STARTTLS` phase of connections. Check this by looking at the output of `phpinfo()` or `php -i` (look for an 'openssl' section), or `openssl` listed in the output of `php -m`, or run this line of code:
```php
<?php echo (extension_loaded('openssl')?'SSL loaded':'SSL not loaded')."\n"; ?>
```

### Encryption flavours
There are two "flavours" of transport encryption available for email:
* "SMTPS", also referred to as "implicit" because it assumes that you're going to be using encryption right from the start of the connection. In PHPMailer this mode is selected by setting `SMTPSecure = PHPMailer::ENCRYPTION_SMTPS` (or `'ssl'`), and usually requires `Port = 465`.
* "SMTP+STARTTLS", also referred to as "explicit" because it initially connects _insecurely_ then explicitly asks for the connection to start using encryption. In PHPMailer this mode is selected by setting `SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS` (or `'tls'`), and usually requires `Port = 587` (defined in [RFC6409](https://tools.ietf.org/html/rfc6409)), though it can work on any port.

SMTPS on port 465 [deprecated in 1998](http://en.wikipedia.org/wiki/SMTPS) and was mostly only used by Microsoft; the standards recommended using SMTP+STARTTLS on port 587 instead. However, SMTPS on port 465 become a recommended solution again in 2018 in [RFC8314](https://tools.ietf.org/html/rfc8314).

```php
$mail->SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS;
$mail->Host = 'smtp.gmail.com';
$mail->Port = 587;
```
**or**
```php
$mail->SMTPSecure = PHPMailer::ENCRYPTION_SMTPS;
$mail->Host = 'smtp.gmail.com';
$mail->Port = 465;
```
**Don't mix up these modes**; `SMTPS` on port 587 or `SMTP_STARTTLS` on port 465 **will not work**.

### Opportunistic TLS
PHPMailer 5.2.10 introduced opportunistic TLS - if it sees that the server is advertising TLS encryption (after you have connected to the server), it enables encryption automatically, even if you have not set `SMTPSecure`. This *might* cause issues if the server is advertising TLS with an invalid certificate, but you can turn it off with `$mail->SMTPAutoTLS = false;`.

## Certificate verification failure
PHP versions since 5.6 verify certificates on SSL connections. If there's a problem relating to the certificate, you will get an error like this:

```
Warning: stream_socket_enable_crypto(): SSL operation failed with code 1.
OpenSSL Error messages: error:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed
```

You may not see this error; In implicit encryption mode (SMTPS) it may be hidden because there isn't a way for the channel to show messages - SMTP+STARTTLS is generally easier to debug because of this. In an SMTP transcript this will typically be shown as trying to send a `STARTTLS` command immediately followed by a `QUIT` command. It will also not be shown if you set `SMTPDebug = SMTP::DEBUG_CLIENT`; set it to at least `SMTP::DEBUG_SERVER` to see server responses.

There are three likely explanations and solutions for this error:

1. The server is publishing a bad, self-signed, or expired certificate - fix by replacing the certificate on your mail server. If you don't have access, ask whoever the admin is. You can run [some diagnostic tests](https://www.checktls.com) which will tell you whether the problem is at your end (if the tests pass) or the mail server's.
1. Your ISP is transparently redirecting your SMTP traffic to a different server - this is effectively a man-in-the-middle attack and is exactly the kind of thing that TLS is designed to protect you from. An appropriate fix here is to use your ISP's server explicitly - this is especially true for GoDaddy - but you may find this interferes with your ability to use some *from* addresses, especially if you're using common hosts like Gmail or Yahoo.
1. Your operating system or PHP configuration is using an out of date CA (certificate authority) certificate file, preventing it being able to verify a perfectly valid certificate from the server.

### Checking CA certificates
First of all find out where your PHP instance gets its CA certificates from:

    php -i | grep cafile
    openssl.cafile => /etc/ssl/cacert.pem => /etc/ssl/cacert.pem

Your location may be different, or not specified manually in your PHP config. If it's empty, it means it's relying on your OS default locations for CA certificates, and you'll need to consult your OS docs to find out where they are kept.

Ask openssl to test the connection using whatever CA cert bundle path you're using in the `CAfile` parameter:

    echo QUIT | openssl s_client -crlf -starttls smtp -CAfile /etc/ssl/cacert.pem -connect smtp.gmail.com:587

A successful result will look like this, where the `verify return` values are all `1`:

```
CONNECTED(00000003)
depth=2 OU = GlobalSign Root CA - R2, O = GlobalSign, CN = GlobalSign
verify return:1
depth=1 C = US, O = Google Trust Services, CN = Google Internet Authority G3
verify return:1
depth=0 C = US, ST = California, L = Mountain View, O = Google Inc, CN = smtp.gmail.com
verify return:1
---
Certificate chain
 0 s:/C=US/ST=California/L=Mountain View/O=Google Inc/CN=smtp.gmail.com
   i:/C=US/O=Google Trust Services/CN=Google Internet Authority G3
 1 s:/C=US/O=Google Trust Services/CN=Google Internet Authority G3
   i:/OU=GlobalSign Root CA - R2/O=GlobalSign/CN=GlobalSign
```
A bad result (suggesting you need to update or relocate your CA certs) would be:
```
CONNECTED(00000003)
depth=1 C = US, O = Google Trust Services, CN = Google Internet Authority G3
verify error:num=20:unable to get local issuer certificate
```

The OpenSSL output is not especially easy to read - you may find that [testssl.sh](https://testssl.sh) is better. Run it like this:

    ./testssl.sh --starttls smtp smtp.gmail.com:587

### Updating CA certificates
To update your CA certificates, make sure your operating system is fully up to date - CA certs are usually updated via OS updates. Alternatively, you can [download the latest CA cert file from curl](https://curl.haxx.se/ca/cacert.pem), install it somewhere accessible (for example `/etc/ssl/cacert.pem`) and point at it from the `openssl.cafile` and `curl.cainfo` directives in your php.ini file (this location will vary according to your OS and PHP config; where you need to put it is beyond the scope of PHPMailer!):

    openssl.cafile = /etc/ssl/cacert.pem
    curl.cainfo = /etc/ssl/cacert.pem

A highly recommended alternative is to use the [Certainty](https://packagist.org/packages/paragonie/certainty) package which ensures that you always have the latest CA cert bundle.

Failing that, you can allow **insecure** connections via the `SMTPOptions` property introduced in PHPMailer 5.2.10 (it's possible to do this by [subclassing the SMTP class](https://github.com/PHPMailer/PHPMailer/wiki/Overriding-the-SMTP-class) in earlier versions), though this is **not recommended** as it defeats much of the point of using a secure transport at all:

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

## cURL error 60
You may see the error `cURL error 60: SSL certificate problem: unable to get local issuer certificate`. This may be because your CA file is out of date or missing - see above for how to fix that.

This error can also be caused if your PHP is using a libcurl compiled with libressl (a common option on homebrew), which has [a bug relating to this](https://github.com/libressl-portable/portable/issues/80), instead of the default openssl or OS X's built-in Secure Transport - running `curl -V` will tell you what yours is compiled with, like this:

    curl 7.48.0 (x86_64-apple-darwin15.4.0) libcurl/7.48.0 OpenSSL/1.0.2g zlib/1.2.5 libssh2/1.7.0 nghttp2/1.9.2

A standard OS X installation will use Secure Transport:

    curl 7.43.0 (x86_64-apple-darwin15.0) libcurl/7.43.0 SecureTransport zlib/1.2.5

## Testing SSL outside PHP
In order to eliminate PHP config or your code from encryption issues, you can use your local openssl installation to test the config directly using its built-in SMTP client, for example to test an explicit SMTP+STARTTLS config:

    echo QUIT | openssl s_client -starttls smtp -crlf -connect smtp.gmail.com:587

or (if you're using implicit SSL on port 465)

    echo QUIT | openssl s_client -connect smtp.gmail.com:465

Of course this doesn't have to be gmail's server - you can substitute any mail server name in these commands.

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
...
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

(just type "QUIT" to get out of that). Notice that the verify return code is 0, which indicates successful verification. The `verify error:num=20:unable to get local issuer certificate` is not a problem.

## "Could not instantiate mail function"

This means that your PHP installation is not configured to call the `mail()` function correctly (e.g. `sendmail_path` is not set correctly in your `php.ini`), or you have no local mail server installed and configured. To fix this you need to do one or more of these things (in order of preference):
* Ideally, use `isSMTP()` and send directly using SMTP; it's faster, safer, and easier to debug than using `mail()`.
* Install a local mail server (e.g. postfix).
* Ensure that your `sendmail_path` points at the sendmail binary (usually `/usr/sbin/sendmail`) in your `php.ini`. Note that on Ubuntu/Debian you may have multiple `.ini` files in `/etc/php5/mods-available` and possibly other locations.
* Use `isSendmail()` and set the path to the sendmail binary in PHPMailer (`$mail->Sendmail = '/usr/sbin/sendmail';`). It's very unlikely this option is appropriate; evaluate other solutions first.

If you also see the message `More than one "from" person`, it's likely that your php.ini's `sendmail_path` property already contains a `-f` parameter, and so your code is trying to add a second envelope sender, which is not allowed.

# Addressing
It's important that you use valid email addresses. Every place that PHPMailer accepts an email address property, it expects an RFC821-format address, **not** an RFC822 one, for example `user@example.com`, **not** `Joe User <user@example.com>`. All the functions that accept an email address, like `addAddress` will return a boolean `true` if the address was accepted. Domain names containing non-ascii chars like `café.com` will use IDN 'punycode' format, which can't be evaluated properly until you ask PHPMailer to `send()`, so errors relating to them will appear later than for regular addresses.

# Messages end up in the spam folder
There is [a separate document about deliverability and spam filtering](https://github.com/PHPMailer/PHPMailer/wiki/Improving-delivery-rates,-avoiding-spam-filters).

# It's still not working!
**If any of the above checks fail, PHPMailer will not work either**, and usually there's nothing that PHPMailer can do about it. So go fix your network, then try again. If you are not in control of your own firewall or DNS, you probably need to raise a support ticket with your ISP to fix this (it's very common for them to block or divert port 25 outbound). If they won't fix it, you need to replace your ISP.

# Where else to get help?
Several resources are worth checking:
* [The code examples](https://github.com/PHPMailer/PHPMailer/tree/master/examples) provided with PHPMailer. Base your code on these, not some ancient example from 2003.
* [The API docs](http://phpmailer.github.io/PHPMailer/).
* [The code itself](https://github.com/PHPMailer/PHPMailer/blob/master/src/PHPMailer.php) - it's very well commented.
* [The issue tracker](https://github.com/PHPMailer/PHPMailer/issues) - it's very likely a problem similar to yours has happened before, so **search in there** _before_ opening a ticket. If you do create an issue, be sure to include your code, preferably the minimum necessary to reproduce or define the problem so that we have a chance to see what you're seeing - saying "It doesn't work" is not a bug report!
* [StackOverflow](http://stackoverflow.com/questions/tagged/phpmailer) - there are a ton of PHPMailer questions on there, the vast majority of which could be fixed by reading this page! **Search the questions** for the error message you're seeing **before** posting a question. If you post a question on SO, make sure you tag it as `PHPMailer` so that we will see it and **please don't** open an issue here as well. The issue tracker here is intended for actual bugs in PHPMailer, not problems with your server.