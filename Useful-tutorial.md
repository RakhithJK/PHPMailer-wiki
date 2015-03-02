# Introduction

This tutorial describes how to use PHPMailer and explains the use of the class functions. by Tom Klingenberg

# Contents

PHPMailer: What is it? What does it do? Who needs it? Brief examples of PHPMailer's features.
First_time: Sending your first email with PHPMailer. Tells you how to start. Anyone who completely understands the source on the PHPMailer homepage or README can skip this section.
Using_Attachments: Sending mails with file attachments: filesystem, database or inline.
Using_HTML_Mail: Using the class' integrated HTML Email features.
Support: Where do I get help?: How to find support resources: PHPMailer website, mailing list etc.
Other_Resources: Email and PHP Resources: There is a lot of information in the net about email, php and PHPMailer.

# PHPMailer

PHPMailer is a PHP class for PHP (www.php.net) that provides a package of functions to send email. The two primary features are sending HTML Email and e-mails with attachments. PHPMailer supports nearly all possiblities to send email: mail(), Sendmail, qmail & direct to SMTP server. You can use any feature of SMTP-based e-mail, multiple recepients via to, CC, BCC, etc. In short: PHPMailer is an efficient way to send e-mail within PHP.

As you may know, it is simply to send mails with the PHP mail() function. So why use PHPMailer? Isn't it slower? Yes that's true, but PHPMailer makes it easy to send e-mail, makes it possible to attach files, send HTML e-mail, etc. With PHPMailer you can even use your own SMTP server and avoid Sendmail routines used by the mail() function on Unix platforms.

This tutorial explains how to implement the class into your script or website and how to build an e-mail application. If you just want to send simple e-mail and you have no problems with the mail() function, it's suggested that you continue to use mail(), instead of this class.

# First time

For those who have not used PHP's mail() function before, this section will provide information about email and e-mailing in general, in addition to explaining how to get started with PHPMailer.

Before continuing, please be sure that PHPMailer is installed correctly. If you feel uncertain, please read the installion instructions that accompany the package. If you're still not sure, you can verify that you've installed PHPMailer correctly with this little script:

````php
<?php
require("class.phpmailer.php");
$mail = new PHPMailer();
?>
````

Save it as a PHP document in the same directory where you've saved class.phpmailer.php. If no errors result from running the script, your installation has been done correctly. If you switched error messages and warnings off, then go ahead and set it on; refer to your PHP manual for more on this topic.

This snippet of code is also the start of any use of PHPMailer. require("class.phpmailer.php"); includes the source code of the class and $mail = new PHPMailer(); creates a new instance of the class the class as $mail. All features, functions and methods of the class maybe accessed via the variable (object) $mail.

Merely creating an instance of the class is only the first step, of course. Let's go ahead and send out the first mail. For this you'll require the basics: An recipient address, a from address, a subject and the text of the message (called "mail-body" or simply "body" for short).

You will also need to select a method of delivering the message. We need a program to communicate with an SMTP server which, in turn, sends the mail out to the internet. In the Unix world, Sendmail is very popular and used widely.

PHPMailer provides several methods of sending mail. It's best to start with SMTP, though (as mentioned above) the mail() function, Sendmail or qmail are also possible. When using SMTP, you'll need a valid SMTP server; this may well be the same one that you use in your personal mail client.

Having gathered the necessary information to send your first e-mail, you'd use that information in place of the example information provided here:

````php
<?php

require("class.phpmailer.php");

$mail = new PHPMailer();

$mail->IsSMTP();  // telling the class to use SMTP
$mail->Host     = "smtp.example.com"; // SMTP server

$mail->From     = "from@example.com";
$mail->AddAddress("myfriend@example.net");

$mail->Subject  = "First PHPMailer Message";
$mail->Body     = "Hi! \n\n This is my first e-mail sent through PHPMailer.";
$mail->WordWrap = 50;

if(!$mail->Send()) {
  echo 'Message was not sent.';
  echo 'Mailer error: ' . $mail->ErrorInfo;
} else {
  echo 'Message has been sent.';
}
?>
````

Save this as a php file and change the values to your values, of course. Here is an explanation of what each stanza in this script does:

````php
$mail->IsSMTP();
````

This sets up STMP-Server as method to send out email(s).

````php
$mail->Host = "smtp.example.com";
````

