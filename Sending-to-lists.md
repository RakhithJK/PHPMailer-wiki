PHPMailer is commonly used to send to mailing lists - indeed it is at the core of list managers like [PHPList](https://www.phplist.com) and [Smartmessages.net](https://info.smartmessages.net/).

You can find a simple example of sending to a list held in a MySQL database [here](https://github.com/PHPMailer/PHPMailer/blob/master/examples/mailing_list.phps).

## Maximising performance

Enable `SMTPKeepAlive` and sort your list by domain to reduce SMTP overhead and maximise connection re-use.

For ultimate performance, you should submit to a local mail server, though surprisingly using the `isMail()` or `isSendmail()` options in PHPMailer is not necessarily faster than SMTP to localhost, largely because postfix' `sendmail` binary opens a synchronous SMTP connection to localhost anyway - [postfix' docs recommend SMTP to localhost for best performance](http://www.postfix.org/TUNING_README.html#mailing_tips) for this reason.

If you are sending to localhost, don't use authentication or encryption as the overhead doesn't gain you anything, but if you are using a remote server, use `tls` on port 587 in preference to the obsolete `ssl` on port 465.

Generally sending directly to end users is to be avoided. The SMTP client in PHPMailer is not an MTA - it does not handle queuing at all, so any domains with greylisting or delivery deferrals for traffic control (in particular Yahoo) will fail to be delivered. The best approach is to use SMTP to a nearby MTA and leave the queue handling to that. You can get bounces back from that as well so you can remove bad addresses from your list.

You should make sure you have a local caching DNS - `dnsmasq` on Linux is a great choice.