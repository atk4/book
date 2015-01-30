*******************
User Authentication
*******************

Different web applications will employ different authorization and
authentication strategies. In some cases, access to the entire
application is restricted unless the user enters a password. In other
cases, the users can access most of the functions in the application,
however gain additional functionality if authenticated.

Basic Use
=========

Agile Toolkit provides a basic authentication mechanism, with support
for white-list and black-list strategies. The functionality is implemented
through class Auth.

To use auth, you need to create instance of "Auth" then set it up and call
:php:meth:`Auth_Basic::check` method. Auth class will store authentication state through
:php:meth:`AbstractObject::memorize`, therefore you may create multiple instances of Auth and
use them independently. For consistent use it's recommended to add
Auth into your App class.

Auth class directly inherits Auth_Basic. You can create your own
auth controller and override "Auth" class. The rest of this
documentation will be explaining use of :php:class:`Auth_Basic`.

Terminology
-----------

Authentication
    a process where a user provides their username and password, or uses some other means for identifying themselves, such as OpenID.

Authorization
    defining the type of operations which the current user can or cannot do.

white-list
    an authorization strategy under which all actions/pages are restricted, unless specifically allowed. White-list is considered more secure than black-list.

black-list
    an autorization strategy under which a check for restricted actions/pages is performed in order to disallow user access.

Using The Auth_Basic Class
-------------------------

.. php:class:: Auth

Entirely inherits from :php:class:`Auth_Basic`

.. php:class:: Auth_Basic

The Auth_Basic controller is a simple implementation of the
Authentication class. Below is an example of simplest usage form from inside the API
class::

    $this->add('Auth')
        ->allow('john','secret')
        ->allow('peter','shoeshine')
        ->check();

The above code will add Auth_Basic into the API, manually allow 2 users
to have access, then perform the check.

Checking authentication status
==============================

In terms of Auth class user can either be authenticated or not. If user
is authenticated, then Auth class will store some more information about
currently logged in user. Next few methods can help you determine if
the user is logged-in or not.

.. php:method:: check()

    Call this function to perform a check for logged in user. This will also display a login-form
    and will verify user's credential. If you want to handle log-in form on your own, use
    auth->isLoggedIn() to check and redirect user to a login page.

    check() returns true if user have just logged in and will return "null" for requests when user
    continues to use his session. Use that to perform some calculation on log-in

.. php:method:: isLoggedIn()

    This function determines - if user is already logged in or not. It does it by
    looking at $this->info, which was loaded during init() from session.

.. php:attr:: model

    Contains reference to currently associated user model. When user is logged-in
    the model will be :php:meth:`Model::loaded`. You may access this model to
    get more data about the user.

.. warning::

    When user authenticates, his ID and model fields will be stored in a
    session cache. That means if database contents change, the auth model
    will not reflect the change.

    You can freshen up the data from the database with :php:meth:`Model::reload`

    If you call :php:meth:`Model::save` on Auth's model, then it will save into
    database but will also update the cache.


Log-in verification procedure
=============================

In the example above, you have seen the simplest way to set up access
control through a pre-defined user/password combinations.

.. php:method:: allow

    Configure this Auth controller with a generic Model based on static
    collection of user/password combinations. Use this method if you
    only want one or few accounts to access the system.

There are two ways to call `allow`::

    $auth->allow('john', 'secret');
    $auth->allow('peter', 'test123');

Alternatively you may specify associative array with data::

    $auth->allow([
        ['email'=>'john', 'password'=>'secret'],
        ['email'=>'peter', 'password'=>'test123'],
    ]);

A more common way to verify accounts is when your Auth method would
use Model data originating from SQL table or some other source. The
usage is straightforward:

.. php:method:: setModel

    Associate model with authentication class. Username / password
    check will be performed against the model in the following steps:
    Model will attempt to load record where login_field matches
    specified. Password is then loaded and verified using configured
    encryption method.

Here is example of common use with setModel::

    class Model_User extends SQL_Model {
        public $table='user';
        function init(){
            parent::init();

            $this->addField('email');
            $this->addField('password');
        }
    }

    // in lib/Admin.php or lib/Frontend.php

    $auth = $this->add('Auth');
    $auth->setModel('Model_User');

    if(!$auth->isLoggedIn()){
        // add code here to add "Log-In Menu Item"

        // TODO: this may not be accurate.
    }

    $auth->check();

    // add code here to add "Log-Out" Menu Item

.. note:: You must remember, that calling check() will
    eject execution of your Application's init() method
    if the user is not logged in, preventing your
    application from building stardard page layout
    and menu.

    For more information, see :php:meth:`Auth_Basic::showLoginForm`

The 2 additional arguments for the setModel() methods will instruct
auth class which fields are to be used for matching username and
retrieving password. Specify them if your database uses a different fields.

Display of Login form
=====================

When method check() is called it will terminate execution of your PHP
code if user is not currently logged in and instead will show a
login form on the screen.

Agile Toolkit initializes all views first, then calls recursive rendering
(See :ref:`recursive rendering`)


