***********
Application
***********

What is the smallest imaginable application in Agile Toolkit? It's about
three lines of code::

    include 'atk4/loader.php';
    $app = new App_CLI();
    echo "Hello World\n";

You can create software for a variety of different targets with Agile
Toolkit including command-line software or software which does not
produce HTML.

The creation of the App_CLI class does almost nothing and is a very
lightweight operation. Once you have created your Application class you can
perform various things with it. For example, we can attempt to read a
configuration parameter::

    include 'atk4/loader.php';
    $app = new App_CLI();
    echo $app->getConfig('greeting',"Hello World\n");

This will open the config.php file in the same folder, and look for the
following line::

    $config['greeting']='Hello from Agile Toolkit';

and the output now would be different. Let's also connect to a MySQL
database by adding the following line inside config.php::

    $config['dsn']='mysql://root:secret@127.0.0.1/myproject';

add to main PHP file::

    $app->dbConnect();

Application Classes
===================

A command-line applications tend to be just like that, but in web applications
there are many other things we constantly need - reading headers, routing,
pages, javascript integration and much more.

A class :php:class:`App_Web` extends :php:class:`App_CLI` to add more features
which are typical for a web application such as headers and more. Next comes
:php:class:`App_Frontend` which adds more routing and layout features.

You also have :php:class:`App_Admin` and :php:class:`App_Installer` for various
other tasks. I recommend that you start from the base classes and gradually
look through all the features which end up being in a top-level application classes.

Finally, we will need to talk about YOUR application class. It's typically called
``Frontend`` or ``Admin``. In your class you are going to add even more features.
As a good developer you should follow the extension pattern and try to make
features you add also reusable.

====================================

.. toctree::
    :maxdepth: 1

    app/cli
    app/web
    app/frontend
    app/admin
    app/rest
    app/installer



