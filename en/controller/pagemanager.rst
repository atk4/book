URL Parsing
===========

.. php:class:: PageManager

Base Path and URL of your Web App
---------------------------------

Not always you will install your web application in the root of your
site. Sometimes it may be located in a sub-folder. Agile Toolkit does a
great job at determining where you have installed your application and
automatically splitting path properly:

::

    http://example.org/v3/admin/user/edit

From the URL alone it's impossible to determine that ``/v3/admin/`` is
the Base Path of your application. Agile Toolkit solves this problem by
looking at the folder where that ``index.php`` is located from
``$_SERVER['SCRIPT_NAME']``.

This is variable is saved and available in ``$api->pm->base_path``.

This determines a full URL to your web application, such as
``http://example.org``. This will also contain port number.

If any of the parameters are determined incorrectly in your environment,
you can either fix them inside your Application::init() or simply use a
different PageManager class (by setting Application::pagemanager\_class
property).

Determining Page
----------------

Everything what comes after the Base Path is considered to be page.
Because class names may only contain underscores and may not contain
slashes - all underscores in your pages will be converted into slashes.

::

    http://example.org/admin/user/edit.html
      .. same as ..
    http://example.org/admin/user_edit

You must keep this in mind, if you access ``$api->page``, that it will
contain underscores rather than slashes.

If you wish to manually override page, you can pass argument
``?page=abc123`` and it will be used as a page value instead. This is
what is used by default when you just install Agile Toolkit on a site
without mod\_rewrite.

Debugging
---------

If you suspect that Agile Toolkit is not detecting your base parameters
properly, you might want to turn on debugging for Page Manager object by
adding this into your application:

::

        protected $pagemanager_options=array('debug'=>true);

This will display all the parameters on page start-up.

Agile Toolkit Assets
--------------------

Application location is vital to determine where your ``atk4`` is
located. By default Agile Toolkit will assume that Agile Toolkit assets
must be prepended with ``./atk4/``. This works for default installation
without problems, but if you are using SEF URLs, you might need to be
more specific about location of Agile Toolkit public folder.

::

    $config['atk']['base_path']='/mysite/atk4/';

PathFinder integration
----------------------

PathFinder[todo: link] is class for helping asset management, such as
images, CSS and other files. PathFinder relies on ``base_path`` properly
to build absolute URLs. This is how PathFinder learns about the base
path (from lib/PathFinder.php):

::

    $this->base_location->setBaseURL($this->api->pm->base_path);

-  `Read about Asset Routing <assets.md>`__

File Extensions and Examples
----------------------------

You can use any extension at the end of the page. Agile Toolkit will
ignore everything it encounters after the dot. For the same reason dots
are not allowed to be passed through ?page= parameter.

Here are some of the examples

+-----------------------------+------------------------+-------------------------------------------------------------------------------------+
| Browser URL                 | Page                   | Notes                                                                               |
+=============================+========================+=====================================================================================+
| /preferences.html           | preferences            | Agile Toolkit completely ignores the extension and uses                             |
|                             |                        | remaining location to determine page name.                                          |
+-----------------------------+------------------------+-------------------------------------------------------------------------------------+
| /?page=user/add             | user_add               | Your default install of Agile Toolkit is not configured to                          |
|                             |                        | use mod_rewrite. Therefore the URL in the browser will address                      |
|                             |                        | index.php passing page=XX. GET['page'] will always override determined page-name    |
+-----------------------------+------------------------+-------------------------------------------------------------------------------------+
| /profile/change-password.do | profile_changepassword | Dashes cannot be used in a function or class,                                       |
|                             |                        | they are eliminated from the page name automaically                                 |
|                             |                        | Any extensiocan can be used as long as .htaccess                                    |
|                             |                        | directs them to index.php                                                           |
+-----------------------------+------------------------+-------------------------------------------------------------------------------------+
| /?abc=123                   | index                  | If URL does not contain page, then "index" page name is used.                       |
+-----------------------------+------------------------+-------------------------------------------------------------------------------------+
| /admin/logout               | logout                 | Agile Toolkit does not have to be in your                                           |
|                             |                        | web-root directory.                                                                 |
|                             |                        | If it's installed into subdirectory, Agile Toolkit will detect                      |
|                             |                        | it and will eliminate the installation point (base_path) from the name of the page. |
+-----------------------------+------------------------+-------------------------------------------------------------------------------------+