.. php:method:: showLoginForm

    Method `check()` terminate execution of your PHP code,
    if user is not logged in. It's done in the following steps:

    First - your current page objects will be destroyed and
    substituted with a brand new ones.


Creating custom login page
--------------------------

The simple way how to use your own fancy login form is if you create
a dedicated page (page_login) and redirect non-authenticated users
there if auth things they are not authenitcated::

    $m = $this->add('Model_User');

    $m->addCondition('is_confirmed', true); // only permit confirmed users to login

    $auth = $this->add('Auth');
    $auth->setModel($m, 'user');        // using custom login field

    if(!$auth->isLoggedIn() && $this->page!='login'){
        $this->redirect('login');
    }

Here is a typical code you would need to place on your own login page::

    $form = $this->add('Form');
    $form->addField('Line','user');
    $form->addField('Password','password');
    $form->onSubmit(function($form){

        if(!$form->app->auth->verifyCredentials(
            $form['user'],
            $form['password']
        )){
            return $form->error('password','Login incorrect');
        }

        // Seems to be ok, lets login the user
        $form->app->auth->login($form['user']);

    });

As you can see, you can use the above template for other purposes. You may
be checking authentication token directly against auth->model and then
explicitly logging user in.

Replacing Login form
--------------------

Auth class supports hook createForm which can help you simply build a
different Form without going through the trouble of setting up
a login page.

