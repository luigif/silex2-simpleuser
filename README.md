**This is a version of Jason Grimes Simple User which has been modified to work with Silex 2.0. 

#User Manager Provider for Silex2

A simple, extensible, database-backed user provider for the Silex security service.

UserManager is an easy way to set up user accounts (authentication, authorization, and user administration) in the Silex PHP micro-framework. It provides drop-in services for Silex that implement the missing user management pieces for the Security component. It includes a basic User model, a database-backed user manager, controllers and views for user administration, and various supporting features.

##Quick start example config

This configuration should work out of the box to get you up and running quickly. See below for additional details.

Install with composer. This command will automatically install the latest stable version:

```
composer require luigif/silex2-usermanager
```

Set up your Silex application something like this:

```
<?php

use Silex\Application;
use Silex\Provider;

//
// Application setup
//

$app = new Application();
$app->register(new Provider\DoctrineServiceProvider());
$app->register(new Provider\SecurityServiceProvider());
$app->register(new Provider\RememberMeServiceProvider());
$app->register(new Provider\SessionServiceProvider());
$app->register(new Provider\ServiceControllerServiceProvider());
$app->register(new Provider\RoutingServiceProvider());
$app->register(new Provider\TwigServiceProvider());
$app->register(new Provider\SwiftmailerServiceProvider());

// Register the SimpleUser service provider.
$simpleUserProvider = new SimpleUser\UserServiceProvider();
$app->register($simpleUserProvider);

// ...

//
// Controllers
//

// Mount the user controller routes:
$app->mount('/user', $simpleUserProvider);

/*
// Other routes and controllers...
$app->get('/', function () use ($app) {
    return $app['twig']->render('index.twig', array());
});
*/

// ...

//
// Configuration
//

// SimpleUser options. See config reference below for details.
$app['user.options'] = array();

// Security config. See http://silex.sensiolabs.org/doc/providers/security.html for details.
$app['security.firewalls'] = array(
    /* // Ensure that the login page is accessible to all, if you set anonymous => false below.
    'login' => array(
        'pattern' => '^/user/login$',
    ), */
    'secured_area' => array(
        'pattern' => '^.*$',
        'anonymous' => true,
        'remember_me' => array(),
        'form' => array(
            'login_path' => '/user/login',
            'check_path' => '/user/login_check',
        ),
        'logout' => array(
            'logout_path' => '/user/logout',
        ),
        'users' => function() use($app) { return $app['user.manager']; }
    ),
);

// Mailer config. See http://silex.sensiolabs.org/doc/providers/swiftmailer.html
$app['swiftmailer.options'] = array();

// Database config. See http://silex.sensiolabs.org/doc/providers/doctrine.html
$app['db.options'] = array(
    'driver'   => 'pdo_mysql',
    'host' => 'localhost',
    'dbname' => 'mydbname',
    'user' => 'mydbuser',
    'password' => 'mydbpassword',
);

return $app;
```

##Create the user database:

```
mysql -uUSER -pPASSWORD MYDBNAME < vendor/jasongrimes/silex-simpleuser/sql/mysql.sql
```

You should now be able to create an account at the /user/register URL. Make the new account an administrator by editing the record directly in the database and setting the users.roles column to ROLE_USER,ROLE_ADMIN. (After you have one admin account, it can grant the admin role to others via the web interface.)

##Config options

All of these options are optional. SimpleUser can work without any configuration at all, or you can customize one or more of the following options. The default values are shown below.

```
$app['user.options'] = array(

    // Specify custom view templates here.
    'templates' => array(
        'layout' => '@user/layout.twig',
        'register' => '@user/register.twig',
        'register-confirmation-sent' => '@user/register-confirmation-sent.twig',
        'login' => '@user/login.twig',
        'login-confirmation-needed' => '@user/login-confirmation-needed.twig',
        'forgot-password' => '@user/forgot-password.twig',
        'reset-password' => '@user/reset-password.twig',
        'view' => '@user/view.twig',
        'edit' => '@user/edit.twig',
        'list' => '@user/list.twig',
    ),

    // Configure the user mailer for sending password reset and email confirmation messages.
    'mailer' => array(
        'enabled' => true, // When false, email notifications are not sent (they're silently discarded).
        'fromEmail' => array(
            'address' => 'do-not-reply@' . (isset($_SERVER['HTTP_HOST']) ? $_SERVER['HTTP_HOST'] : gethostname()),
            'name' => null,
        ),
    ),

    'emailConfirmation' => array(
        'required' => false, // Whether to require email confirmation before enabling new accounts.
        'template' => '@user/email/confirm-email.twig',
    ),

    'passwordReset' => array(
        'template' => '@user/email/reset-password.twig',
        'tokenTTL' => 86400, // How many seconds the reset token is valid for. Default: 1 day.
    ),

    // Set this to use a custom User class.
    'userClass' => 'SimpleUser\User',

    // Whether to require that users have a username (default: false).
    // By default, users sign in with their email address instead.
    'isUsernameRequired' => false,

    // A list of custom fields to support in the edit controller.
    'editCustomFields' => array(),

    // Override table names, if necessary.
    'userTableName' => 'users',
    'userCustomFieldsTableName' => 'user_custom_fields',

    //Override Column names, if necessary
    'userColumns' = array(
        'id' => 'id',
        'email' => 'email',
        'password' => 'password',
        'salt' => 'salt',
        'roles' => 'roles',
        'name' => 'name',
        'time_created' => 'time_created',
        'username' => 'username',
        'isEnabled' => 'isEnabled',
        'confirmationToken' => 'confirmationToken',
        'timePasswordResetRequested' => 'timePasswordResetRequested',
        //Custom Fields
        'user_id' => 'user_id',
        'attribute' => 'attribute',
        'value' => 'value',
    ),
);
```
