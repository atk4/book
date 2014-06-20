*******
App_CLI
*******

.. php:class:: App_CLI

    A most basic application class.

This is the most basic application class. The features it offers include:

- Configuration File Interface
- Error logging
- Application version tracking
- Global method support
- Localization
- PathFinder
- URL Generation
- Database Connectivity
- Few low-level utility methods

The App_CLI class does not offer command-line related utility such as argument
parsing, help or color output (I agree that name is a little missleading).

App_CLI is most suitable for using in command-line, cron handlers and possibly
some script which output non-standard things. If you are also building API
interface (but not compatible with :php:class:`App_REST`) you might use App_CLI.

There are no execution loop or any routing, you can create your own routing logic
the way you like.

Integration with 3rd Party Frameworks
=====================================

Another great use for App_CLI is integrating with other frameworks. If you wish
to use Agile Toolkit within a wordpress to render a single form, App_CLI might
be just enough for you.

Previously I have used file ``atk4/loader.php``, but if you are using Composer
you can install Agile Toolkit (in AGPL software) by addign::

    require: "atk4-4.3"

In you ``composer.json`` file. Next example will show how you can output
only one form::

    $app = new App_CLI();
    $app->add('Form', [ 'js_widget'=>false ]);
    $form->addField('name');
    $form->addField('surname');
    $form->addSubmit();
    $form->render();

Because we are using App_CLI, then things like JavaScript binding and POST processing
will not work. This will simply output the form, nothing much more. Continue on to
:php:class:`App_Web` to find out how to create a working form.

Using init() method
===================

Like all the other objects, when you initialize App_CLI it will call init() method.
If you would like to structure your App_CLI applciation properly, you should
extend this class and populate init() method:

.. php:method:: init

    Performs initialization. Extend and override to initialize your app.


Example::

    class MyApp extends App_CLI {
        function init() {
            parent::init();

            echo $this->readConfig('test');
        }
    }

Now if configuration parameter will be missing, the error will be handled
better.

.. _configuration:

Configuration File Interface
============================


.. php:method:: readConfig($file)

    Will read specified configuration file.

.. php:attr:: config

    Contains config from the file. Do not access directly.

.. php:attr:: config_location

.. php:attr:: config_files

.. php:attr:: config_files_loaded

.. php:attr:; setConfig

.. php:attr:: getConfig

In line with our design goal of simplicity, Agile Toolkit uses a PHP array for storing configuration values. Here's how it works:

#. In the root directory of your app, you will find the files ``config-default.php`` and ``config-distrib.php``
#. The file ``config-distrib`` is a template and is never loaded - copy it to a file named ``config.php`` in the same directory when you are installing application for the first time. Avoid adding ``config.php`` into your distribution as it may contain sensitive data (database access). Make sure you do not overwrite this file or you loose some of your settings.
#. If Agile Toolkit finds file ``config-default.php`` it will include it before ``config.php``. You can put some shared configuration options there which can still be overridden inside ``config.php``.

Setting Config Values
---------------------

Agile Toolkit makes extensive use of default values, so you need very few settings to configure a working application. This is just a sample - you should add settings as you require::

    $config['dsn']='mysql://user:secret@localhost/my_db';

    $config['sample']['setting'] = 20;

    $config['billing']['realex']['merchantid'] = 'agile';
    $config['billing']['realex']['account'] = 'internet';
    $config['billing']['realex']['secret'] = 'xLmpVrtzYu';

    $config['logger']['log_output'] = 'full';
    $config['logger']['log_dir'] = 'logs';

Using Config Values
-------------------

To use a config value, call the ``getConfig()`` method in your application object::

    // Use '/' to separate array keys
    $secret = $api->getConfig('sample/setting');

    // Optionally, set a default value to use if no setting is found
    $secret = $this->api->getConfig('sample/setting', 10);

Default Configuration Settings:
-------------------------------

Here are the most important defaults in the Toolkit Core:

-  timezone - sets ``date_default_timezone_set`` for you.
-  license - registration data for Agile Toolkit. See installation
   wizard.
-  session - override cookie settings for session, see
   ``session_set_cookie_params``
-  auth/key - obsolete. Was used for SHA1 unique salt generation in
   Auth\_Basic
-  dsn - connection string for SQL database
-  mongo/db - database for Controller\_Data\_Mongo

