Customizing Look
================

This chapter explains a best practices and different strategies on customizing
look and feel of your Agile Toolkit application. It's important that you understand how
:doc:`templates </views/templates>` and :doc:`view rendering </views/core>` work in general it is more important that
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
    footers, icons and responsiveness. This is further documented in :php:Class:`Layout`
    class documentation.

Use pages with custom template
    Customizing individual look on the page will make your information well-prented,
    more usable and nicely structured. See below for a simple guide how to
    customize template.

Creating custom views with your own template.
    While the fastest way to display information is with :php:class:`Grid` and
    :php:class:`ModelDetails`, quite often you need to obide by a custom presentation
    of data such as lists, view pannels and so on. Those views require creation
    of custom views with custom templates. See below for a simple guide.

Assigning CSS classes to existing Views.
    :doc:`/css` supports a wide varietty of :doc:`/css/components` which can
    change the way how views are presented using :ref:`Component Definition Array`

Use special outline Views and composition.
    Some of the existing views such as :php:class:`Columns` allow you to outline
    your content in a certain way. :php:class:`Form` also support the way to format them
    (:doc:`views/form`). Other classes may also offer some several ways to present them.


Please be familiar with all of the appraches listed above. Sometimes it's more
desirable to use one way over another. For example instead of using Columns
and H2-like tags in succession, you should rather use a view with custom template.

Guide to customizing template
-----------------------------

First you need to decide if you want your changes to affect all object of desired
type in the application or only specific object. For example, if you wish to change
the way how :php:class:`Grid` looks, you may want to only one grid:

- Only affect single object occurance
- Create a new PHP Cass with the new look
- Replace look for a standard class
- Replace look and behaviour for standard class

Copy ``atk4/template/grid.html`` into ``template/mygrid.html``

For a single occurence, you should use 4th argument to :php:meth:`AbstractObject::add`
object ``[ 'mygrid' ]``

To affect multiple objects, extend ``Grid`` and override :php:meth:`AbstractView::defaultTemplate`::

    function defaultTemplate() {
        return [ 'mygrid' ];
    }

If you are willing to affect all ``Grid``s, name your templtae ``template/grid``.
This will take precedence over atk4 template folder.


Page templates
--------------

A most common object to have template replaced is a Page. It is in fact so common, that
the page will actually look for a custom template.

#. URL request to /item/list
#. If ``Application::page_item_list(Page $p)`` exist, it's called.
#. If file page/item/list.php exists and contains page_item_list class, it's used.
#. Otherwise blank :php:class:`Page` is used.

Regardless of which of the above was used, :php:class:`Page` will attempt to load
``template/page/item/list.html``.
