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
for white-list and black-list strategies.

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

Using The BasicAuth Class
-------------------------

The BasicAuth controller is a simple implementation of the
Authentication class. Below is an example of usage from inside the API
class:

::

    $this->add('BasicAuth')->allow('john','secret')->allow('peter','shoeshine')->check();

The above code will add BasicAuth into the API, manually allow 2 users
to have access, then perform the check. You will see a login form when
accessing any page handled by this API. Class SQLAuth can be used if you
wish to authenticate users against the database.

::

    $auth=$this->add('BasicAuth');
    $auth->setModel('User');
    $auth->useEncryption('sha1')->check();

Implementing White-List
-----------------------

Both black-list and white-list implementation depend on executing - or
bypassing - the "check" method. In a white-list implementation, if
``check()`` is not executed, then authentication is not required. You
can use allowPage() to specify which pages do not need authentication,
and then use ``isPageAllowed()`` to determine when to perform the
``check()``. Note that if the user is logged in, even without having
explicitly called ``check()`` - ie., when visiting an allowed page -
their login information will be available.

::

    $this->add('BasicAuth');     // automatically sets $api->auth to self
    $this->auth->allowPage('privacy');
    $this->auth->allowPage(array('faq','contact','about'));
    $this->auth->setModel('User');
    $this->auth->useEncryption('sha1');
    if(!$this->auth->isPageAllowed($this->page)){
        $this->auth->check();
    }

Implementing black-list
-----------------------

To keep non-authenticated users away from certain pages you should move
the call to ``$api->check()`` to inside those pages.

::

    // inside page/secret.php, in page's init() method

    $this->api->auth->check();
    $this->add('Text')->set('Hello, ',$this->api->auth->model['name']);

Creating your custom authentication page
========================================

TODO

Altirnative validation ways (such as open-id)
=============================================

TODO


Replacing Login form
====================

Hook 'createForm' can be used to create and return your own login
form. You can also extend Auth controller and override method createForm

Enhancing Login form
====================

Auth class defines a hook updateForm, which can be used to enhance login form::

    $auth->addHook('updateForm', function($auth){
        $auth->form->owner                          // link after form
            ->add('View')
            ->setElement('a')
            ->setAttr('href',$this->app->url('forgot'))
            ->set('Forgot password?');
        $auth->form->addClass('atk-push-small');    // add gap after form
    });


Remember that you can destroy some of the original fields if you wish.


Enabling Cookie Authentication (Remember Me)
============================================

If you wish to add or change anything on the log-in form, you can either use a
3rd party controller.

Here you can use a special Auth Controller which will enable cookie-based
login persintance. It will add checkbox with label Remember Me. It will
also enhance log-out functionality to destroy the cookie. If user closes
the browser, the cookie will log-in him next time::

    $a->add('auth/Controller_Cookie');

.. note:: The implementation of cookie controller stores password in a secure
    cookie on the browser. If you wish to store a special authentication token
    instead, you should use your own implementation.



Enabing 3rd party authentication
================================

A 3rd party autehnticaiton is implemented through ``romaninsh/opauth`` controller
and supports any strategy which is supported by "opauth" (PHP implementation for OAuth)

Here is a sample::

    $op=$a->add(
        'romaninsh/opauth/Controller_Opauth',
        array(
            'register_page'=>'ofinish',
            'default_action'=>array('redirect_me'=>'my'),
        )
    );
    $op->addStrategy('github,facebook,twitter,google');

