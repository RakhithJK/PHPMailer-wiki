You may encounter the error **smtp connect() failed. https //github.com/phpmailer/phpmailer/wiki/troubleshooting oauth2**

You can trouble shoot by adding following code to PHPMailer.php

**new Method**

    public function initiateOauth()
    {

        $options ['provider'] = new Google([
                'clientId' => $this->oauthClientId,
                'clientSecret' => $this->oauthClientSecret
            ]
        );

        $options['userName'] = $this->oauthUserEmail;
        $options['clientId'] = $this->oauthClientId;
        $options['clientSecret'] = $this->oauthClientSecret;
        $options['refreshToken'] = $this->oauthRefreshToken;


        try {
            $this->setOAuth(new OAuth($options));
        } catch (OAuthException $e) {
        }


    }

**Insert Code **

**After:**
                    if ($this->SMTPAuth) {

-----------------------
**                        if ($this->AuthType == 'XOAUTH2') {**
                            **$this->initiateOauth();**
                        **}**

-----------------------------
**Before**
                        if (!$this->smtp->authenticate(
                            $this->Username,
                            $this->Password,
                            $this->AuthType,
                            $this->oauth
                        )
                        ) {
                            echo 'test-2';
                            throw new Exception($this->lang('authenticate'));
                        }
                    }