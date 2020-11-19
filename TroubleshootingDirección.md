require_once('mail/PHPMailerAutoload.php');
$mail = new PHPMailer;
$mail->From = "damazonhosting@gmail.com";
$mail->FromName = "Jesus Cloak";
echo (extension_loaded('openssl')?'SSL loaded':'SSL not loaded')."\n"; 

$mail->isSMTP();                                      // Set mailer to use SMTP
$mail->Host = 'tls://smtp.gmail.com:587';
  // Specify main and backup SMTP servers
$mail->SMTPAuth = true;                    
$mail->SMTPDebug = 2;
           // Enable SMTP authentication
$mail->Username = 'damazonhosting@gmail.com';                 // SMTP username
$mail->Password = 'jesusdaniel25(hidden atm)';                           // SMTP password
// $mail->SMTPSecure = 'tls';                            // Enable TLS encryption, `ssl` also accepted
// $mail->Port = 587;               

if(isset($_POST['send'])){
	$to = $_POST['to'];
	$subject = $_POST['subject'];
	$text= $_POST['text'];
	$mail->addAddress($to); //Recipient name is optional

$mail->From = "damazohosting@gmail.com";
$mail->FromName = "Jesus Cloak";
$mail->addAddress($to); //Recipient name is optional
$mail->isHTML(true);
$mail->Subject = $subject;
$mail->Body =  $text;
// $mail->AltBody = "This is the plain text version of the email content";
if(!$mail->send()) 
{
	$message['type'] = 'error';
    $message['msg'] =  "Mailer Error: " . $mail->ErrorInfo;
} 
else 
{$message['type'] = 'success';
    $message['msg'] =  "Message has been sent successfully";
}

	
} 