If you are having problems connecting or sending emails through your SMTP server, the SMTP class can provide more information about the processing/errors taking place.
Use the debug functionality of the class to see what's going on in your connections. To do that, set the debug level in your script. For example:

```php
$mail->SMTPDebug = SMTP::DEBUG_SERVER;
$mail->SMTPDebug = 2; //Alternative to above constant
$mail->isSMTP();  // tell the class to use SMTP
$mail->SMTPAuth   = true;                // enable SMTP authentication
$mail->Port       = 25;                  // set the SMTP port
$mail->Host       = "mail.yourhost.com"; // SMTP server
$mail->Username   = "name@yourhost.com"; // SMTP account username
$mail->Password   = "your password";     // SMTP account password
```

## Debug levels

Setting the `PHPMailer->SMTPDebug` property to these numbers or constants (defined in the `SMTP` class) results in different amounts of output:

 * `SMTP::DEBUG_OFF` (`0`): Disable debugging (you can also leave this out completely, 0 is the default).
 * `SMTP::DEBUG_CLIENT` (`1`): Output messages sent by the client.
 * `SMTP::DEBUG_SERVER` (`2`): as 1, plus responses received from the server (this is the most useful setting).
 * `SMTP::DEBUG_CONNECTION` (`3`): as 2, plus more information about the initial connection - this level can help diagnose STARTTLS failures.
 * `SMTP::DEBUG_LOWLEVEL` (`4`): as 3, plus even lower-level information, very verbose, don't use for debugging SMTP, only low-level problems.

You don't need to use levels above 2 unless you're having trouble connecting at all - it will just make output more verbose and more difficult to read.

Note that you will get no output until you call `send()`, because no SMTP conversation takes place until you do that.

## Debug output format

The form that the debug output takes is determined by the `Debugoutput` property. This has several options:

 * `echo` Output plain-text as-is, appropriate for CLI
 * `html` Output escaped, line breaks converted to `<br>`, appropriate for browser output
 * `error_log` Output to error log as configured in php.ini

By default PHPMailer will use `echo` if run from a `cli` or `cli-server` SAPI, `html` otherwise. Alternatively, you can implement your own system by providing a `callable` expecting two parameters: a message string and the debug level:

    $mail->Debugoutput = function($str, $level) {echo "debug level $level; message: $str";};

You can of course make this as complex as you like - for example you could capture all the output and store it in a database.

And finally, don't forget to disable debugging before going into production.
