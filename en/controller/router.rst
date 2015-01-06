**************************
Pattern Routing Controller
**************************

.. php:class:: Controller_PatternRouter

.. _routing:

Request Routing
===============

Routing is the process by which Agile Toolkit determines which code to
run in response to a request.

Requests are passed to the webserver from the browser in the form of a
URL. Each URL should define a unique resource for your app to return to
the requester. It is considered a proper design if URL alone can fully
determine user's desired action - avoid storing some routing logic in
Session data or POST data. Here is a typical URL::

    http://example.org/admin/user/edit?user_id=132

Routing in Agile Toolkit is a simple yet sophisticated method and can be
easily adopted to handle URLs like this::

    http://en.example.com/about.me
    http://johndoe.mysites.com/
    http://listings.com/houses/riverside-lake-3-bedroom/apointments/6545/edit

Agile Toolkit will parse your URL on request and connect it to the
proper code. It will also generate URLs for you. Select sub-topic or
continue reading for introduction.

-  `Interpretation and parsing of URLs <routing/parsing.md>`__
-  `Generating URLs and redirecting, relative URLs <routing/url.md>`__
-  `Sticky GET arguments <routing/sticky.md>`__
-  `URLs to your assets: images, css, js <routing/assets.md>`__
-  `Using PatternRouter for SEF (search engine friendly)
   routing <routing/patternrouter.md>`__
-  `Application - dependent routing - ApiFrontend, ApiWeb, ApiInstall,
   ApiAdmin <routing/application.md>`__

Routing in Agile Toolkit
------------------------

Agile Toolkit is following Model View Presenter. Presenter contains
presentation logic. To keep things simple we call it "Page". In other
words, "Page" object is responsible of initializing all other elements
you have on your page.

*Routing is the process which determines which Page class should be
loaded and used.*

.. figure:: /figures/routing-flow.png

1. PageManager is a standard class designed to break down your URL into
   essential components (explained below in more details)
2. PatternRouter is an **optional** step which can convert "beautiful"
   or SEF URLS into normal form.
3. Application routes **page** to a class or a method
4. Page object continues to initialize objects.

Essential parts of every URL
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. figure:: /figures/url-components.png

All request to Agile Toolkit must be translated into four essential
parts as displayed on the graph to the right.

If the URL is search-engine friendly and some arguments are hidden in
the "page" part, they need to be extracted by PatternRouter. Note that
you may use ``Controller_PatternRouter`` or you may write your own
extraction logic, but this transformation must take place inside your
``Application::init()``.

You should also note how slashes are distributed. ``base_url`` has no
training slash, while ``base_path`` always begins and ends with a slash.

.htaccess
~~~~~~~~~

It does not matter how exactly you route your request - as long as it
reaches the Application class (which is typically through index.php),
that should be acceptable. Below is a sample ``.htaccess`` file which
you can use in your installations::

    RewriteEngine On
    RewriteRule ^[^\.]*$            index.php   [L]

If you are running NGINX, here is sample configuration:

::

    location ~ ^[^.]*$ {
      include /etc/nginx/fastcgi_params;      ##Includes our fastcgi setup
      fastcgi_param SCRIPT_NAME     "/index.php";
      fastcgi_param SCRIPT_FILENAME "/www/path/admin/index.php";
    }


SEF Link routing with PatternRouter
===================================

Agile Toolkit comes with a simple two-way routing controller called
Controller\_PatternRouter.

::

    $api->add('Controller_PatternRouter')
        ->link('user/edit/:id')
        ->link('browse/:category/:action', 'browse/:action')
        ->route();


Pattern Router works according to a simple principle of dissembling
"page" into GET parameters and then reassembling. When URL are
generated, it simply reverses the arguments to reconstruct the original
URL.

If the above example is accessed using URL: ``browse/furniture/share``,
the numeric arguments will be moved to
:math:`_GET['category'] and `\ \_GET['action'] respectively. Without
second argument request would be routed to the ``browse`` page, however
in it's presence the URL is reconstructed back to ``browse/share`` and
the category is passed as a get argument.

Note, that if for some reason you'll receive request like this::

    browse/share?category=furniture

then it will go through without re-routing and will access the same
page.

When you use ``url()`` the reverse process applies and the URL is
matched against second arguments (or if it is omitted then against first
argument with any dynamic sections stripped out)::

    url('browse/details',array('category'=>'fashion'));

Would produce URL::

    browse/fashion/details

Transparency is the most important feature of pattern routing in Agile
Toolkit - as soon as you add a new route, application will begin using
it through URL.

Consuming multiple parts of the page
------------------------------------

A regular route breaks down your request into parts, such as
foo/bar/baz. Regardless of what variables you use, you must supply 3
parts separated with slashes. Sometimes you would want to match variable
amount of sub-pages::

    ->link('doc/::rest')

In this case all sub-pages of doc/ will be matched and remainder of the
page will be placed inside the ``::rest``, e.g.:
``doc/application/routing/url``

Regular Expressions
-------------------

You can also match certain arguments against regular expressions. The
format is the following:

::

    ->link('browse/:id~[0-9]/:action~[a-z]')

This will instruct pattern router to patch id against numeric patterns
and action against alphabetic sequences. Regular expression must always
fully match (from ^ to $).

You can't match part of the section against a variable otherwise that
would break the reversing ability.

Types
-----

For convenience you can use a simplified form on some of the regular
expressions::

    ->link('browse/:id=int/:action=az')

Which is equal to the same pattern as above.

calling route() and url()
-------------------------

If you call route() without any arguments, then PatternRouter will
re-write ``$api->page`` and ``$_GET``. You can however specify two
arguments - page and arguments. If you do that, then route() will return
new page and will modify the array to contain new argument set.

Method ``url()`` performs the same but other way around.

Conditional Routing
-------------------

When calling ``link()``, the third argument can be specified as routing
condition callback. This function will be executed and only if it
returns "true" in additional to regular matching, the substitution will
take place. This allows you to use really interesting approaches, where
you would route request to one of the several pages depending on
conditions.

Compatible formats
~~~~~~~~~~~~~~~~~~

Before we arrived at the above routing, several other formats were used
with PatternRouter. Those are obsolete and should be avoided:

::

    ->link('user/edit',array('id'))  // OBSOLETE

This line would be identical to route user/edit/:id. This format have
several disadvantages, e.g. it could not pick arguments from within the
URL.

Tutorial: Beautiful blog URL matching
-------------------------------------

Would you want to match article URLs without impact on the rest of the
pages?

::

    http://example.org/503-my-article-title

This tutorial will also explain to you how to generate URLs containing
combined values.

Advanced Tutorial: Building your own router
-------------------------------------------

Find out how you could develop your own two-way routing controller
working entirely based on your desired logic.




.. todo:: implement in code!!
