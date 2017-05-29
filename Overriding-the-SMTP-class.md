The SMTP class is usable by itself. It only provides basic SMTP commands and does not provide higher-level operations like "send a message" that involves a sequence of commands - that is left to the controlling code, which is typically the PHPMailer class. Historically this SMTP class was a hard-coded dependency from PHPMailer, which made it difficult to substitute your own subclass or reimplementation. Recently this was altered so that you could do this, but it's slightly fiddly as it was done in a way that would not break backward compatibility.

Given an SMTP subclass:

```php
class mySMTP extends SMTP {
    //Custom stuff...
}
```

Create a PHPMailer subclass and override the `getSMTPInstance()` method to point at it:

```php
class myMailer extends PHPMailer {
    public function getSMTPInstance()
    {
        if (!is_object($this->smtp)) {
            $this->smtp = new mySMTP;
        }
        return $this->smtp;
    }
}
```

Now your `myMailer` instance will use your `mySMTP` class. Ideally there should be an SMTP interface file in order to make reimplementations more reliable, but it's not likely you want to do that - though pull requests are always welcome!

# PHPMailer 6.0 alternative

PHPMailer 6.0 adds a method that allows you to inject your own SMTP implementation, so you no longer need to override the PHPMailer class to use a custom SMTP class. It works like this:

```php
$mail = new PHPMailer;
$mail->setSMTPInstance(new mySMTP);
```

Note that the instance you inject must be an instance or a subclass of the `PHPMailer\PHPMailer\SMTP` class.