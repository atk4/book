*******
App_Web
*******

.. php:class:: App_Web

App_Web extends :php:class:`App_CLI` to add basic Web handling. Here is a sample
App_Web single-file application. Place the following code in ``myfile.php``::

    $app = new App_Web();
    $app->add('LoremIpsum');

    $app->main();   // executes render and outputs HTML

App_Web assumes that you will be using views and they will produce HTML.

Headers
=======

.. php:method:: sendHeaders

App_Web will send headers to the browsers automatically on init. Headers are
always set to expire in the past and to prevent page caching.

.. php:method:: cleanMagicQuotes

Reverses older PHP behavior which had magic quotes enabled.

Miscelanious
============

.. php:method:: showExecutionTime

License Checking
================

If your configuration file contains Agile Toolkit certificate, Agile Toolkit
will attempt to validate it.

Agile Toolkit framework can operate in commercial and open-source mode. If
no certificate is present, it's assumed that your installation of Agile Toolkit
is not registered.

:doc:`/sandbox` will offer a user-friendly way to select and manage license.

Several other add-ons will rely valid license verification to perform properly.

.. php:method:: license

.. php:method:: licenseCheck

Default Controller
==================

This application will automatically initialize:

- Logger (pretty error logging and reporting)
- PageManager (determining base URL and URL generation)
- HTML Headers (uses no-cache headers by default)
- A basic application template
- Pages and routing (for handling HTTP requests).

EC Cookie Law
=============

To make your web app more compliant with EC Cookie Law the session will only be initialized if you use memorize/recall to store session values. Some Controllers, such as Authentication, rely on memorize/recall and will cause the session to be initialized.

We recommend that you do not use memorize/recall unless necessary. This way you can display a Cookie warning on the log-in screen

`ApiWeb` does not use Page classes by default, so if you want to display any Views you need to add them directly into the Application.

    // In index.php

    include 'atk4/loader.php';
    $api = new ApiWeb();
    $api->add('View_HelloWorld');
    $api->main();

Global Tags
===========
