Message could not be sent. Mailer Error: SMTP connect() failed.
Please how do i solve this issue... my login credentials are correct but i still cant use it to send mail.


<?php
	/**
	 * this handles mail sending 
	 */
	// Import PHPMailer classes into the global namespace
	// These must be at the top of your script, not inside a function
	use PHPMailer\PHPMailer\PHPMailer;
	use PHPMailer\PHPMailer\Exception;

	// Load Composer's autoloader
	require 'vendor/autoload.php';

	// Instantiation and passing `true` enables exceptions
	$mail = new PHPMailer(false);

	class EmailClass 
	{
		
		public function mail(array $data = []){
			$mail = new PHPMailer(true);

			$name = $data['name'];
			$email = $data['email'];
			$message = $data['message'];
			try {
			    //Server settings
			    $mail->SMTPDebug = 0; 
			    // 0 = off (for production use, No debug messages)
				// 1 = client messages
				// 2 = client and server messages 
			    // Enable verbose debug output
			    $mail->isSMTP();                                            // Set mailer to use SMTP
			    $mail->Host       = 'smtp.gmail.com';  // Specify main and backup SMTP servers
			    $mail->SMTPAuth   = true;                                   // Enable SMTP authentication
			    $mail->Username   = 'harvardkidneyservices@gmail.com';               // SMTP username
			    $mail->Password   = '....mywrongpassword....'; 
			    // SMTP password
			    $mail->SMTPSecure = 'ssl'; 
			    // or ssl    or tls                             
			    // Enable TLS encryption, `ssl` also accepted
			    $mail->Port       = 465;                                    // TCP port to connect to
			    // port 587 (tls) or 465 (ssl)

			    //Recipients
			    $mail->setFrom('noreply@havardkidneyservices.com', 'Contact');
			    $mail->addAddress($email, $name);     // Add a 
			    // Content
			    $mail->isHTML(true);                                  // Set email format to HTML
			    $mail->Subject = 'Contact Havard Kidney Services';
			    $mail->Body    = "<h3>
							Mr/Mrs/Master/Miss ". $name. " with this email address =>". $email ." sent a message saying - ".
							$message ."
						</h3>";
			    // $mail->AltBody = 'This is the body in plain text for non-HTML mail clients';

			    $mail->send();
			    return true;
			    // echo 'Message has been sent';
			} catch (Exception $e) {
			    echo "Message could not be sent. Mailer Error: {$mail->ErrorInfo}";
			}
		}
	}

?>