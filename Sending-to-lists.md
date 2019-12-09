PHPMailer is commonly used to send to mailing lists - indeed it is at the core of list managers like [PHPList](https://www.phplist.com) and [Smartmessages.net](https://info.smartmessages.net/).

You can find a simple example of sending to a list held in a MySQL database [here](https://github.com/PHPMailer/PHPMailer/blob/master/examples/mailing_list.phps).

Some basic optimizations can be seen in that script:
* Don't create a new PHPMailer instance every time around the loop.
* Enable `SMTPKeepAlive` and sort your list by domain to reduce SMTP overhead and maximise connection re-use.
* Set all the properties that will remain the same for every recipient (e.g. `From`, `FromName`, `Subject` etc) *before* the sending loop so they only happen once.
* Inside the loop you should have only calls to `addAddress()`, `send()` and `clearAddresses()` (after sending).

If the messages you send are absolutely identical, you can add all the addressees using `addBCC()`, which will mean you only need to `send()` a single message, though most mail servers will have a limit on the number of addresses you can send at once.

## Maximising performance


For ultimate performance, you should submit to a local mail server, though surprisingly using the `isMail()` or `isSendmail()` options in PHPMailer is not necessarily faster than SMTP to localhost, largely because postfix' `sendmail` binary opens a synchronous SMTP connection to localhost anyway - [postfix' docs recommend SMTP to localhost for best performance](http://www.postfix.org/TUNING_README.html#mailing_tips) for this reason.

If you are sending to localhost, don't use authentication or encryption as the overhead doesn't gain you anything, but if you are using a remote server, use `tls` on port 587 in preference to the obsolete `ssl` on port 465.

Generally sending directly to end users is to be avoided. The SMTP client in PHPMailer is not an MTA - it does not handle queuing at all, so any domains with greylisting or delivery deferrals for traffic control (in particular Yahoo) will fail to be delivered. The best approach is to use SMTP to a nearby MTA and leave the queue handling to that. You can get bounces back from that as well so you can remove bad addresses from your list.

You should make sure you have a local caching DNS - `dnsmasq` on Linux is a great choice.

## Performance Improvements on Windows systems

PHP on Windows may suffer from a 3-12 second delay when sending emails to a remote SMTP server. This is due to delay verifying the SSL certificate when sending over an encrypted SSL/STARTTLS connection. See more details and solution [here](https://github.com/PHPMailer/PHPMailer/issues/1911).