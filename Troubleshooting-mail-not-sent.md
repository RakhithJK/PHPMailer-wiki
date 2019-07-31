<?php

$mailto = "sman80121@gmail.com";
$mailSub ="hellso";
$mailMsg = "worlds";

require 'PHPMailer/PHPMailerAutoload.php';

$mail = new PHPMailer();
$mail ->isSMTP();
$mail -> SMTPDebug = 2;
$mail ->SMTPAuth = true;
$mail ->SMTPSecure = 'ssl';
$mail ->Host = "smtp.gmail.com";
$mail -> Port =465; // 587
$mail ->isHTML(true);
$mail -> Username = "test@gmail.com";
$mail ->Password = "testpwd";
$mail ->setFrom("test@gmail.com");
$mail -> Subject = $mailSub;
$mail ->Body = $mailSub;
$mail ->addAddress($mailto);

if(!$mail ->Send())
{
    echo "mail not sent";
}
 else 
 {
     echo "mail sent";
 }
    
?>

