Agile Toolkit Overview
######################

To know where framework is heading, you must know where it is coming from. Agile Toolkit is a rewrite of the AModules3 framework which dates back to 1999 and was maintained by the same author, meaning that Agile Toolkit have lived through at least 4 generation and have established a very solid foundation.

Main Qualities of Agile Toolkit
===============================


Not so Volatile
'''''''''''''''

Agile Toolkit reached two years ago where the syntax is so well defined
and accepted that no fundamental changes are longer necessary. You can
safely invest your time into learning Agile Toolkit knowing that the
next release is not going to turn on it's head.

Object-oriented, not Class-oriented
'''''''''''''''''''''''''''''''''''

Majority of PHP frameworks use class-oriented approach, static
singletons, factories or global methods. Agile Toolkit uses classical
Object-Oriented paradigms which are much more suitable for User
Interface manipulation.

Performance and Flexibility
'''''''''''''''''''''''''''

By taking advantage of a quick template engine and widget library, Agile
Toolkit remains very swift and flexible at the same time. It creates
quite a few UI objects as it works with your page, but it's a manageable
amount which would not slow your application more than an extra SQL
query.

Giving you a tool, not a swiss-army knife
'''''''''''''''''''''''''''''''''''''''''

Instead of trying to offer every problem, Agile Toolkit delivers a good
selection of base tools. A skilled developer can use the tools to solve
any problem.

Fully integrated with a CSS framework
'''''''''''''''''''''''''''''''''''''

While AgileToolkit CSS is a great CSS framework on it's own it also
follows a very similar principles to the core framework. In a hands of a
master you will be able to solve any possible problem.

A new web platform
------------------

When applications were switching from DOS to Windows they discovered
that they no longer need to worry about drawing dialog boxes on the
screen. a windowing system can do it more efficiently and more
consistently.

Agile Toolkit is the same to your web application. Not only it is a
framework, but it is a platform you use to develop your application.

This is a very important concept, because as you download an Agile
Toolkit module, it will typically rely on all core features of Agile
Toolkit. This makes add-on and extension development so much simpler,
allows them to use a User Interface and automatically adjust it to your
theme settings.

Academical Importance
---------------------

There are just some web platforms you should stay away if you are just
learning to program. They will teach you bad software development
practices and will turn you into an in-efficient and poor developer. PHP
developers has been frown upon thanks to platforms such as Wordpress and
Drupal. While great products, they are good example of poor coding
practices.

Agile toolkit, on the contrary, is designed to cultivate progressive
thinking and as you learn it, you will become much more proficient
developer. Even if you will have to switch to a different language later
on, the principles you will learn in Agile Toolkit will let you remain a
good software architect.

Two-pass initialization and rendering
-------------------------------------

Most frameworks go through a number of object and have them produce
chunks of HTML which is glued together into a resulting web page output.

Agile Toolkit uses a two-pass approach. Your objects are initialized
during the first step, then objects identify what needs to be repainted
and render only relevant part of the page. This approach natively
enables developers to use partial re-renders and encourage use of AJAX
and building of rich applications.

Agile Toolkit is full-stack
---------------------------

By no means Agile Toolkit is a "minimalistic" framework. It has more
stuff than some mature and heavyweight frameworks. However with
efficient code re-use and clever architecture, Agile Toolkit wins on
performance, is much more lightweight, simple and agile.

Data manipulation
-----------------

Agile Toolkit views support connectivity with Data Sources - which can
be anything: array, SQL, NoSQL or API. This means that creating a form
and linking it with RESTful API is just as simple as saving it into the
SQL table.

SQL Awesomeness
---------------

Practically most frameworks let you use SQL. Not many of them are doing
good job with giving you object-oriented data modeling component. Lately
you get one, but you still need to write your queries in SQL.

Agile Toolkit features a clever objective data and relationship manager
with support for all SQL server features including joins, sub-selects,
expressions and conditions.

jQuery / JavaScript event system
--------------------------------

Agile Toolkit possibly is the only framework which tightly integrates
with JavaScript layer allowing you to write direct bindings between
libraries like jQuery and your Views.

















Everything is an Object
=======================
In our application so far we have defined and created objects, such as:
model, crud, button and error-box. Agile Toolkit only work with objects.

The classes of those objects would inherit from ``AbstractObject``, so
all objects have a common anchestor.

Objects live in a Runtime-Tree
==============================
In Agile Toolkit you don't create instances using a "new" statement, instead
you "add" objects into other objects. It is, in a way, similar, but objects
are by default connected with the rest of your application.

Any object of Agile Toolkit would have a reference to it's owner - the object
which spawned it. It also has a reference to the top-most object - Application
object.

Create a new PHP file called "cmd.php"::

    include'vendor/autoload.php';

    $app = new App_CLI('myapp');

    $model = $app->add('Model', 'mymodel');
    $ctl = $model->add('Controller', 'mycontroller');

    echo $ctl->name."\n";
    // Outputs myapp_mymodel_mycontroller

    echo $model->name."\n";
    // Outputs myapp_mymodel

    echo $ctl->owner->name."\n";
    // Outputs myapp_mymodel

    echo $ctl->app->name."\n";
    // Outputs myapp



.. toctree::
    :maxdepth: 1

    agiletoolkit-overview/design-goals
    agiletoolkit-overview/view
    agiletoolkit-overview/model
    agiletoolkit-overview/controller
    agiletoolkit-overview/application
    agiletoolkit-overview/what-is-cakephp-why-use-it
    agiletoolkit-overview/understanding-model-view-controller
    agiletoolkit-overview/where-to-get-help


.. meta::
    :title lang=en: CakePHP Overview
    :keywords lang=en: web application framework,model view controller,object oriented programming,piece of cake,cookbook,functionality,xml,cakephp