-  date/js - passed to date picker
-  locale/data - date formatting inside Grid
-  locale/datetime - date formatting inside Grid

-  js/versions/jquery - custom version of jQuery
-  js/versions/jqueryui - custom version of jQuery UI
-  js/jquery - set URL for jQuery if you wish to use CDN
-  js/jqueryui - set URL for jQuery if you wish to use CDN
-  logger/log\_dir - folder where logs are written. By default
   ``/var/log/atk4/REALM``
-  logger/log\_output - how much info to write into log: false, true
   (for short) or ``full`` (default ``"false"``)
-  logger/web\_output - how much information to show to user, false,
   true (for short string) or ``"full"`` (default ``"full"``)

-  url\_prefix - prepends to page name in URLs, for example
   ``index.php?page=`` unless you have mod\_rewrite.
-  url\_postfix - what to add after page name, such as ``.html`` - will
   look like user is accessing HTML files in the url.

-  atk/base\_path - normally ATK will look for ATK assets (such as css
   files, images) inside ``./akt4``. If you are using mod-rewrite then
   sub-pages will break this behaviour. Set a proper prefix for ``atk4``
   folder, such as ``/myproj/atk4``.
-  atk/base\_url - prefix URL for your application such as
   ``http://mysite.com``
-  smlite/extensions - how do you want to end your temp ate files. Don't
   change (default is ``.html``) this or you'll have to rename all your
   templates inside atk4/templates/shared
-  tmail/transport - by default TMail sends email with ``mail()``
   method. You can set a different method here. If you are developing
   locally then using ``debug`` transport is a great idea - it will not
   send emails, but will simply dump email to the screen.
-  tmail/from - specifies the default sender for all emails sent from
   Agile Toolkit (TMail)

Loading Additional Config Files
-------------------------------

Sometimes it's handy to keep the config settings for a part of your app
in an additional file. Read them in with the ``readConfig()`` method::

    $this->api->readConfig('config-additional.php');


Error logging
=============

Agile Toolkit relies on Exception (See :php:meth:`AbstractObject::exception`).
Most of those exceptions will bubble up to the application level.

App_CLI includes a method which is designed to receive all unhandled exceptions:

.. php:method:: caughtException

Stop exceptions
===============

Some exceptions of Agile Toolkit will simply terminate execution of init or
render loop::

    throw $this->exception(null, 'Exception_StopInit');
    // place this inside init, to use it as bail-out.

The reason to use this exception is when your view is palced inside another
view and init() methods being called recursively cannot be easily aborted.
This exception will bubble out to the very top and make sure no more init
methods are executed.

:php:class:`VirtualPage` will use StopInit exception to bypass the rest

Version tracking
================

Some Agile Toolkit add-ons may requrie certain version of Agile Toolkit. This
method allows you to request a minimum version of Aglie Tolkit.

.. php:method:: requires(component, version)

    Component may be omitted and is 'atk' by default. You can also specify
    some other component name here. This would be used typically by commercial
    modules.

.. php:method:: getVersion(component)

    Return version of component (atk by default), such as "4.3".

Global Methods
==============

You can register a global method which will be present in all the
objects created under the same application.

.. php:method:: addGlobalMethod

Example::

    $obj->api->addGlobalMethod('test', function($obj, $first_arg, $second_arg){

        echo "Values are $first_arg, $second_arg";
    });

    $this->add('OtherObject')->test(1, 2); // Outputs: "Values are 1, 2"



Localization
============

Application class has a method ``_($str)`` which returns translated copy
of your string. Agile Toolkit core does not offer any particular implementation
of localization plugin, however you can override this method in your application
and introduce your own method, such as using ``gettext``.

Additionally Application class has hook ``localizeString`` which can be used
to supply translation through add-on.

.. php:method:: _

    Localize string

PathFinder
==========

For a full documentation on PathFinder see :doc:`/controller/pathfinder`.

There are few methods in ``App_CLI`` offering a quick access to a current
pathfinder object:

.. php:method:: locate
.. php:method:: locateURL
.. php:method:: locatePath
.. php:method:: addLocation


URL building
============

Calling Application's method url() is a handy way to create a new :php:class:`URL`
object.

.. php:method:: url


Database Connectivity
=====================

For full documentation on database connectivity see :php:class:`DB`

.. php:method:: dbConnect



Other miscelanious methods
==========================

.. php:method:: normalizeName

.. php:method:: normalizeClassName

