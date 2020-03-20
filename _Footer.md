include_once('PHPMailer/Exception.php');
include_once('PHPMailer/PHPMailer.php');
include_once('PHPMailer/SMTP.php');
$mail = new PHPMailer\PHPMailer\PHPMailer();
$mail = new PHPMailer\PHPMailer\PHPMailer();

$mail->SMTPDebug = 2;                                 // Enable verbose debug output
$mail->isSMTP();    
$mail->SMTPAuth = true;
$mail->Host = 'mail.nediapp.com';
$mail->SMTPSecure = 'tls';
$mail->Port = 587;
$mail->Username = 'administrador@nediapp.com'; //Correo de donde enviaremos los correos
$mail->Password = '74074222$leoc'; // Password de la cuenta de envío

$mail->setFrom('administrador@nediapp.com', 'Administrador');
$mail->addAddress('eliseo.ortegacoronado@gmail.com', 'Receptor'); //Correo receptor
$mail->Subject = 'Aviso Importante de Iso 27001';
//$mail->msgHTML(file_get_contents('contenido.html'), dirname(__FILE__)); 
$mail->CharSet = 'UTF-8';


$body  = "<html lang='es'><head><meta charset='UTF-8'></head><body>
<div><center><span style='color: #ff000c;font-size: 30px;'><strong>No Conformidad</strong></span></center></div><br>";
$body .= "<div><span style='color: #000;'><strong>Curso: <span>hola</span></span></strong></div><br>";
$body .= "<div><span style='color: #000;'><strong>Fecha Final: <span>aaaa</span></span></strong></div><br>";
$body .= "<div><span style='color: #000;'><strong>Estado: <span style='color: #ff000c;'>aaaaaaa</span></span></div></strong><br></body></html>";
$mail->Body = $body;
$exto=$mail->send();
if($exto){
            echo 'Correo Enviado';
            }else{
            echo 'Error al enviar correo';
            }
