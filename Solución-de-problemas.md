include_once('PHPMailer/Exception.php');
include_once('PHPMailer/PHPMailer.php');
include_once('PHPMailer/SMTP.php');
$mail = new PHPMailer\PHPMailer\PHPMailer();
$mail = new PHPMailer\PHPMailer\PHPMailer();

$mail->SMTPDebug = 2;                                 // Enable verbose debug output
$mail->isSMTP();    
$mail->SMTPAuth = true;
$mail->Host = 'smtpout.secureserver.net';
$mail->SMTPSecure = 'tls';
$mail->Port = 587;
$mail->Username = 'administrador@nediapp.com'; //Correo de donde enviaremos los correos
$mail->Password = '74074222$'; // 74074222$l

$mail->setFrom('administrador@nediapp.com', 'Administrador');
$mail->addAddress('eliseo.ortegacoronado@gmail.com', 'Receptor'); //Correo receptor
$mail->Subject = 'Aviso Importante de Iso 27001';
//$mail->msgHTML(file_get_contents('contenido.html'), dirname(__FILE__)); 
$mail->CharSet = 'UTF-8';

$body  = "es una prueba de correo";
$body .= "Curso: ";
$body .= "Fecha Final: ";
$body .= "Estado: ssss";
$mail->Body = $body;
$exto=$mail->send();
if($exto){
            echo 'Correo Enviado';
            }else{
            echo 'Error al enviar correo';
            }