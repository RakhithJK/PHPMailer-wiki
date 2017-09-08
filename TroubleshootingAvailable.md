Hi, people. Sorry, but i don't know why nearly this code works for me, then 2 days ago, without changes breakedown

unction enviarEmail($email, $nombre, $asunto, $cuerpo){
		echo "$email";
		require 'PHPMailer/PHPMailerAutoload.php';

		$mail = new PHPMailer();
		$mail->isSMTP();
		$mail->SMTPAuth = true;
		$mail->SMTPSecure = 'ssl';
		$mail->Host = 'mail.example.site';
		$mail->Port = 465;
		$mail->Username = 'info@example.site';
		$mail->Password = 'password';
		$mail->setFrom('info@example.site', 'Sistema de Usuarios');
		$mail->addAddress($email, $nombre);
		$mail->Subject = $asunto;
		$mail->Body    = $cuerpo;
		$mail->IsHTML(true);
		if(!$mail->Send()) {
                echo 'Message could not be sent.';
        echo 'Mailer Error: ' . $mail->ErrorInfo;
        exit;
        }
        echo 'Message has been sent';
	    }