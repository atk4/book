Customizing Look
================

This chapter explains a best practices and different strategies on customizing
look and feel of your Agile Toolkit application. It's important that you understand how
:doc:`templates </views/templates>` and :doc:`view rendering </views/views work in general it is more important that
you use apply your knowledge correctly.


Getting Started
---------------

Agile Toolkit application you have just started comes with default set of
templates. There exists a template for any view element (except AbstractView).

.. note:: The HTML templates use :doc:`/css` for formatting output. While it is possible
    to use a different CSS framework, you would need to supply HTML code for
    views yourself. This documentation will not cover that area.

To customize Look of Agile Toolkit you can use several strategies:

Create a custom :doc:`/css/theme`
    Will affect overall look and feel of your application, may affect color scheme,
    spacing between elements and other visual enhancements such as shadows, borders,
    responsiveness.

Use different or your own Layouts
    The layout will affect overal element positioning on the page - menus, headers,
    footers, icons and responsiveness.

Use pages with custom template
    Customizing individual look on the page will make your information well-prented,
    more usable and nicely structured.

Creating custom views with your own template.
    While the fastest way to display information is with :php:class:`Grid` and
    :php:class:`ModelDetails`, quite often you need to obide by a custom presentation
    of data such as lists, view pannels and so on. Those views require creation
    of custom views with custom templates.

Assigning CSS classes to existing Views.
    :doc:`/css` supports a wide varietty of :doc:`/css/components` which can
    change the way how views are presented using :ref:`Component Definition Array`

- Use special outline Views and composition.
