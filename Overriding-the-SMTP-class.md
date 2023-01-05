The SMTP class is usable by itself. It only provides basic SMTP commands and does not provide higher-level operations like "send a message" that involves a sequence of commands - that is left to the calling code, which is typically the PHPMailer class. Historically this SMTP class was a hard-coded dependency from PHPMailer, which made it difficult to substitute your own subclass or reimplementation. Fortunately that changed, and it's now possible to override the SMTP class that PHPMailer uses.

## PHPMailer 6.0+

PHPMailer 6.0 added a method that allows you to inject your own SMTP implementation:

```php
$mail = new PHPMailer;
$mail->setSMTPInstance(new mySMTP);
```

Note that the instance you inject **must** be an instance or a subclass of the `PHPMailer\PHPMailer\SMTP` class.

## Older PHPMailer versions

It's slightly fiddly in older versions of PHPMailer as it was done in a way that would not break backward compatibility. You need to override the PHPMailer class to use a custom SMTP class.

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

Now your `myMailer` instance will use your `mySMTP` class instead of the default one.