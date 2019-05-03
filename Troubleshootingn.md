<?php
require 'vendor/autoload.php';
use PHPMailer\PHPMailer\PHPMailer;

$mail = new PHPMailer();
$mail->isSMTP();  // Set mailer to use SMTP
$mail->Host = 'smtp.mailgun.org';  // Specify mailgun SMTP servers
$mail->SMTPAuth = true; // Enable SMTP authentication
$mail->Username = 'postmaster@robustitconcepts.com'; // SMTP username from https://mailgun.com/cp/domains
$mail->Password = '5dcd5d37e759c3c8915e71779f5add0f'; // SMTP password from https://mailgun.com/cp/domains
$mail->SMTPSecure = 'tls';   // Enable encryption, 'ssl'
$mail->From = 'info@robustitconcepts.com'; // The FROM field, the address sending the email
$mail->FromName = 'Robust IT Concepts Pvt. Ltd.'; // The NAME field which will be displayed on arrival by the email client
$mail->addAddress('shashi.gharti@gmail.com', 'Shashi Gharti');     // Recipient's email address and optionally a name to identify him
$mail->isHTML(true);
$mail->Subject = 'Email sent with Mailgun';
$mail->Body    = '<span style="color: red">Mailgun rocks</span>, thank you <em>phpmailer</em> for making emailing easy using this <b>tool!</b>';
$mail->AltBody = '';
echo 'sending';
if(!$mail->send()) {
    echo "Message hasn't been sent.";
    echo 'Mailer Error: ' . $mail->ErrorInfo . "n";
} else {
    echo "Message has been sent";
}
?>
