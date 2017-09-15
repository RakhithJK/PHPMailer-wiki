# Introduction

This tutorial describes how to use PHPMailer.

**Note that this tutorial is based on an old version of PHPMailer, and parts of it no longer work the same way with PHPMailer 6.0. Please refer to the readme file.**

Based on a tutorial by Tom Klingenberg

# Contents

* PHPMailer: What is it? What does it do? Who needs it? Brief examples of PHPMailer's features.
* First time: Sending your first email with PHPMailer.
* Using Attachments: Sending mails with file attachments: filesystem, database or inline.
* Using HTML Mail: Using the class' integrated HTML email features.
* Support: Where do I get help?: How to find support resources: PHPMailer website, mailing list etc.
* Other Resources: Email and PHP Resources: There is a lot of information in the net about email, php and PHPMailer.

# PHPMailer

PHPMailer is a class library for [PHP](http://php.net/) that provides a collection of functions to build and send email messages. PHPMailer supports several ways of sending email: `mail()`, Sendmail, qmail & direct to SMTP servers. You can use any feature of SMTP-based e-mail, multiple recepients via to, CC, BCC, etc. In short: PHPMailer is an efficient way to send e-mail within PHP.

PHP has a built-in `mail()` function. So why use PHPMailer? Isn't it slower? Not really, because before you can send a message you have to construct one correctly, and this is extremely complicated because there are so many technical considerations. PHPMailer makes it easy to send e-mail, makes it possible to attach files, send HTML e-mail, etc. With PHPMailer you can even use your own SMTP server and avoid Sendmail routines used by the `mail()` function on Unix platforms.

This tutorial explains how to implement the class in your script or website and how to build an e-mail application.

Because using `mail()` has so many hidden problems, we strongly suggest you don't call it yourself - if you don't want to use PHPMailer, use another established email library such as SwiftMailer, Zend_Mail etc. Almost every code example you will find online using `mail()` (including the PHP documentation for the `mail()` function) has problems that can be avoided by using a library.

# First time

Before continuing, please be sure that PHPMailer is installed correctly. If you feel uncertain, please read the installion instructions that accompany the package. If you're still not sure, you can verify that you've installed PHPMailer correctly with this little script:

````php
<?php
require 'PHPMailerAutoload.php';
$mail = new PHPMailer;
````

Save it as a PHP document in the same directory where you've saved `class.phpmailer.php`. If no errors result from running the script, your installation has been done correctly. If you switched error messages and warnings off, then go ahead and set it on; refer to your PHP manual for more on this topic.

This snippet of code is also the start of any use of PHPMailer. `require 'PHPMailerAutoload.php';` loads an autoloader that knows how to load the other classes used by PHPMailer; `$mail = new PHPMailer;` creates a new instance of the class as `$mail`; All features, functions and methods of the class may be accessed via this variable.

Let's go ahead and send out the first mail. For this you'll require the basics: A recipient address, a from address, a subject and the text of the message.

You will also need to select a method of delivering the message. We need a program to communicate with an SMTP server which, in turn, sends the mail out to the internet. In the Unix/Linux world, Sendmail is very popular and used widely, but nearly all other mail servers, such as postfix and exim, provide a compatibility wrapper.

PHPMailer provides several methods of sending mail. It's best to start with the `mail()` function as that's the simplest method (and PHPmailer's default sending mechanism), but requires that you have a working local mail server.
Having gathered the necessary information to send your first e-mail, use that information in place of the example information provided here:

```php
<?php
require 'PHPMailerAutoload.php';
$mail = new PHPMailer;
$mail->setFrom('from@example.com', 'Your Name');
$mail->addAddress('myfriend@example.net', 'My Friend');
$mail->Subject  = 'First PHPMailer Message';
$mail->Body     = 'Hi! This is my first e-mail sent through PHPMailer.';
if(!$mail->send()) {
  echo 'Message was not sent.';
  echo 'Mailer error: ' . $mail->ErrorInfo;
} else {
  echo 'Message has been sent.';
}
```

Save this as a php file and change the values to your values, of course. Here is an explanation of what each stanza in this script does:

```php
$mail->setFrom('from@example.com', 'Your Name');
```
Enter the address that the e-mail should appear to come from. You can use any address that the SMTP server will accept as valid. The optional second parameter to this function is the name that will be displayed as the sender instead of the email address itself.

The following will add an address to which the e-mail will be sent. You must use a valid e-mail here  so that you can verify that your PHPMailer test worked. Just your own e-mail address here for this test. As with the `setFrom` method, you may optionally provide a display name for the recipient.

```php
$mail->addAddress('myfriend@example.net', 'My Friend');
```

Setting the subject and body is done next by setting the `Subject` and `Body` properties directly - note that they are case-sensitive, so don't try to set `subject` or `body`.

Finally, we send out the e-mail, once all necessary information has been provided. This is done with `$mail->Send();`. In this example script, it's inside an `if` statement; if `send()` fails, it'll return `false` and you can catch it and display an error message. This is done in the last lines. Otherwise it displays a success message.

# Using Attachments
Sending plain-text e-mails is often insufficient. Perhaps you need to attach something to your mail, such as an image or an audio file. Or perhaps you need to attach multiple files.

There are two ways of attaching something to your mail: You can simply attach a file from the filesystem or you can attach (binary) data stored in a variable. The latter is called string attachment. This makes it possible put extract data from a database or web API call and attach it to an e-mail, without ever having to save it as a file.

## File Attachments

The command to attach a local file is simply `$mail->addAttachment($path);`, where `$path` contains the path to the file you want to send, and can be placed anywhere between `$mail = new PHPMailer;` and sending the message. Note that *you cannot use a URL for the path* - you may only use local filesystem path. See notes on string attachments below for how to use remote content.

The $path be a relative one (from your script, not the PHPMailer class) or a full path to the file you want to attach. If you want to send content from a database or web API (e.g. a remote PDF generator), do not use this method - use `addStringAttachment` instead.

If you want more options or you want to specify encoding and the MIME type of the file, then you can use three more parameters, all of which are optional:

```php
$mail->addAttachment($path, $name, $encoding, $type);
```

`$name` is an optional parameter, used to set the name of the file that will be embedded within the e-mail. The person who will recieve your mail will then only see this name, rather than the original filename.

`$encoding` is a little more technical, but with this parameter you can set the type of encoding of the attachment. The default is `base64`. Other types that are supported include: `7bit`, `8bit`, `binary` & `quoted-printable`. Generally any binary file (e.g. an image) should use `base64`; text-based attachments will usually use `quoted-printable`.

`$type` is the MIME type of the attached file. Content types are defined not necessarily by file suffixes (i.e., `.jpg` or `.mp3`), but a MIME type (MIME = Multipurpose Internet Mail Extensions) is used. This parameter makes it possible change the MIME type of an attachment from the default value of `application/octet-stream` (which works with every kind of file, but means the receiver may not handle it correctly) to a more specific MIME type, such as `image/jpeg` for a JPEG photo, for instance. You don't usually need to set this yourself as PHPMailer will map it from the file extension automatically.

## String Attachments

The `addStringAttachment()` method works just like `addAttachment()`, but you pass the actual contents of the item instead of a file system path. The `$filename` parameter is required as it's used to provide a filename for the string data at the receiver end.

It also accepts the same other parameters as described above.

So, why use `addStringAttachment()` instead of `addAttachment()`? Is it for text-only files? No, not at all. It's primarily for databases and other non-file content. Data stored in a database is always stored as a string (or a BLOB: Binary Large OBject). You could query your database for an image stored as a BLOB and pass the resulting string to the `addStringAttachment()`.

If you want to use a remote URL for getting your content (for example getting PDF content from a remote URL), just do this:

```php
$mail->addStringAttachment(file_get_contents($url), 'myfile.pdf');
```

## Inline Attachments

If you want to make a HTML e-mail that refers to images that are also attached to the message (as opposed to pointed at remotely), it's necessary to attach the image with a content identifier and then link the tag to it. For example, if you add an image as inline attachment with the content ID of `my-photo`, you would access it within the HTML body using `<img src="cid:my-photo" alt="my-photo">`.

In detail, here is the function to add an embedded attachment:

```php
$mail->addEmbeddedImage($filename, $cid);
```
The process of connecting image tags to content identifiers is a bit complicated, but the `msgHTML()` method can do most of the work for you.

For more Information about HTML Email, see the section Using HTML E-Mail.

# Using HTML Email

Sending out HTML e-mail is a simple enough task with PHPMailer, though it can require significant knowledge of HTML. In particular, mail clients vary greatly in their rendering of HTML e-mail (far more than web browsers do), with some refusing to show it entirely. For those mail clients which are not able to display HTML, you can provide an alternate e-mail body containing the message as plain text.

First we'll create a basic HTML message:

```php
<?php
require 'PHPMailerAutoload.php';
$mail = new PHPMailer;
$mail->setFrom('from@example.com', 'Your Name');
$mail->addAddress('myfriend@example.net', 'My Friend');
$mail->Subject = 'An HTML Message';
$mail->isHTML(true);
$mail->Body = 'Hello, <b>my friend</b>! This message uses HTML!';
```
    
It's as easy as creating a plain text e-mail. Simply call `$mail->isHTML(true);` and use an HTML string for `$mail->Body`.

To make ensure that the recipient will be able to read the e-mail, even if their e-mail client doesn't support HTML, we can add a plain-text version of the message:

```php
$mail->AltBody = "Hello, my friend! This message uses plain text !";
```

This sets the alternative body (`AltBody` for short). If you use this feature, the message will automatically use the MIME type `multipart/alternative`, which builds the message in a way that MIME-compliant clients can use to pick the format the recipient prefers.

Be warned that HTML support varies enormously, but generally you can only use pure HTML - That means no scripts, no Flash, etc. What you can use is documented by the [email standards project](http://www.email-standards.org).

# Support

PHPMailer is open source and published under the [LGPL 2.1 license](http://opensource.org/licenses/lgpl-2.1.php). This tutorial covers only the basics - much more is covered by other examples and documentation.

* PHPMailer [project home](https://github.com/PHPMailer/PHPMailer): https://github.com/PHPMailer/PHPMailer
* PHPMailer [documentation start](https://github.com/PHPMailer/PHPMailer/wiki): https://github.com/PHPMailer/PHPMailer/wiki
* PHPMailer [code examples](https://github.com/PHPMailer/PHPMailer/tree/master/examples): https://github.com/PHPMailer/PHPMailer/tree/master/examples
* PHPMailer [troubleshooting guide](https://github.com/PHPMailer/PHPMailer/wiki/Troubleshooting): https://github.com/PHPMailer/PHPMailer/wiki
* PHPMailer [generated API documentation](http://phpmailer.github.io/PHPMailer/): http://phpmailer.github.io/PHPMailer/
* PHPMailer [bug reports](https://github.com/PHPMailer/PHPMailer/issues) (not "how do I..." questions!): https://github.com/PHPMailer/PHPMailer/issues
 
There's an enormous amount of other information available about e-mail, PHP and PHPMailer:

* Questions are answered on [Stackoverflow](http://stackoverflow.com/questions/tagged/phpmailer) - tag your questions with `phpmailer` and the maintainers will see them.
* PHP [`mail()` function reference](http://www.php.net/mail)
* The [email standards project](http://www.email-standards.org) (about client support for HTML messages)
* Many email standards are applicable to PHPMailer, including:
  * Email [RFC5322](http://www.ietf.org/html/rfc5322)
  * SMTP [RFC5321](https://tools.ietf.org/html/rfc5321)
  * MIME RFCs [2045](http://www.ietf.org/html/rfc2045), [2046](http://www.ietf.org/html/rfc2046), [2047](http://www.ietf.org/html/rfc2047)