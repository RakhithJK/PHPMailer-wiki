<?php
	require_once('../config/config.php');
	require_once('PHPMailerAutoload.php');
?>
<?php
$mail = new PHPMailer;
$to = $_SESSION['email'];
$activation_code = $_SESSION['activation_code'];
$password = $_SESSION['password'];
//$mail->SMTPDebug = 3;                               // Enable verbose debug output

$mail->isSMTP();                                      // Set mailer to use SMTP
$mail->Host = 'smtp.gmail.com';  // Specify main and backup SMTP servers
$mail->SMTPAuth = true;                               // Enable SMTP authentication
$mail->Username = 'qasim6722946@gmail.com';                 // SMTP username
$mail->Password = 'abcd7806527';                           // SMTP password
$mail->SMTPSecure = 'tls';                            // Enable TLS encryption, `ssl` also accepted
$mail->Port = 587;                                    // TCP port to connect to
$to=$_SESSION['email'];
$mail->setFrom('qasim6722946@gmail.com', 'Mailer');
$mail->addAddress($to);     // Add a recipient

$mail->isHTML(true);                                  // Set email format to HTML

$mail->Subject = 'Account Confirmation Message';
$mail->Body = "
 
Thanks for signing up!
Your account has been created, you can login with the following credentials after you have activated your account by pressing the url below.
 
------------------------<br><br><br><br>
Emali:" .$_SESSION['email']."<br>
Password:" .$_SESSION['password']."<br><br><br><br>
------------------------
 
Please click this link to activate your account:----------------------<br><br><br><br>
http://localhost/Doctor.Pk/verify.php?email=".$_SESSION['email']."&activation_code=".$_SESSION['activation_code']."  "; // Our message above including the link

$mail->send();

if(!$mail->send()) {
    echo 'Message could not be sent.';
    echo 'Mailer Error: ' . $mail->ErrorInfo;
} 
?>


