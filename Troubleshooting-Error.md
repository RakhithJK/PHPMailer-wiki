#  SMTP ERROR: Failed to connect to server: php_network_getaddresses: getaddrinfo failed: No such host is known
This is the full error_message shows in browser  : 
"2017-06-08 04:31:11	Connection: opening to ssl://mail.smtp2go.com:2525, timeout=300, options=array ( ) 2017-06-08 04:31:11	Connection failed. Error #2: stream_socket_client(): php_network_getaddresses: getaddrinfo failed: No such host is known. [C:\xampp\htdocs\photo_gallery\includes\PHPMailer\class.smtp.php line 292] 2017-06-08 04:31:11	Connection failed. Error #2: stream_socket_client(): unable to connect to ssl://mail.smtp2go.com:2525 (php_network_getaddresses: getaddrinfo failed: No such host is known. ) [C:\xampp\htdocs\photo_gallery\includes\PHPMailer\class.smtp.php line 292] 2017-06-08 04:31:11	SMTP ERROR: Failed to connect to server: php_network_getaddresses: getaddrinfo failed: No such host is known. (0) 2017-06-08 04:31:11	SMTP connect() failed. https://github.com/PHPMailer/PHPMailer/wiki/Troubleshooting Error"
## code : 
<?php
	error_reporting(E_ALL);
	require_once("../photo_gallery/includes/PHPMailer/class.phpmailer.php");
	require_once("../photo_gallery/includes/PHPMailer/PHPMailerAutoload.php");
	require_once("../photo_gallery/includes/PHPMailer/class.smtp.php");
	require_once("../photo_gallery/includes/PHPMailer/language/phpmailer.lang-en.php");
	date_default_timezone_set('Asia/Kolkata');
	setlocale(LC_TIME, "C");

	$to_name = "Junk mail";
	$to = "s.singh.hbn@gmail.com";

	$subject = "Mail Test at ".strftime("%T", time());

	$message = "This is a test.";
	$message = wordwrap($message,70);

	$from_name = "Abhishek Singh";
	$from = "s.singh.hbn@gmail.com";

	//using SMTP 
	$mail = new PHPMailer();
	$mail->IsSMTP();
	$mail->Host      = "mail.smtp2go.com";
        //$mail->Host      = "mail.gmail.com";
	$mail->SMTPDebug  = 4;
	$mail->Port      = 2525;
	$mail->SMTPAuth  = true;
	$mail->SMTPSecure = "ssl";   
	$mail->Username  = "mypersonal_email@gmail.com";
	$mail->Password  = "mypassword";
	$mail->SetFrom('s.singh.hbn@gmail.com', 'PRSPS');
	$mail->MsgHTML($message);
	$mail->isHTML(true); 

	$mail->From_name = $from_name;
	$mail->From 	 = $from;
	$mail->AddAddress($to,$to_name);
	$mail->Subject   = $subject;
	$mail->Body      = $message;
	
	$result = $mail->send();
	echo $result ? 'Sent' : 'Error';
  
?>