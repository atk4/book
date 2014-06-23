Pathfinder - resource management
================================

Pathfinder is a system-wide controller for locating different types of
your application resources: classes, images, css files, etc.

Pathfinder is accessible through ``$this->api->pathfinder`` however some
methods are duplicated in the API class for easier access. Here is a
sample use:

::

    $mail_template = $this->api->locate('mail','welcome.mail');
    // returns templates/mail/welcome.mail

In Agile Toolkit different components and add-ons can contribute by
registering additional locations within PathFinder helping your
application locate the necessary resources without copying files.

Purpose of PathFinder
---------------------

No doubt you are familiar with "INCLUDE\_PATH" which have been used in
PHP for locating include files. Pathfinder applies similar approach to
other type of resources. Below you will see the commonly used resource
types. You can easily extend by registering your own types of resources.

+----------+---------------------------+---------------------------------------------------------------------+
| Type     | Default Location          | Description                                                         |
+==========+===========================+=====================================================================+
| php      | lib, atk4/lib             | Contains class files from the default namespace.                    |
+----------+---------------------------+---------------------------------------------------------------------+
| page     | page                      | Location for "page" files, which can be directly accessible through |
|          |                           | standard routing (See routing)                                      |
+----------+---------------------------+---------------------------------------------------------------------+
| template | templates, atk4/templates | Contains SMLite template files with extension .html                 |
+----------+---------------------------+---------------------------------------------------------------------+
| public   | public                    | Contains publicly available resources, such as CSS, JS, etc files   |
+----------+---------------------------+---------------------------------------------------------------------+
| js       | public/js                 | JavaScript include files                                            |
+----------+---------------------------+---------------------------------------------------------------------+
| css      | public/css                | Stylesheets                                                         |
+----------+---------------------------+---------------------------------------------------------------------+
| mail     | public/mail               | TMail template location                                             |
+----------+---------------------------+---------------------------------------------------------------------+

Default locations will already be set by Agile Toolkit, however you can
register more locations for existing types or even for new types.

Locations
---------

PathFinder relies on the concept of "Locations". Each location defines
it's own contents through an array. For example:

::

    $this->atk_location=$this->addLocation(array(
        'php'=>'lib',
        'template'=>'templates',
        'public'=>'public/atk4',
        'js'=>'public/atk4/js',
        'css'=>'public/atk4/css',
    ));

This describes resources which come with Agile Toolkit.

Often addons would bundle more resources and will register additional
locations. For example a standalone CMS product developed on Agile
Toolkit, can use ``cms_plugin`` location for locating it's own plugins.

Paths and URLs
--------------

PathFinder operates with and distinguishes a physical locations versus
URLs. For example even through a physical file is located in
``vendor/atk4/atk4/public/atk4/js/file.js`` the relative URL would be
``public/atk4/js/file.js``. In some cases the URL would be absolute, for
example if you store your files on CDN:
``https://secure.agiletoolkit.org/js/file.js``.

You can use methods ``setBasePath`` and ``setBaseURL`` for
PathFinder\_Location to specify both.

When locating files, there are also two methods: ``locateURL`` and
``locatePath``.

AutoLoading classes
-------------------

Agile Toolkit Pathfinder registers two loader functions. The first
function will take precedence and will be used to locate include files
before Composer.

-  Agile Toolkit PathFinder autoloader: will take advantage of location
   definition to locate your class.
-  Composer's autoloader: will be used to load non-agile toolkit
   extensions
-  Agile Toolkit fallback: this loader will simply display error and
   list attempted locations.

This loader order helps you understand which file is missing and where
it was requested.

Default Locations
-----------------

Agile Toolkit defines three locations for you:

::

    $this->api->pathfinder->base_location

    $this->api->pathfinder->public_location

    $this->api->pathfinder->atk_location

You can either add more locations or define more contents of the
existing locations. For example, let's add additional CSS folder:

::

    $this->api->pathfinder->base_location->defineContents(array(
        'css'=>'public/css/'.$skin;
    ));

Next time the CSS file would be located in the following folders and in
the following order: ``public/css``, ``public/css/myskin``,
``public/atk4/css``. The physical file is located first, then the first
matching location will be used to generate URL or Path.

Relative Locations
------------------

If you place a certain location inside another location, you can use
method ``addRelativeLocation``. This will re-use the URL and Path of the
parent location and apply it to your new location. For example you will
find that on some projects you want to create ``shared`` folder which
contains the resources you share between different applications within
your project. Here is how you can do it:

::

    $this->api->pathfinder->base_location->addRelativeLocation(
        'shared', array(
            'php'=>'lib',
            'template'=>'templates',
        )
    );

If you are building an "admin" system located under a sub-folder but you
still want to access some of the classes from your frontend, you can use
the following inside your admin:

::

    $this->api->pathfinder->base_location->addRelativeLocation(
        '..', array(
            'php'=>'lib',
            'mail'=>'templates/mail',
        )
    );

Bundling locations inside add-ons
---------------------------------

Finally if you are building an add-on, you can add locations from within
the add-on:

::

    $this->api->addAddonLocation(__NAMESPACE__, array(
        'cms_plugins'=>'cms_plugins',
        'css'=>'public/css'
    ));

There are two aspects of add-on installation you might need at this
point (for more info read about automated add-on installation)

-  create class ``myaddon\Controler extends \Abstract_Controller``, then
   use $api->add('myaddon/Controller'); to activate your add-on. Define
   location inside this class.
-  create ``public`` subfolder inside your add-on. it will be sym-linked
   as ``public/__NAMESPACE`` by installer. ``addAddonLocation`` will
   automatically replace ``public/css`` with
   ``public/__NAMESPACE__/css`` for you if necessary.

Isolated installations
----------------------

Historically Agile Toolkit have been operating in two modes - in first
you install EVERYTHING into web-root. In other set-up you point your
web-root inside ``public`` folder.

From 4.3 your setup will automatically be detected and locations will
configure themselves appropriately, however the secure install with
``public`` folder isolated is default option now for new installation.
