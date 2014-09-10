**************************
Asset and Class Management
**************************

PathFinder
==========

.. php:class:: PathFinder

    PathFinder is responsible for locating resources in Agile
    Toolkit. One of the most significant principles it implements
    is ability for any resource (PHP, JS, HTML, IMG) to be
    located under several locations. Pathfinder will look for
    the location containing the resource you ask for and will
    provide you with either a relative path, URL or absolute
    path to a file.
    
    To make this possible, PathFinder relies on a class
    :php:class:`PathFinder_Location`, which describes each individual
    location and it's contents.
    
    You may add additional locations in your application,
    add-on or elsewhere. Some locations may be activated only
    in certain circumstances, for example add-on will be able
    to add it's location if add-on was added to a page.

Pathfinder is accessible inside your application through ``$this->api->pathfinder`` however some
methods are duplicated in the API class for easier access:

- :php:meth:`App_CLI::locate` - simply calls :php:meth:`PathFinder::locate`
- :php:meth:`App_CLI::locateURL` - calls :php:meth:`PathFinder::locate` with type='url'
- :php:meth:`App_CLI::locatePath` - calls :php:meth:`PathFinder::locate` with type='path'
- :php:meth:`App_CLI::addlocation` - calls :php:meth:`PathFinder::addLocation`

Here is an example how you can locate path of a certain mail template::

    $mail_template = $this->api->locate('mail','welcome.mail');
    // returns templates/mail/welcome.mail - relative path to your project root

Default Folder Structure for Agile Toolkit project
--------------------------------------------------

No doubt you are familiar with "INCLUDE\_PATH" which have been used in
PHP for locating include files. Pathfinder applies similar approach to
other type of resources. Below you will see the commonly used resource
types. You can easily extend by registering your own types of resources.

+----------+-------------------------+---------------------------------------------------------------------+
| Type     | Default Location        | Description                                                         |
+==========+=========================+=====================================================================+
| php      | lib                     | Contains class files from the default namespace. A file location is |
|          | ../shared/lib           | determined based on PSR-1 rules, however classes in those folders   |
|          | ../vendor/atk4/atk4/lib | would have no namespace.                                            |
+----------+-------------------------+---------------------------------------------------------------------+
| page     | page                    | Location for "page" files, which can be directly accessible through |
|          |                         | standard routing (See routing)                                      |
+----------+-------------------------+---------------------------------------------------------------------+
| template | template                | Contains Template files with extension .html. Agile Toolkit         |
|          | ../shared/tempalte      | templates actually originate from .jade files, but both the .jade   |
|          | ../shared/lib           | and resulting .html will be found nearby.                           |
+----------+-------------------------+---------------------------------------------------------------------+
| public   | public                  | Contains directly acessible resources such as images, fonts etc     |
+----------+-------------------------+---------------------------------------------------------------------+
| js       | public/js               | JavaScript include files. Use {js}myjs.js{/}                        |
|          | public/js/atk4/js       | or $this->js()->_load('myjs')                                       |
+----------+-------------------------+---------------------------------------------------------------------+
| css      | public/css              | Stylesheets, use {css}mycss.css{/} inside tempaltes.                |
|          | public/css/atk4/css     |                                                                     |
+----------+-------------------------+---------------------------------------------------------------------+
| mail     | template/mail           | TMail template location                                             |
+----------+-------------------------+---------------------------------------------------------------------+
| addons   | shared/addons           | Addons (namespaces), see description below                          |
|          | vendor/me/myaddon/...   |                                                                     |
+----------+-------------------------+---------------------------------------------------------------------+

.. php:method:: addDefaultLocations

    Agile Toolkit-based application comes with a predefined resource
    structure. For new users it's easier if they use a consistest structure,
    for example having all the PHP classes inside "lib" folder.
    
    A more advanced developer might be willing to add additional locations
    of resources to suit your own preferences. You might want to do
    this if you are integrating with your existing application or
    another framework or building multi-tiered project with extensive
    structure.
    
    To extend the default structure which this method defines - you should
    look into :php:class:`App_CLI::addDefaultLocations` and
    :php:class:`App_CLI::addSharedLocations`

Add-ons would typically be added during init() so their added locations would have low
precedence. In other words, if you would have a class, public file or js include
with the same name as in add-on, Agile Toolkit will automatically use your. For
this reason add-on authors are asked to use prefixes on their resources where possible.

As each location must know both physical path and URL, for custom
configuration you will need to define both - base path and base url.

Adding Locations
----------------

Each location in Agile Toolkit must be able to come up with a physical
path OR a URL to the location. PathFinder relies on
:php:class:`Controller_PageManager` and it's properties:

