WordPress uses PHPMailer for its [`wp_mail()` function](https://developer.wordpress.org/reference/functions/wp_mail/). A common problem is that PHPMailer is hidden behind this interface, so it's initially difficult for you, or plugins, to get at the underlying instance to change its parameters. For that reason, plugins often bundle their own copy of the PHPMailer package. This is generally a bad idea, as often it will not be kept up to date, and it's adding bloat to your Wordpress installation which already provides the package.

Fortunately WordPress provides [a hook called `phpmailer_init`](https://developer.wordpress.org/reference/hooks/phpmailer_init/) that allows you to get your hands on the internal PHPMailer instance and configure it. Then, when you call `wp_mail`, it will use the PHPMailer instance configured as you like.

To make this work, we ask WordPress (using [`add_action()`](https://developer.wordpress.org/reference/functions/add_action/) to call a function we have defined when the `phpmailer_init` event happens. As the docs for this hook say, the function is passed an instance of PHPMailer, so we define the function to expect that.

The second parameter to `add_action` is a [`callable`](https://www.php.net/callable), which means there are multiple ways of providing a reference to code that will be run by this hook. The most straightforward is an anonymous function (which pollute your namespace with unnecessary functions that will never be called from anywhere else), but you can also provide a simple function name, class and method names in an array, and any of the other ways that PHP allows for callables. Note that the PHPMailer instance is passed by reference, so you don't need to `return` anything from this function.

```php
add_action('phpmailer_init', function ($mailer){
  $mailer->isSMTP();
  $mailer->Host = "mail.example.com"; // your SMTP server
  $mailer->Port = 25;
  $mailer->SMTPDebug = 2;
  $mailer->CharSet  = "utf-8";
  //etc
});
```

Once you've set that up, calling `wp_mail` will work as usual, but using the configuration you set in this function instead of WordPress' own config.