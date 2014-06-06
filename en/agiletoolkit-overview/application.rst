Application Class
##################

As mentioned already, application class will be created first and will
have all the other objects added into it. Depending on the type of
your application (Web, CLI, RestAPI) you would have to use different
class for your application object.

Any object of Agile Toolkit will have a reference to the application class::

    $this->app

Understanding Application Classes
=================================

Agile Toolkit does not use any global variables and therefore you can have
multiple applicaiton classes. This can be used for testing, debugging,
sandboxing or other advanced patterns. Most importantly, however, is that
you keep it in mind, that there may be multiple application classes,
so don't go outside of your application.


Execution of a Web Request
==========================

An API in Agile Toolkit splits the whole execution of the application into two
pahes. In the first part, Application initializes the objects, coupling them with
proper models. The second part is the execution: the rendering will ask
objects to query the database, iterate through results, produce output.

Agile Toolkit objects never ``echo`` anything directly. Instead, they insert
their :php:meth:`AbstractView::output` into their parent's template. The
Application sends the combined output to the user's browser through "echo"
after everything is initalized and rendered.

.. figure:: ./apiseq.png

  Application Execution Sequence Schematic


.. warning:: This image is outdated and needs redrawing.





Initialization
Depending on which API you use, it will initialize several sub-components which call Controllers. When you extend an API class, you should re-define the init() function and add initialization in there - things like connecting to the database, adding controllers, etc.

ApiFrontend will automatically initialize Logger() which makes error logging more bearable.

Layouts
A Layout is a region in the shared template (shared.html), which is parsed by the API class. If the region is defined, then a corresponding layout function is called.

The first and most significant layout element is "Content". Content will determine which page is requested and proceed with all the logic of page initialization. However if "Content" is not present in the API template, the page will not be initialized.

You can add additional layout elements. For example, calling addLayout('Menu') inside api->init() will instruct API to call layout_Menu() method if tag <?Menu?> is found when parsing the template.

This approach can be used to initialize various things such as sidebars, toolbars and widgets, if they exist inside the API template. You can manipulate different templates from inside the defaultTemplate() function of your API.