Setting smtp.example.com as the SMTP server. Just replace it with your own SMTP server address. You can even specify more then one: just separate them with a semicolon (;): "smtp.example.com;smtp2.example.com". If the first one fails, the second one will be used, instead.

Enter the address that the e-mail should appear to come from. You can use any address that the SMTP server will accept as valid.

````php
$mail->From = "from@example.com";
````

If displaying only an e-mail address in the "From" field is too simple, you can add another line of code to give the address an associated name. This will add 'Your Name' to the from address, so that the recipient will know the name of the person who sent the e-mail. The following line

````php
$mail->FromName = "Your Name";
````

Andy Prevost: with PHPMailer version 5.0.0, you can now achieve this with one command:

````php
$mail->SetFrom("from@example.com","Your Name");
````

The following will add the to address, the address to which the e-mail will be sent. You must use a valid e-mail here, of course, if only so that you can verify that your PHPMailer test worked. It's best to use your own e-mail address here for this inintial test. As with the "From" field, you may provide a name for the recipient. This is done somewhat differently:

````php
$mail->AddAddress("myfriend@example.net","Friend's name");
$mail->AddAddress("myfriend@example.net");
````

Setting the subject and body is done next. $mail->WordWrap = 50; is a feature of PHPMailer to word-wrap your message limiting all lines to a maximum length of X characters, even if not set as body= .... Admittedly, there's not much point in wrapping the text in a message as brief as the one in this example.

Finally, we send out the e-mail, once all necessary information has been provided. This is done with $return = $mail->Send();. In this example script, it's combined with an error message; if Send() fails, it'll return false and you can catch it and display an error message. This is done in the last lines. Or, in case of success, it displays a kind of "done " message.

# Using Attachments

Sending plain text e-mails is often insufficient. Perhaps you need to attach something to your mail, such as an image or an audio file. Or perhaps you need to attach multiple files.

There are two ways of attaching something to your mail: You can simply attach a file from the filesystem or you can attach (binary) data stored in a variable. The latter is called Stringattachment. This makes it possible put extract data from a database and attach it to an e-mail, without ever having to actually save it as a file.

## File Attachments

The command to attach a file can be placed anywhere between $mail = new PHPMailer(); and !$mail->Send(); and it's called AddAttachment($path);. This single line will add the attachment to your mail.

$path is the path of the filename. It can be a relative one (from your script, not the PHPMailer class) or a full path to the file you want to attach.

If you want more options or you want to specify encoding and the MIME type of the file, then you can use three more parameters, all of which are optional:

AddAttachment($path,$name,$encoding,$type);
$name is an optional parameter, used to set the name of the file that will be embedded within the e-mail. The person who will recieve your mail will then only see this name, rather than the original filename.

$encoding is a little more technical, but with this parameter you can set the type of encoding of the attachment. The default is base64. Other types that are supported include: 7bit, 8bit, binary & quoted-printable. Please refer to your SMTP documentation about encoding and the differences between types. In general, mail servers will convert encodings they don't want to handle into their preferred encoding type.

$type is the MIME type of the attached file. Content types are defined not necessarily by file suffixes (i.e., .GIF or .MP3), but, rather, a MIME type (MIME = Multipurpose Internet Mail Extensions) is used. This parameter makes it possible change the MIME type of an attachment from the default value of application/octet-stream (which works with every kind of file) to a more specific MIME type, such as image/jpeg for a .JPG photo, for instance.

## String Attachments

String attachments have been supported since PHPMailer version 1.29. This method works much like AddAttachment(), and is called with AddStringAttachment($string,$filename,$encoding,$type). The string data is passed to the method with the first parameter, $string. Because the string will become a standard file (which is what the attachment will be when received via e-mail), the $filename parameter is required. It's used to provide that filename for the string data.

The rest is just the same as described in detail above.

So, why use AddStringAttachment() instead of AddAttachment()? Is it for text-only files? No, not at all. It's primarily for databases. Data stored in a database is always stored as a string (or perhaps, in the case of binary data, as as a BLOB: Binary Large OBject). You could query your database for an image stored as a BLOG and pass the resulting string to the AddStringAttachment().

## Inline Attachments

There is an additional way to add an attachment. If you want to make a HTML e-mail with images incorporated into the desk, it's necessary to attach the image and then link the  tag to it. For example, if you add an image as inline attachment with the CID my-photo, you would access it within the HTML e-mail with `&ltimg src="cid:my-photo" alt="my-photo" />.

In detail, here is the function to add an inline attachment:

