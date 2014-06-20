**************************
Pattern Routing Contnoller
**************************

.. php:class:: Controller_PatternRouter

What Is Routing?
----------------

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

Introduction to Routing
-----------------------

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



.. todo:: integrate roman's route docs
