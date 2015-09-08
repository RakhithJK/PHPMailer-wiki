The official PHPMailer wiki provides an example of using Google's XOAuth2 SMTP authentication using the [League OAuth2 client library](https://github.com/thephpleague/oauth2-client). This library works fine, but has one drawback to it.

For small projects, using league/oauth2-client library to make api call to Google will do just fine but it comes with many dependencies; ircmaxell/random-lib, guzzlehttp/guzzle, and a few others. On top of this, you must install the [league oauth2 google provider library](https://github.com/thephpleague/oauth2-google).

But you can still reduce on the number of third party libraries you're using in your project by using the official [Google API client](https://github.com/google/google-api-php-client) with [Curl](http://curl.haxx.se/). (Ofcourse if you're using the league OAuth2 library generally in you project for example for authentication, then this would not do you so much good - so stick with that)

Firstly, lets install PHPMailer using composer, remember Google's XOAUTH2 SMTP & IMAP authentication mechanism is only supported starting with PHPMailer 5.2.11. So you must install that or later;<br/>
`composer require phpmailer/phpmailer ~5.2`

So hope you have curl install on your working machine, and enabled for php? If not, you may refer to this great discussion; ["How do I install cURL on Windows?"](http://stackoverflow.com/questions/1347146/how-to-enable-curl-in-php-xampp) on stackoverflow.

Once you have Curl installed and configured, use composer to install the Google API client library by <br/>
`composer require google/apiclient 1.*`