````php
$mail->AddEmbeddedImage(filename, cid, name);
````

By using this function with this example's value above, results in this code:

````php
$mail->AddEmbeddedImage('my-photo.jpg', 'my-photo', 'my-photo.jpg ');
````

For more Information about HTML Email, see the section Using HTML E-Mail.

## Handling Attachments

If you want to attach multiple files (or strings), just call AddAttachment() or AddStringAttachment() multiple times. All attachments (file, string, and inline) may be stripped from an e-mail by invoking ClearAttachments().

# Using HTML Email

Sending out HTML e-mail is a simple enough task with PHPMailer, though it can require significant knowledge of HTML. In particular, mail clients vary greatly in their rendering of HTML e-mail, with some refusing to show it entirely. For those mail clients which are not able to display HTML, you can provide an alternate e-mail body containing the message as plain text.

First we'll create a basic HTML message:

````php
<?php

require("class.phpmailer.php");

$mail = new PHPMailer();

$mail->IsSMTP();  // telling the class to use SMTP
$mail->Host     = "smtp.example.com"; // SMTP server

$mail->From     = "from@example.com";
$mail->AddAddress("myfriend@example.net");

$mail->Subject  = "An HTML Message";
$mail->Body     = "Hello, <b>my friend</b>! \n\n This message uses HTML entities!";

?>
````
    
It's as easy as creating a plain text e-mail. Simply use a string with embedded HTML for $mail->body.

Before sending this out, we have to modify the PHPMailer object to indicate that the message should be interpreted as HTML. Technically-speaking, the message body has to be set up as multipart/alternative. This is done with:

````php
$mail->IsHTML(true);
````

To make ensure that the recipient will be able to read the e-mail, even if his e-mail client doesn't support HTML, use $mail->AltBody="Hello, my friend! \n\n" This message uses HTML entities, but you prefer plain text !"; to set the alternative body (AltBody in short). If you use this feature, the mail will be automatically set as multipart/alternative, making it unnecessary to specify $mail->IsHTML(true);.

And that's how to create an HTML email. Simply apply the send command and the mail will be sent. It's recommended that you avoid using HTML messages, because these messages are often used for viruses and exploits, they are much larger than plain text mails, and they're not standarized in the way the plain text mail is. On the other hand, even attachments are used for viruses, but nobody would propose that we stop sending mails with attachments. Do what fits your needs: PHPMailer can be your solution, whatever you decide.

If you want to send out a HTML email with pictures, Flash animations or whatever else, PHPMailer supports this as well.


> Andy Prevost: Please note that it is not possible to send anything but pure W3C compliant HTML inline with your emails. That means no scripts, no Flash, etc. ... these can be included as attachments, but not as inline viewable objects – other than images (JPG, JPEG, PNG, & GIF). While some email clients may be able to display some objects such as other image types, the vast majority of email clients will either block them, crash, or remove them entirely as inline objects.
> Adding an inline picture to your HTML e-mail, for example, is explained in the Inline Attachments section. Briefly, here are 2 lines of code you'd use to insert an image before sending out an e-mail:

````php
$mail->AddEmbeddedImage("rocks.png", "my-attach", "rocks.png");
$mail->Body = 'Embedded Image: <img alt="PHPMailer" src="cid:my-attach"> Here is an image!';
````

# Support

PHPMailer is open source and published under the GPL. Most of the questions how to start are answered within this tutorial, but inevitably some questions may well remain. For this, PHPMailer has its own homepage on SourceForge and a mailing list where you can post your questions.

 * PHPMailer Homepage: http://phpmailer.sourceforge.net/
 * PHPMailer Mailing list: http://lists.sourceforge.net/lists/listinfo/phpmailer-general
 * PHPMailer Documentation: http://phpmailer.sourceforge.net/phpdoc/phpmailer.html
 * PHPMailer Tutorials by Community MX: A series of third-party tutorials by Community MX.
 * Other Resources to E-mail and PHP

There's an enormous amount information avaiable about e-mail, PHP and PHPMailer:

 * PHPMailer homepage http://phpmailer.sourceforge.net/
 * php mail() function reference http://www.php.net/manual/en/ref.mail.php or with the new shortcut as http://www.php.net/mail
 * Standard E-mail RFC 822 http://www.w3.org/Protocols/rfc822/
 * MIME E-mail RFC 2046 http://www.ietf.org/rfc/rfc2046.txt
