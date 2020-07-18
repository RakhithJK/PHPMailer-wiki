SERVER -> CLIENT: 220 smtp.gmail.com ESMTP h23sm20114034wmb.3 - gsmtp

CLIENT -> SERVER: EHLO sunsagevnsconsulting.rf.gd

SERVER -> CLIENT: 250-smtp.gmail.com at your service, [185.27.134.47]250-SIZE 35882577250-8BITMIME250-STARTTLS250-ENHANCEDSTATUSCODES250-PIPELINING250-CHUNKING250 SMTPUTF8

CLIENT -> SERVER: STARTTLS

SERVER -> CLIENT: 220 2.0.0 Ready to start TLS

CLIENT -> SERVER: EHLO sunsagevnsconsulting.rf.gd

SERVER -> CLIENT: 250-smtp.gmail.com at your service, [185.27.134.47]250-SIZE 35882577250-8BITMIME250-AUTH LOGIN PLAIN XOAUTH2 PLAIN-CLIENTTOKEN OAUTHBEARER XOAUTH250-ENHANCEDSTATUSCODES250-PIPELINING250-CHUNKING250 SMTPUTF8

CLIENT -> SERVER: AUTH LOGIN

SERVER -> CLIENT: 334 VXNlcm5hbWU6

CLIENT -> SERVER: dGlsdmF2YXN1MTExQGdtYWlsLmNvbQ==

SERVER -> CLIENT: 334 UGFzc3dvcmQ6

CLIENT -> SERVER: VmFzdTEyMzQ1Njc4OQ==

SERVER -> CLIENT: 535-5.7.8 Username and Password not accepted. Learn more at535 5.7.8  https://support.google.com/mail/?p=BadCredentials h23sm20114034wmb.3 - gsmtp

SMTP ERROR: Password command failed: 535-5.7.8 Username and Password not accepted. Learn more at535 5.7.8  https://support.google.com/mail/?p=BadCredentials h23sm20114034wmb.3 - gsmtp

SMTP Error: Could not authenticate.

CLIENT -> SERVER: QUIT

SERVER -> CLIENT: 221 2.0.0 closing connection h23sm20114034wmb.3 - gsmtp

SMTP connect() failed. https://github.com/PHPMailer/PHPMailer/wiki/Troubleshooting

Message could not be sent.Mailer Error: SMTP connect() failed. https://github.com/PHPMailer/PHPMailer/wiki/Troubleshooting