If hook 'createForm' is defined, it will be used to build a login form
form. (Please note that fields 'username' and 'password' should be
in the returned form::

    $auth->addHook('createForm', function($page){

        $form = $page->add('Form');
        $form->addField('Line','username',$email);
        $form->addField('Password','password',$password);
        $form->addSubmit('Login');
        return $form;

    });


Enhancing Defaut Login form
---------------------------

Auth class defines a hook updateForm, which can be used to enhance login form::

    $auth->addHook('updateForm', function($auth){
        $auth->form->owner                          // link after form
            ->add('View')
            ->setElement('a')
            ->setAttr('href',$this->app->url('forgot'))
            ->set('Forgot password?');
        $auth->form->addClass('atk-push-small');    // add gap after form
    });

Remember that you can destroy some of the original elements if you wish,
but 'username' and 'password' fields must be present in the form when
you are done.


Password Encryption
===================

.. php:method:: verifyCredentials

    This function verifies credibility of supplied authenication data.
    It will search based on user and verify the password. It's also
    possible that the function will re-hash user password with
    updated hash.

    if default authentication method is used, the function will
    automatically determine hash used for password generation and will
    upgrade to a new php5.5-compatible syntax.

    This function return false OR the id of the record matching user

.. php:method:: usePasswordEncryption

    Specifies how password will be encrypted when stored. It's recommended
    that you do not specify encryption method, in which case a built-in
    password_hash() will be used, which is defined by PHP.

    Some other values are "sha256/salt", "md5", "rot13". Note that
    if your application is already using 'md5' or 'sha1', you can
    remove the argument entirely and your user passwords will keep
    working and wil automatically be "upgraded" to password_hash
    when used.

    If you are having trouble with authentication, use auth->debug()

.. php:method:: addEncryptionHook

    Adds a hook to specified model which will encrypt password before save.
    This method will be applied on $this->model, so you should not call
    it manually. You can call it on a fresh model, however.

When you execute Auth::setModel() Agile Toolkit will add a hook on
your model (beforeSave) that will encrypt the password if it was modified.
This makes it super simple for you to change password::

    $m = $this->add('Model_User');
    $this->app->auth->addEncryptionHook($m);

    $m->loadBy('email', 'joe@mail.com');
    $m['password'] = 'foo';
    $m->save();

This code will change password of a specified user. I recommend you
using $this->app->auth->model in which case you do not need to worry
about the hook::


    $m = $this->auth->model;
    $this->app->auth->addEncryptionHook($m);

    $m->loadBy('email', 'joe@mail.com');
    $m['password'] = 'foo';
    $m->save();

.. note:: Please remember that auth->model may be loaded. If you use
    tryLoad in this scenario you may have undesired effects.

If you operate front-end and Admin, then typically you would use
different authentication methods in each interface. If you want Admin
to be able to change password for the front-end, you can use the following
code::

    $auth = $this->add('Auth');     // Independent Auth Controller.
    $auth->setModel('Model_User');
    $auth->usePasswordEncryption();

    $m = $auth->model()->unload();
    $m->loadBy('email', 'joe@mail.com');
    $m['password'] = 'foo';
    $m->save();


Implementing White-List
=======================

I'd like to further improve this example from a "custom login page" section
above::

    if(!$auth->isLoggedIn() && $this->page!='login'){
        $this->redirect('login');
    }

This piece of code would redirect non-authenticated user to "login"
page unless he is on that page already. In reality, there are usually
many other pages which could be considered "safe" for non-authenticated
users to visit: password reminder, signup and so on.

.. php:meth:: allowPage($page)

    Specify page or array of pages which will exclude authentication. Add your registration page here
    or page containing terms and conditions

.. php:meth:: isPageAllowed($page)

    Verifies if the specified page is allowed to be accessed without
    authentication.

Calling method :php:meth:`Auth_Basic::check` will not require authentication
if user is located on one of the allowed pages::

    $auth->allowPage('signup');
    $auth->allowPage('reminder');
    $auth->check();

or alternatively::

    $auth->allowPage(['signup','reminder']);

Revisiting the code above::

    $auth->allowPage(['login', 'signup', 'forgot'])

    if(!$auth->isLoggedIn() && !$auth->isPageAllowed($this->page)){
        $this->redirect('login');
    }


Implementing black-list
=======================

In the scenario with black-list you only want authentication on one
page or sub-set of pages. In this scenario you should still add
auth controller from inside Application init, however call
:php:meth:`Basic_Auth::init` from the init method of
restricted pages. I recommend you creating a separate page class
for that::

    class RestrictedPage extends Page {
        function init(){
            parent::init();
            $this->app->auth->check();
        }
    }

For the situation when you wish all pages starting with certain
pattern to match, you can use this code::

    list($top_page, $junk) = explode('_',$this->page, 2);
    if($top_page == 'admin')$this->app->auth->check();


Alternative validation ways
===========================

If your model contains method ``verifyCredentials`` then it will
be called with username and password by the Auth class. Your custom
method must return either true or false.

Auth_Basic extensions
=====================

There are several extensions to Auth_Basic, which you can use or
you can create your own extensions.


Enabling Cookie Authentication (Remember Me)
--------------------------------------------

If you wish to add or change anything on the log-in form, you can either use a
3rd party controller.

Here you can use a special Auth Controller which will enable cookie-based
login persintance. It will add checkbox with label Remember Me. It will
also enhance log-out functionality to destroy the cookie. If user closes
the browser, the cookie will log-in him next time::

    $a->add('auth/Controller_Cookie');

The add-on source is located here: https://github.com/atk4/atk4-addons/blob/master/auth/lib/Controller/Cookie.php

.. note:: The implementation of cookie controller stores password in a secure
    cookie on the browser. If you wish to store a special authentication token
    instead, you should create a different extension.

Creating your own extension
---------------------------

You can create your own extension. There is another code example apart from
the Cookie login implementation that you can use:

https://github.com/atk4/atk4-addons/blob/master/auth/lib/Controller/DummyPopup.php

This extension will add a new button to a login form (Pick a User), that will
open up new window with the list of all available users. Clicking on a user
will authenticate you with the specified user.

.. warning:: This extension is insecure as it allows anyone to authenticate as anyone.

If you create your own extension for Authenication, please share it with others
by placing it on Github and announcing it in our forums.

Implementing "Log-in As" transition between Admin and Frontend
==============================================================

If you have Frontend and Backend, then you might want to allow admin to be able
to authenticate with any valid user. The challenge here is that you must
transition user between "admin" and "frontend" interfaces, wich might be
located on a different URLs and even different domains.

There are no unified way to implement this, but the concept is simple:

1. Define a shared secret which will be the same for admin and frontend. Keep
it well secured and always use SSL connections for both interfaces.

2. From the admin, redirect to frontend but pass an authentication cookie
along::

    // in Admin
    $user_id = $_GET['user_id'];
    $paload = $user_id.':'
        .hash_hmac('sha256', $user_id, $this->app->getConfig('shared_secret'));
    $url = $frontend.'?sudo_auth='.urlencode($payload);

    // inFrontend
    if($_GET['sudo_auth']){
        list($user_id, $sig) = explode(':',$_GET['sudo_auth']);
        if($sig != hash_hmac('sha256', $user_id, $this->app->getConfig('shared_secret')){
            // sig incorrect
            throw $this->exception('incorrect sudo_auth token');
        }

        $auth->loginByID($user_id);
    }

3. Frontend will decode the argument and verify signature. While this is the
most basic implementation, I recommend you adding "date" inside payload
(in case URL was intercepted) and time-out tokens. You can also use shared
database for storing temporary tokens if you don't like to have shared secrets.


Enabing 3rd party authentication
================================

Romans have implemented OAuth authentication extensions for Agile Toolkit, which
relies on OPauth (http://opauth.org).

A 3rd party authenticaiton is implemented through ``romaninsh/opauth`` controller
and supports any strategy which is supported by the 3rd party OPauth library.

https://github.com/romaninsh/opauth

Here is how you can add github, facebook, twitter and google login to your
Agile Toolkit login form::

    $op=$a->add(
        'romaninsh/opauth/Controller_Opauth',
        array(
            'register_page'=>'ofinish',
            'default_action'=>array('redirect_me'=>'my'),
        )
    );
    $op->addStrategy('github,facebook,twitter,google');

You will also need to add something like that into your config file::

    $config['opauth']=array(
        // See: https://github.com/uzyn/opauth/wiki/Opauth-configuration
        'Strategy'=>array(
            'GitHub'=>array(
                'client_id'=>'23981937',
                'client_secret'=>'918237912873918273918273'
            )
        )
    );


You are welcome to contribute additional documentation or implementation
improvements for OPauth extension.
