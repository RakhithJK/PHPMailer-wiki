This question comes up fairly regularly. There are a few causes that are quite common.

## Don't add multiple addresses for yourself
This might seem obvious, but some people forget:

```php
$mail->addAddress('myaddress@someisp.example.com');
$mail->addCC('myaddress@someotherisp.example.com');
```

## Don't call `send()` more than once
I've seen this structure as an inappropriate way of checking whether sending was successful:

```php
$mail->send();
if (!$mail->send()) {
    echo 'Message didn't send!';
}
```

This should be fixed by removing the first call to `send()`.

## Browser plugins causing multiple posts
Some browser plugins (and anti-virus products) cause repeated posts of forms when you hit submit. If this includes the same form content, you'll end up with multiple messages being sent, and this may happen too quickly for you to notice. You can identify whether your form handler script is being run more than once by altering the message that is sent, and the obvious place to do that is in the subject line:

```php
$mail->Subject = 'My subject line' . rand(0, 999999);
```
If a single invocation of your script sends multiple messages, they will end up with the same random number in the subject line. If your script is being run more than once, you will have different numbers in each subject line. Another way to check this is by looking at your web server's log files - multiple requests will result in multiple log entries.

CSRF tokens will prevent multiple submissions from being processed, but it's better to not have the repeat posts in the first place if it can be avoided.

## Mail server misconfiguration
It's possible that your mail server could be configured to send you a copy of some or all outbound messages, so a single message might be sent to you twice - use the subject line technique described above to figure out if this might be happening.

## Exploding mailing lists
This is a more extreme duplicate problem. If you're sending to a list, it's common to loop over it and add an address each time around. The important word there is **add**; when you call `addRecipient()` it adds another address to the recipient list, and they will all receive a copy of the message. For example:

```php
$mail = new PHPMailer;
//Set subject line, to, from etc
foreach ($mailinglist as $recipient) {
    $mail->addAddress($recipient['address'], $recipient['name']);
    $mail->send();
}
```
This will result in the first recipient in the list receiving the first message, the first & second recipients receiving the second message, the first, second, and third receiving the third message, and so on. If your list contains 100 people, this will result in **5050** messages being sent, and the first recipient will receive **100 copies**!
The trick is to reset the recipient list each time around, ensuring that you only get 100 messages sent, each to only one recipient:

```php
...
foreach ($mailinglist as $recipient) {
    $mail->addAddress($recipient['address'], $recipient['name']);
    $mail->send();
    $mail->clearAddresses();
}
```
See [the mailing list example](https://github.com/PHPMailer/PHPMailer/blob/master/examples/mailing_list.phps) for how to send to lists, and read the wiki doc about [sending to lists](https://github.com/PHPMailer/PHPMailer/wiki/Sending-to-lists) in general.