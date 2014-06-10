Addons
======


Add-on in Agile Toolkit is a collection of PHP class files or some
additional resources located in a folder and sharing a namespace.

Add-on can be installed through Composer or some proprietary add-ons may
be distributed differently. Add-ons can also be added directly from
github, etc.

Agile Toolkit will continue to work OK without Agile Toolkit, while your
project may depend on one or several add-ons for it's functionality.

Any package from packagist.org are valid add-ons fro Agile Toolkit.
Using composer add-ons with Agile Toolkit is like with any other
project.

What is Agile Toolkit Add-on
----------------------------

An Agile Toolkit Add-on relies on the ATK framework and may use some of
the following features:

-  Initiator - enables automatic inclusion and initialization of add-on
   in the project.
-  PathFinder - add-on can cary additional file types and can configure
   PathFinder to locate them (templates, scripts, images)
-  Public folder - add-on may contain a public folder, which will be
   made available for direct access by the framework.
-  Database migrations - add-ons may rely on database and provide
   migration scripts.


Types of add-ons
^^^^^^^^^^^^^^^^

.. todo:: Mayur - please clarify addon types:

Open-source PHP class dsitribution usually focus on providing you with
decoupled class implementation with dependencies. Agile Toolkit
add-ons assume that you have Agile Toolkit installed and working. This
allows add-on developers to deploy a much more higher-level add-ons:



Installation of Add-ons
^^^^^^^^^^^^^^^^^^^^^^^

Agile Toolkit allows developer to easily use UI and install any
packagist add-on. Those add-ons do not require any additional actions
and they will be available for us throughout the project.

Installation of Agile Toolkit Add-ons
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Addons specifically developed for Agile Toolkit, on other hand, can be
installed first and then initiated. Application can initialize those
addons automatically.

While we advise to distribute Agile Toolkit Add-ons using packagist
repository, they can be distributed by other means.

Reserved namespace - "atk"
^^^^^^^^^^^^^^^^^^^^^^^^^^

The following namespaces are reselved and must not be used by your add-ons:
atk, atk4, form, model, field, view.


Marketplace
-----------

AgileToolkit.org lists all Agile Toolkit addons and offers a rich
catalogue of addons, descriptions, reviews, ratings and thumbnails.
Market also supports paid add-ons or can limit add-on to only be
available for commercial users.

Any developer may publish their add-on in the market either for free
or under paid distributions.

Installing Add-ons
------------------

After you install your Agile Toolkit and complete installation wizard
you'll have a Frontend and Admin sections. Installer will also register
your installation and link it up with your agiletoolkit.org account.

If you have purchased any add-ons in the past, they will be available
for your installations. Some addons may only be available for paid
developer accounts.

If Add-on developers have decided to distirbute their add-on under MIT
license, those add-ons will be available and can be used at no cost,
however such add-ons would be maintained by community and may not have
commercial support available.

Packagist/Composer Add-ons
^^^^^^^^^^^^^^^^^^^^^^^^^^

Agile Toolkit is compatible with composer add-ons. Follow instruction on
packagist.org to update your composer.json file and to install add-on.


Creating an add-on
------------------

You can and should use add-ons inside your project. To create one,
create new folder in shared/addons/yourname/addonname/ and place files
there. Make the path correspond with the namespace and PathFinder will
be able to initialize your add-on directly.

.. todo:: Provide a more detailed guide or tutorial.

When your add-on is ready to be distributed, commit it into it's own
repository, create ``composer.json`` inside that folder and register it
with packagist.org. Now you can install your add-on using composer.

Composer are installed inside vendor/yourname/addonname/ but the add-on
should work regardless of where it is installed.

Because you're building new add-on, you might need to create a "public"
link for your add-on: go to frontend/public/ and create symlink called
"yourname\_addonname" pointing to
``../../shared/addons/yourname/addonname/public``. Remember that this
link will automatically be crated when anyone else is installing your
add-on through composer.

Using Add-ons.
--------------

When using generic add-ons you should use whatever instructions they
provide to invoke them. For Agile Toolkit add-on you should use
``$object->add()`` method to create add-on object. For example, your
Add-on might contain a class 'Model\_Article'. To use it you should do:

::

    $ob=$this->add('atk4/blog/Model_Article');

Please keep in mind that all the paths and class names are case
sensitive. If you are using assumes beginning of the class, you can
still use namespace:

