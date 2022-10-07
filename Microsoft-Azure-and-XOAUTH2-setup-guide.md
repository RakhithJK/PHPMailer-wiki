Using XOAUTH2 can be quite a bit of a pain in the... you know... 

This guide will show you how to set up XOAUTH2 with Microsoft Azure / Office365. 

## Important part 1
[The OAuth2 libraries this depends upon](https://packagist.org/packages/greew/oauth2-azure-provider) require **PHP 7.3 or later**, so you need to be running at least that in order to be able to use this authentication system. 

Because of this requirement, this package is **not** enabled by default in PHPMailer's `composer.json` file, but appears [in the 'suggests' section](https://github.com/PHPMailer/PHPMailer/blob/master/composer.json#L49). You should take the suggested package and add it to your own `composer.json` file (the same one you use to install PHPMailer itself in your own project, **not** PHPMailer's own composer file), and then re-run `composer install` to load it. 

This example requires the [greew/oauth2-azure-provider](https://packagist.org/greew/oauth2-azure-provider) OAuth2 provider, but any provider connecting to Microsoft will do.

## Important part 2

Microsoft Azure client secrets expires after a maximum of between 3 and 24 months (you can select the length yourself).

If your authentication suddenly doesn't work anymore, please check the if the Client Secret has expired.

## Configure an OAuth2 app in Microsoft Azure

The following is a step-by-step guide to configuring an OAuth2 app in your Azure Portal. 

Go to https://portal.azure.com and log in using your usual credentials. 

![image](https://user-images.githubusercontent.com/189321/194594349-6d72d2e5-1d20-4e55-84f4-c37cf58573a5.png)

1. Go to the Azure Active Directory link in the left sidebar.
2. Click the Add button and ...
3. Select App registration.

![image](https://user-images.githubusercontent.com/189321/194622747-042c72bd-1601-4b25-8a73-66dd4d4ce78c.png)

Enter data for your application:

1. A name for your application.
2. Select who should be able to use this application.
3. Select Web in the dropdown.
4. Provide a Redirect URI. Even though it says `(optional)`, it most certainly isn't. In our case we will use the [`get_oauth_token.php`](https://github.com/PHPMailer/PHPMailer/blob/master/get_oauth_token.php) bundled with PHPMailer. This script must be run on a webserver either on your local machine (as shown in the screenshot) or on another host. More on that part later!
5. Click Register when you are finished.

You will now be sent to your app page.

![image](https://user-images.githubusercontent.com/189321/194629151-d86538a7-ff59-4f84-b869-7404b2c00367.png)

1. Copy down the "Application (client) ID" ...
2. ... and also the "Directory (tenant) ID" - we will need them later.
3. We now need to create a new client secret - click the "Add a certificate or secret" link.

![image](https://user-images.githubusercontent.com/189321/194625683-bf49d775-4c37-47ff-9a02-0766ad16ee65.png)

1. First, click on the "Client secrets (0)" tab, if it is not already active.
2. Click "New client secret"

![image](https://user-images.githubusercontent.com/189321/194626380-ef6d4295-88e8-4413-9094-37256cfa2f64.png)

1. Give your secret a name.
2. Set when the secret should expire (Microsoft forces a max of 24 months after which the service won't work anymore until a new secret has been created, tokens been made and added to the mailer script).
3. Click "Add".

![image](https://user-images.githubusercontent.com/189321/194626246-b1f0e768-3ef3-4c43-b7ba-637eaa3e46c2.png)

[1. Click the "Copy to clipboard" link at the end of the Value and save this value somewhere safe. You will need this secret later.](#clientSecret)

![image](https://user-images.githubusercontent.com/189321/194626606-f97b8e78-cd3e-4a5b-8892-379bbda8a519.png)

1. We now to set up which scopes this script needs to work. Click the "API permissions" link.

![image](https://user-images.githubusercontent.com/189321/194626837-d8f545e7-6963-4488-bff8-ead644c922d6.png)

1. Click the "Add a permission" link

![image](https://user-images.githubusercontent.com/189321/194626977-e84e0545-34cf-44be-be3a-6e5e51587d7e.png)

1. Select the "Microsoft Graph" API

![image](https://user-images.githubusercontent.com/189321/194627110-171acea1-4ce0-42cd-bade-933a83be7f19.png)

1. Click the "Delegated Permissions" block

![image](https://user-images.githubusercontent.com/189321/194627328-264bda24-db49-4016-929a-2d9b90170d0a.png)

1. First, click the checkbox for `offline_access`. 
2. Now, search for `SMTP` in the search box.

![image](https://user-images.githubusercontent.com/189321/194627939-f153f1f5-182b-4e86-986e-9650f1f11569.png)

1. Expand the SMTP group and click the `SMTP.Send` checkbox.
2. Click the "Add permissions" button

## Obtaining "Authorization Code", "Access Token" and "Refresh Token"

Actually, both the Authorization Code and Access Token won't be shown to us nor needed in this part. We just need to obtain the Refresh Token to be able to use the mailer.

This part of the guide makes the following assumptions:
* that you're on a linux system with a terminal
* that you have the PHPMailer repository checked out on your local computer
* that you use `composer` for package management

Go the to PHPMailer repo folder on your computer.

Run the following command: 

```sh
composer require greew/oauth2-azure-provider
```

We now have the needed packages installed. We now need to spin up a webserver. If you already have a webserver running, you can copy the PHPMailer folder into this. Else, use the following command

```sh
sudo php -S 127.0.0.1:80
```

The webserver has now been spun up! 

Open a browser and go to [`http://localhost/get_oauth_token.php`](http://localhost/get_oauth_token.php). You will now see a page like this:

![image](https://user-images.githubusercontent.com/189321/194644546-ac651889-0685-4b6d-b512-9ed1ccaef4d1.png)

Fill in the following information

- *Provider* should be "Azure"
- *ClientID* should be the value, you copied from "Application (client) ID"
- *ClientSecret* should be the value you copied from the "Client secret"
- *TenantId* should be the value you copied from the "Directory (tenant) ID".

Click "Continue".

If all steps have been successful, you should now get a screen like this (or maybe a login screen at first :).

![image](https://user-images.githubusercontent.com/189321/194645971-97543858-a62b-45d2-b464-f81c26cb72c9.png)

Click "Accept".

You will now be forwarded to your own page again with a branch new Refresh Token.

![image](https://user-images.githubusercontent.com/189321/194646279-cb286034-c18f-4f2a-87f0-badf8f78394c.png)

Copy everything except the "Refresh Token: " part at the beginning and save at the same place as the other values.

## Setting up a mailer

Now we have all information needed to send mails.

See an example in [azure_xoauth2.phpx](https://github.com/PHPMailer/PHPMailer/blob/master/examples/azure_xoauth2.phps)