- :php:attr:`Controller_PageManager::base_url`
- :php:attr:`Controller_PageManager::base_path`

Location is defined as a separate object of class :php:class:`PathFinder_Location`.
To add a new location you must call addLocation or
:php:meth:`PathFinder_Location::addRelativeLocation`:

.. php:method:: addLocation

    Cretes new PathFinder_Location object and specifies it's contents.
    You can subsequentially add more contents by calling:
    :php:meth:`PathFinder_Location::defineContents`

Example::

    $my_location = $this->app->addLocation([
        'php' => 'my-lib',
        'template' = 'my-template'
    ]);


Paths, URLs and CDN
-------------------

PathFinder operates with and distinguishes a physical locations versus
URLs. For example even through a physical file is located in
``vendor/atk4/atk4/public/atk4/js/file.js`` the relative URL would be
``public/atk4/js/file.js``. In some cases the URL would be absolute, for
example if you store your files on CDN:
``https://cdn.agiletoolkit.org/js/file.js``.


You can asociate your location with Path and URL using
:php:meth:`PathFinder_Location::setBaseURL` and
:php:meth:`PathFinder_Location::setBasePath`. Alternatively you can
use :php:meth:`PathFinder_Location::setCDN` and

Locating Resources
------------------

.. php:method:: locate

    Search for a $filename inside multiple locations, associated with resource
    $type. By default will return relative path, but 3rd argument can
    change that.
    
    The third argument can also be 'location', in which case a :php:class:`PathFinder_Location`
    object will be returned.
    
    If file is not found anywhere, then :php:class:`Exception_PathFinder` is thrown
    unless you set $throws_exception to ``false``, and then method would return null.

.. php:method:: search

    Search is similar to locate, but will return array of all matching
    files.

.. php:method:: searchDir

    Specify type and directory and it will return array of all files
    of a matching type inside that directory. This will work even
    if specified directory exists inside multiple locations.

Using searchDir is very handy if you want user to select an appropriate
controller::

    $form = $this->add('Form');

    $form->addField('DropDown','use_menu_type')
        ->setValueList($this->app->pathfinder->searchDir('php', 'Menu'));


    // Will list classes like:
    //   Menu_Basic
    //   Menu_Vertical
    //   Menu_Horizontal
    //   etc



AutoLoading classes
-------------------

.. php:method:: loadClass

    Provided with a class name, this will attempt to
    find and load it

Agile Toolkit Pathfinder registers two loader functions. The first
function will take precedence and will be used to locate include files
before Composer. Here is the typical call order:

-  Agile Toolkit PathFinder autoloader: will take advantage of location
   definition to locate your class.
-  Composer's autoloader: will be used to load non-agile toolkit
   extensions
-  Agile Toolkit fallback: this loader will simply display error and
   list attempted locations.

This loader order helps you understand which file is missing and where
it was requested.

Properties
----------

PathFinder object instance is accessible through ``app->pathfinder`` and
contains references to some useful locations.

.. php:attr:: base_location

    Base location is where your interface files are located. Normally
    this location is added first and all the requests are checked here
    before elsewhere. Example: /my/path/agiletoolkit/admin/

.. php:attr:: public_location

    This is location where images, javascript files and some other
    public resources are located. Ex: /my/path/agiletoolkit/public

.. php:attr:: atk_location

    Agile Toolkit comes with some assets: lib, template. This
    location describes those resources. It's not publicly available

.. php:attr:: atk_public

    There are also some public files in ATK folder. Normally
    this folder would be symlinked like this:
    
    public/atk4  -> /vendor/atk4/atk4/public/atk4
    
    If that folder is not there, PathFinder will point directly
    to vendor folder (such as if on development environment),
    if that is also unavailable, this can fall back to Agile Toolkit CDN.

PathFinder Location
===================

.. php:class:: PathFinder_Location

    Represents a location, which contains number of sub-locations. Each
    of which may contain certain type of data

Relative Locations
------------------

.. php:method:: addRelativeLocation

    Adds a new location object which is relative to $this location.

If you place a certain location inside another location, you can use
method ``addRelativeLocation``. This will re-use the URL and Path of the
parent location and apply it to your new location. For example you will
find that on some projects you want to create ``shared`` folder which
contains the resources you share between different applications within
your project. Here is how you can do it::

    $this->api->pathfinder->base_location->addRelativeLocation(
        'shared', array(
            'php'=>'lib',
            'template'=>'templates',
        )
    );

If you are building an "admin" system located under a sub-folder but you
still want to access some of the classes from your frontend, you can use
the following inside your admin::

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