::

    $this->setModel('atk4/blog/Article');

The same can be used with field types, grid column types etc.

Initiator
~~~~~~~~~

The class located in ``atk4/blog/lib/Initiator.php`` - if exists - will
be initialized automatically on every request to your Application. This
class must extend ``\Controller_Addon`` [todo - maybe rename to
something like AddonInitiator ? ]

If you define this class, you can describe your add-on in more details:

-  define add-on name.
-  specify if your add-on will have public content or not
-  specify if your add-on will have database migration
-  describe additional resources inside your add-on
-  page routing (e.g. all pages for blog/\* can be routed to the add-on)

TODO: explain all of the above add-on features with examples.

=================== REWRITTEN STUFF ==========================

-  ATK4 Addon is a Composer package. To become a composer package it
   needs to contain the file called composer.json located in root
   directory of addon and can be registered on packagist.org.
-  ATK4 addon doesn’t use composer autoload because ATK4 has its own
   autoload pathfinder.
-  To enable your addon to be installed from sandbox UI it needs to be
   registered in AgileToolkit Market. For manual installation you need
   to add required line to composer.json file of your project.

Autoload
--------

Each addon has its own namespace. This will help to avoid class name
duplication in addons from different vendors. Namespace structure is the
following: atk4, where

-  atk4 is a vendor name on github
-  blog is a repository name on github

If you have class Model\_Article in your addon you can load it in your
project like this:

::

    $this->add(‘atk4/blog/Model_Article’);

Initiator class
---------------

This class is used for initiation of your addon on every application
call and is not required. You can use this class to perform some actions
what your addon require. Your addon initiator must be inherited from
Controller\_Addon and must contain few fields

=========================== REWRITTEN STUFF ENDS ===========

::

    namespace rvadym\blog;
    class Initiator extends \Controller_Addon {

       // this var is used to store initiator object in $this->api->addons array
       // in our case this initiator will be accessible like this $this->api->addons[‘rvadym_blog’]
       // it is resposibility of addon creator to make accessible other part of addon from Initiator
        public $api_var        = 'rvadym_blog';


       // addon name to be shown in admin part if sandbox is installed
        public $addon_name     = 'Agile Toolkit Blog Addon';

       // types of private files what ATK4 will try to find in your addon
        public $addon_private_locations = array(
            'docs'      => 'docs',
            'php'       => 'lib',
            'page'      => 'page',
            'template'  => 'templates',
        );

       // types of public files what ATK4 will try to find in your addon
        public $addon_public_locations  = array(
            'js'     => 'js',
            'css'    => 'css',
        );

       // true is your addon contain some pages for Admin part
        public $with_pages = true;
    }

Controller\_addon::\ :math:`api_var This var is used to store initiator object in `\ this->api->addons array. In our case this initiator will be accessible like this

::

    $this->api->addons[‘atk4_blog’].

It is resposibility of addon creator to make accessible other part of
addon from Initiator.

Controller\_addon::$addon\_name
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

addon name to be shown in admin part if sandbox is installed

Controller\_addon::$atk\_version
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

suported ATK4 version

Controller\_addon::$namespace
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

namespace of this addon.

Controller\_addon::$addon\_obj
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

addon object from config

Controller\_addon::$addon\_private\_locations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

types of private files what ATK4 will try to find in your addon

Controller\_addon::$addon\_public\_locations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

types of public files what ATK4 will try to find in your addon

Controller\_addon::$with\_pages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

TRUE if your addon contain pages for Admin part.

Addon filesystem
----------------

Addon has almost same filesystem structure like regular atk4 project.It
can contain following folders inside of addon root folder:

-  lib - folder for all php classes based on atk4 AbstractObject.
-  templates - templates for addon pages and UI elements.
-  public - all files what should be visible from the web.

and following files:

-  composer.json

Addon can have these files inside its lib directory:

-  Initiator.php - class Initiator describes all actions what should be
   performed each time addon is called.

Addon filesystem structure
~~~~~~~~~~~~~~~~~~~~~~~~~~

::

          {your addon}↴
                      ↦ lib ———↴
                      |         ↦ Controller
                      |         ↦ Model
                      |         ↳ View
                      |
                      ↦ page
                      |
                      ↦ template↴
                      |          ↦ page
                      |          ↳ view
                      |
                      ↳public——↴
                                ↦ css
                                ↳ js

