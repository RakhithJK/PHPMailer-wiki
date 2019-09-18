If you find your messages are being classified as spam or being rejected outright, there are many reasons why this could be, most of which are unrelated to your code.

# How to tell **why** your message is being rejected or classified as spam

There are several sources of information about message delivery:
1. Your mail server's logs will show details of immediate rejections, so look there if messages are not arriving at all.
1. Bounce messages - if a message is accepted and later rejected, it will generate a bounce message. Ensure that your mail server is set to forward these to somewhere you can see them, or pipes them into a script to be processed.
1. Message headers - if a message arrives in a spam folder, the receiving server will usually add headers telling you why it ended up there. Most email clients have the ability to view the raw message source: in Apple Mail, select View -> Message -> All Headers. In gmail select "Show original" from the message menu; this view also reports other useful details that are relevant to the items discussed below.
1. Postmaster tools. Some large ISPs, such as [gmail](https://postmaster.google.com/) and [hotmail](https://sendersupport.olc.protection.outlook.com/snds/), provide services to help senders identify sources of problem email. Unfortunately gmail's tools are extremely poor and unreliable.

# Causes of rejection and spam filtering

## Forging the from address

This is probably the most common reason for spam filtering. A really common anti-pattern is this on a typical contact form:

    $mail->From($_POST['email']);

This uses an address supplied by the submitter as the message's from address; it's very unlikely that your server will have permission to do that, and it will often be rejected completely, not just spam filtered. Do this instead:

    $mail->From('me@example.com'); //Your own email address
    $mail->addReplyTo($_POST['email']);

This way you are not forging anything, but replies will go to the form submitter's address. [The contact form example](https://github.com/PHPMailer/PHPMailer/blob/master/examples/contactform.phps) uses this approach.

## Missing or incorrect SPF record

[Sender Policy Framework](https://tools.ietf.org/html/rfc7208) (SPF) is a system that allows you to say which servers are allowed to send email from your domain. If you try to send from a server that is not listed in your SPF record, it will usually be rejected immediately. This means for example that you can't send from a gmail or Yahoo address *unless you send through gmail or yahoo mail servers*. If your domain lacks an SPF record at all, you'll get a `neutral` rating, which won't help you get through spam filters. Set one up and [check it here](https://www.kitterman.com/spf/validate.html). You must have SPF configured if you're going to get value from using DMARC as well.

## Missing or failing DKIM signatures

[Domain Keys Identified Mail](https://tools.ietf.org/html/rfc6376) (DKIM) is complementary to SPF (i.e. you should use both). DKIM provides a cryptographic proof that a message was sent by who it claims to be sent by (i.e. it's a good anti-phishing measure), and has not been tampered with in transit. These rules are applied to the `MAIL FROM` address at the SMTP level (known as the "envelope sender"), not within the message content (see DMARC for more).

Messages containing valid DKIM signatures more likely to be trusted, and less likely to be classified as spam, however, if you are sending spam, then spam filters know that it really is you that is sending it, so it cuts both ways.

PHPMailer can generate DKIM signatures, but it's better to have your mail server do it for you. Gmail for example will do this.

## Mismatched reverse DNS for your mail server

When your mail server connects to a recipient's mail server it sends an SMTP command identifying itself called `EHLO`. This contains the host and domain name of your server, which may be something like `mail.example.com`. The receiving server also sees the IP address your mail server is connecting from, and does a reverse lookup of your IP address - the result should match the name provided with the EHLO command, that is, a lookup for the name should result in the IP address, and looking up the IP address should return the same name. If these two do not match, your connection may be rejected.

## Message content tripping spam filters

This is unfortunately very difficult to avoid because content-based filters are often based on fixed string matching - for example it should be entirely possible to have a legitimate conversation about viagra or teenagers without tripping a spam filter, however, in practice it's effectively impossible. If you're talking about a sensitive subject, it's very hard to avoid this problem.

## A history of sending spam

If your IP address was previously used for sending spam, messages sent later may be classified as spam even if there is nothing wrong with them now. The only way to avoid this is not to send spam in the first place, but you can gain some insight into your reputation using services such as [SenderScore](https://www.senderscore.org). It's entirely possible to end up classified this way even if you have never sent spam - if you have not implemented SPF and DMARC, spammers can send spam using your domain name, resulting in damage to your reputation that you had nothing to do with - the message here is to implement strict SPF and DMARC so that this is not possible.

# Using DMARC

[Domain-based Message Authentication, Reporting, and Conformance](https://tools.ietf.org/html/rfc7489) (DMARC) provides an additional layer of authentication on top of SPF and DKIM. SPF validates the message source, DKIM validates the envelope sender, but that's not a complete set of validations; DMARC adds validation of the from address _within_ the message (in the `From` header), ensuring that it matches the SMTP envelope sender - it refers to this as "alignment". 

DMARC also provides a mechanism to tell senders what to do if any of its checks fail - for example it's possible to tell senders that they should deliver messages anyway, even if such checks fail. DMARC can also be configured to send summaries of validation failures on a daily basis, and also to forward all rejected messages to somewhere else. DMARC thus provides assurance that mail from your domain cannot be forged, but also lets you know when someone is trying to do so. This is a critical line of defence against phishing attacks.

One historical artefact of DMARC means that your default SPF mechanism should be `~all` and not `-all`; some SPF implementations were written before DMARC existed, and a `-all` mechanism can prevent DMARC from operating properly. If you're not using DMARC, it is reasonable to use `-all`.

Ultimately, you want to aim for DMARC settings that say that:

* all email from your domain should be validated with both SPF and DMARC
* envelope and message from addresses should always match
* any messages not meeting these rules should be rejected outright and not just spam filtered (or "quarantined")
* any validation failures should be reported to you

In DMARC syntax, this equates to a DMARC record that contains this amongst its other settings:

    p=reject; pct=100; fo=1

Being able to do this requires that you nail down your own email infrastructure very firmly, and to be fully aware of the implications of such strict settings - so don't just stick this in your DNS as it will likely result in rejection of legitimate messages. For example, if you have a web site that sends email from a contact form, you would need to ensure that it sends via your domain's designated mail server, and not one belonging to your hosting provider - this presents problems with hosting providers that block outbound SMTP, such as GoDaddy.