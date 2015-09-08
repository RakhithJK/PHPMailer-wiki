The official PHPMailer wiki provides an example of using Google's XOAuth2 SMTP authentication using the [League OAuth2 client library](https://github.com/thephpleague/oauth2-client). This library works fine, but has one drawback to it.

For small projects, using league/oauth2-client library to make api call to Google will do just fine but it comes with many dependencies; ircmaxell/random-lib, guzzlehttp/guzzle, and a few others. On top of this, you must install the [league oauth2 google provider library](https://github.com/thephpleague/oauth2-google).

But you can still reduce on the number of third party libraries you're using in your project by using the official [Google API client](https://github.com/google/google-api-php-client) with [Curl](http://curl.haxx.se/). (Ofcourse if you're using the league OAuth2 library generally in you project for example for authentication, then this would not do you so much good - so stick with that)

Firstly, lets install PHPMailer using composer, remember Google's XOAUTH2 SMTP & IMAP authentication mechanism is only supported starting with PHPMailer 5.2.11. So you must install that or later;<br/>
`composer require phpmailer/phpmailer ~5.2`

So hope you have curl install on your working machine, and enabled for php? If not, you may refer to this great discussion; ["How do I install cURL on Windows?"](http://stackoverflow.com/questions/1347146/how-to-enable-curl-in-php-xampp) on stackoverflow.

Once you have Curl installed and configured, use composer to install the Google API client library by <br/>
`composer require google/apiclient 1.*`

When you register you client app, download it details in json and rename the file to `gmail-xoauth2-credentials.json` or any name you prefer, create another file and name it `gmail-xoauth2-token.json` this will hold the access token after user authorization.

define ( 'APPLICATION_NAME', 'YOUR_APPLICATION NAME' );
define ( 'APP_CREDENTIALS', 'path_to/gmail-xoauth2-credentials.json' );
define ( 'CREDENTIALS_PATH', 'path_/gmail-xoauth2-token.json');
define ( 'SCOPES', implode ( ' ', array (
		\Google_Service_Gmail::GMAIL_COMPOSE 
) ) );

class GmailXOAuth2 {

    private $oauthUserEmail = '';
    private $oauthRefreshToken = '';
    private $oauthClientId = '';
    private $oauthClientSecret = '';
    
    public function __construct(
        $UserEmail,
        $ClientSecret,
        $ClientId,
        $RefreshToken
    ) {
        $this->oauthClientId = $ClientId;
        $this->oauthClientSecret = $ClientSecret;
        $this->oauthRefreshToken = $RefreshToken;
        $this->oauthUserEmail = $UserEmail;
    }
	
        /*
         * @returns $google_client object
         */
	private function getClient() {
	
		$google_client = new \Google_Client ();
	
		$google_client->setApplicationName ( APPLICATION_NAME );
		$google_client->setScopes ( 'https://mail.google.com/' );
		$google_client->setAuthConfigFile ( APP_CREDENTIALS );
                /* Its a must to request for 'offile access type' */
		$google_client->setAccessType ( 'offline' );
	
		return $google_client;
	
	}
	
        /*
         * checks the credentials for the access token, if present; it returns that
         * or refreshes it if expired. 
         * if the credentials file is empty, it will return the authorization url to which you must redirect too 
         * for user user authorization 
         */
	public static function authenticate () {
	
		$client = GmailXOAuth2::getClient();
	
		if (!empty(file_get_contents(CREDENTIALS_PATH))) {
			
			$accessToken = file_get_contents(CREDENTIALS_PATH);
		
		} else {
			
			return array( 'authorization_uri' => $client->createAuthUrl() );
			
		}
		
		$client->setAccessToken($accessToken);
		
		// Refresh the token if it's expired.
		if ($client->isAccessTokenExpired()) {
			
			$client->refreshToken($client->getRefreshToken());
			
			$new_accessToken = $client->getAccessToken();
			
			if (file_put_contents(CREDENTIALS_PATH, $new_accessToken)) {
				
				return json_decode($new_accessToken, true);
				
			}
			
		}
		
		return json_decode($accessToken, true);
	
	}
	
        /*
         * call this in your callback (redirect url), code the authorization for and exchanges it for an 
         * access token. 
         * it stores this in the token file for future reference.
         * if the user denies your app access, it will still return just that error and not write to the token file
         */
	public static function resetCredentials( $authCode ) {
		
		$client = GmailXOAuth2::getClient();
		
		$accessToken = $client->authenticate( $authCode );
		
		if( file_put_contents( CREDENTIALS_PATH, $accessToken ) ) {
			
			return json_decode( $accessToken, true );
			
		}
		
		return false;
		
	}
	
	/**
	 * GetOauth64
	 * 
	 * encode the user email related to this request along with the token in base64
	 * this is used for authentication, in the phpmailer smtp class
	 * 
	 * @return string
	 */
	public function getOauth64 () {
		
		$client = GmailXOAuth2::getClient();
		
		if (!empty(file_get_contents(CREDENTIALS_PATH))) {
				
			$accessToken = file_get_contents(CREDENTIALS_PATH);
		
		} else {
				
			return false;
				
		}
		
		$client->setAccessToken($accessToken);
		
		// Refresh the token if it's expired.
		if ($client->isAccessTokenExpired()) {
				
			$client->refreshToken($client->getRefreshToken());
				
			$accessToken = $client->getAccessToken();
				
			file_put_contents(CREDENTIALS_PATH, $accessToken);
				
		}

		$offlineToken = GmailXOAuth2::request_offline_token();
		
		return base64_encode("user=" . $this->oauthUserEmail . "\001auth=Bearer " . $offlineToken . "\001\001");
	
	}
	
        /*
         * this makes a request to the Google API, using Curl to get another access token that we can use 
         * for authentication on the Gmail API when sending messages
         */
	private function request_offline_token() {
		 
		$token_uri = "https://accounts.google.com/o/oauth2/token";
		$parameters = array(
				"grant_type" => 'refresh_token',
				"client_id" => $this->oauthClientId,
				"client_secret" => $this->oauthClientSecret,
				"refresh_token" => $this->oauthRefreshToken
		);
		 
		$curl = curl_init($token_uri);
	
		curl_setopt($curl, CURLOPT_POST, true);
		curl_setopt($curl, CURLOPT_POSTFIELDS, $parameters);
		curl_setopt($curl, CURLOPT_HTTPAUTH, CURLAUTH_ANY);
		curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
		curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
	
		$response = curl_exec($curl);
		curl_close($curl);
	
		$response = json_decode($response, true);
		
		return $response['access_token'];
	}
	
}

<h3>Why must we request for another access token?</h3>
This is the most crucial/tricky part I found in the Google XOAuth2 mechanism. Remember, we requested for offline access but we received and access token from that.
However we can't use that very access token to make interaction with the Gmail API in offline access type so we must request for another access token using the <b>refresh token</b> we received.
This is done in `GmailXOAuth2::request_offline_token()` using the refresh grant type.

<h3>I didn't now receive a refresh token?</h3>
Yes, this is possible since you're developing and you may try authorizing your test applications a couple of time. In the offline access type, Google will issue a refresh token only once and you're expected to use that to gain an access token offline. if you may further authorizations, you wont received the refresh token again.<br/>
<b>Solution: </b> Revoke access from your application and authorize it again. In this case you will have to first manually delete contents in the token.json file so as for the refresh token to be written there.

Now that you can authorize your application and receive a refresh token, let's handle sending emails using this setup. But before we start sending emails, we must create our custom PHPMailerOAuth class since we are not using the default class that uses League OAuth2 client library.

`class PHPMailerOAuth extends \PHPMailer {
	
    /**
     * The OAuth user's email address
     * @type string
     */
    public $oauthUserEmail = '';

    /**
     * The OAuth refresh token
     * @type string
     */
    public $oauthRefreshToken = '';

    /**
     * The OAuth client ID
     * @type string
     */
    public $oauthClientId = '';

    /**
     * The OAuth client secret
     * @type string
     */
    public $oauthClientSecret = '';

    /**
     * An instance of the OAuth class.
     * @type OAuth
     * @access protected
     */
    protected $oauth = null;
    
    /**
     * Get an OAuth instance to use.
     * @return OAuth
     */
    public function getOAUTHInstance()
    {
        if (!is_object($this->oauth)) {
            /* this is the only part that differs,
             * we create an object of our class GmailXOAuth2 instead of the original OAuth class 
             */
            $this->oauth = new GmailXOAuth2 (
                $this->oauthUserEmail,
                $this->oauthClientSecret,
                $this->oauthClientId,
                $this->oauthRefreshToken
            );
        }
        return $this->oauth;
    }

    /**
     * Initiate a connection to an SMTP server.
     * Overrides the original smtpConnect method to add support for OAuth.
     * @param array $options An array of options compatible with stream_context_create()
     * @uses SMTP
     * @access public
     * @throws phpmailerException
     * @return boolean
     */
    public function smtpConnect($options = array()) {
        if (is_null($this->smtp)) {
            $this->smtp = $this->getSMTPInstance();
        }
        
        if (is_null($this->oauth)) {
            $this->oauth = $this->getOAUTHInstance();
        }
       
        // Already connected?
        if ($this->smtp->connected()) {
            return true;
        }

        $this->smtp->setTimeout($this->Timeout);
        $this->smtp->setDebugLevel($this->SMTPDebug);
        $this->smtp->setDebugOutput($this->Debugoutput);
        $this->smtp->setVerp($this->do_verp);
        $hosts = explode(';', $this->Host);
        $lastexception = null;

        foreach ($hosts as $hostentry) {
            $hostinfo = array();
            if (!preg_match('/^((ssl|tls):\/\/)*([a-zA-Z0-9\.-]*):?([0-9]*)$/', trim($hostentry), $hostinfo)) {
                // Not a valid host entry
                continue;
            }
            // $hostinfo[2]: optional ssl or tls prefix
            // $hostinfo[3]: the hostname
            // $hostinfo[4]: optional port number
            // The host string prefix can temporarily override the current setting for SMTPSecure
            // If it's not specified, the default value is used
            $prefix = '';
            $secure = $this->SMTPSecure;
            $tls = ($this->SMTPSecure == 'tls');
            if ('ssl' == $hostinfo[2] or ('' == $hostinfo[2] and 'ssl' == $this->SMTPSecure)) {
                $prefix = 'ssl://';
                $tls = false; // Can't have SSL and TLS at the same time
                $secure = 'ssl';
            } elseif ($hostinfo[2] == 'tls') {
                $tls = true;
                // tls doesn't use a prefix
                $secure = 'tls';
            }
            //Do we need the OpenSSL extension?
            $sslext = defined('OPENSSL_ALGO_SHA1');
            if ('tls' === $secure or 'ssl' === $secure) {
                //Check for an OpenSSL constant rather than using extension_loaded, which is sometimes disabled
                if (!$sslext) {
                    throw new \phpmailerException($this->lang('extension_missing').'openssl', self::STOP_CRITICAL);
                }
            }
            $host = $hostinfo[3];
            $port = $this->Port;
            $tport = (integer)$hostinfo[4];
            if ($tport > 0 and $tport < 65536) {
                $port = $tport;
            }
            if ($this->smtp->connect($prefix . $host, $port, $this->Timeout, $options)) {
                try {
                    if ($this->Helo) {
                        $hello = $this->Helo;
                    } else {
                        $hello = $this->serverHostname();
                    }
                    $this->smtp->hello($hello);
                    //Automatically enable TLS encryption if:
                    // * it's not disabled
                    // * we have openssl extension
                    // * we are not already using SSL
                    // * the server offers STARTTLS
                    if ($this->SMTPAutoTLS and $sslext and $secure != 'ssl' and $this->smtp->getServerExt('STARTTLS')) {
                        $tls = true;
                    }
                    if ($tls) {
                        if (!$this->smtp->startTLS()) {
                            throw new \phpmailerException($this->lang('connect_host'));
                        }
                        // We must resend HELO after tls negotiation
                        $this->smtp->hello($hello);
                    }
                    if ($this->SMTPAuth) {
                        if (!$this->smtp->authenticate(
                            $this->Username,
                            $this->Password,
                            $this->AuthType,
                            $this->Realm,
                            $this->Workstation,
                            $this->oauth
                        )
                        ) {
                            throw new \phpmailerException($this->lang('authenticate'));
                        }
                    }
                    return true;
                } catch (\phpmailerException $exc) {
                    $lastexception = $exc;
                    $this->edebug($exc->getMessage());
                    // We must have connected, but then failed TLS or Auth, so close connection nicely
                    $this->smtp->quit();
                }
            }
        }
        // If we get here, all connection attempts have failed, so close connection hard
        $this->smtp->close();
        // As we've caught all exceptions, just report whatever the last one was
        if ($this->exceptions and !is_null($lastexception)) {
            throw $lastexception;
        }
        return false;
    }
}`

The only difference we have from the original PHPMailerOAuth class is that instead of create an object of the original OAuth class that uses League OAuth2 client library, we now create an object of your custom GmailXOAuth2 class, the rest remains the same...

class Gmail {

	public static function sendMail() {
		
		$mail = Gmail2::setup();
		
		//Set who the message is to be sent from
		$mail->setFrom('sender@gmail.com', 'Brian Matovu');
		
		//Set an alternative reply-to address
		//$mail->addReplyTo('reply_to@gmail.ug', 'James Scott');
		
		//Set who the message is to be sent to
		$mail->addAddress('receiver@gmail.com', 'John Doe');
		
		//Set the subject line
		$mail->Subject = 'PHPMailer GMail XOAuth SMTP';
		
		//Read an HTML message body from an external file, convert referenced images to embedded,
		//convert HTML into a basic plain-text alternative body
		//$mail->msgHTML(file_get_contents('contents.html'), dirname(__FILE__));
		
		$mail->Body = "
				<!DOCTYPE html>
				<html>
				<head>
				<meta charset='ISO-8859-1'>
				<title>Datum :: PHPMailer Testing</title>
				</head>
				<body>
					<h3>Test email</h3>
					<p>This is a test email using phpmailer library 5.1.12</p>
					<hr/>
					<p>Using Google API Client instead of League OAuth2 client </p>
				</body>
				</html>";
		
		//Replace the plain text body with one created manually
		$mail->AltBody = 'AltBody :: This is a plain-text message body';
		
		//send the message, check for errors
		if (!$mail->send()) {
			return "Mailer Error: " . $mail->ErrorInfo;
		} else {
			return "Message sent!";
		}
	}
	
	
	private function setup() {
	
		// Create a new PHPMailer instance
		$mail = new PHPMailerOAuth; /* this must be the custom class we created */
	
		// Tell PHPMailer to use SMTP
		$mail->isSMTP();
	
		// Enable SMTP debugging
		$mail->SMTPDebug = 2;
	
		// Ask for HTML-friendly debug output
		$mail->Debugoutput = 'html';
	
		// Set AuthType
		$mail->AuthType = 'XOAUTH2';
	
		// Whether to use SMTP authentication
		$mail->SMTPAuth = true;
	
		// Set the encryption system to use - ssl (deprecated) or tls
		$mail->SMTPSecure = 'tls';
	
		// Set the hostname of the mail server
		$mail->Host = 'smtp.gmail.com';
	
		// Set the SMTP port number - 587 for authenticated TLS, a.k.a. RFC4409 SMTP submission
		$mail->Port = 587;
		
		// User Email to use for SMTP authentication - Use the same Email used in Google Developer Console
		$mail->oauthUserEmail = 'vdatum@gmail.com';
		
		$gmail_credentials = json_decode(file_get_contents('path_to\gmail-xoauth2-credentials.json'), true);
	
		//Obtained From Google Developer Console
		$mail->oauthClientId = $gmail_credentials['web']['client_id'];
		
		//Obtained From Google Developer Console
		$mail->oauthClientSecret = $gmail_credentials['web']['client_secret'];
		
		$gmail_token = json_decode(file_get_contents('path_to\gmail-xoauth2-token.json'), true);
		
		//Obtained By running get_oauth_token.php after setting up APP in Google Developer Console.
		//Set Redirect URI in Developer Console as [https/http]://<yourdomain>/<folder>/get_oauth_token.php
		// eg: http://localhost/phpmail/get_oauth_token.php
		$mail->oauthRefreshToken = $gmail_token['refresh_token'];
	
		return $mail;
	}
	
}

That is it, so we can now try sending an email
`Gmail::sendMail();`

