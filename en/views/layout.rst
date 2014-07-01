**************
Global Layouts
**************

Starting from 4.3, Agile Toolkit have broken down it's global HTML template
(``shared.html`` previously) in two components:

- Boilerplate HTML template located in html.jade (html.html)
- Layout view


.. _boilerplate_html:

Boilerplate HTML
================

A ``html.jade`` file is designed to contain a recommended values for a
HTML5 application. It's unlikely you would ever need to change it ever.


Layouts
=======

This is a type of View which you either use out-of-the-box or create your own.
Layout represents a certain composition of views on the page, which your
application will fill out with dynamic elements.

You can use different views on different pages or switch it depending on
user authentication status.

Next is a layout of a typical :php:class:`App_Admin` application. For
this api the layout will automatically be initilaized and available
at ``$app->layout``. If you wish to use this layout inside other app class,
you would need to add it inside your Frontend::init() method.

Layout_Fluid
------------

.. figure:: /figures/layouts.png

Layout define various regions inside a full-screen layout of the page.

.. php:class:: Layout_Fluid

    Implements a fluid HTML layout.

.. php:method:: addMenu

    Adds and returns Main_Menu object.

.. php:method:: addFooter

    Adds a view allowing you to customize footer.

.. php:method:: addHeader


.. note:: This class API is not fully stable yet and may be further refactored in 4.3.x


Layout_Centred
--------------

.. figure:: /figures/layout-centered.png

.. php:class:: Layout_Centred

    Centered layout is a very basic layout which squeezes your content inside
    a box centered in the middle of the screen.

This layout is used by default on your Frontend page. If you just started with
Agile Toolkit frontend development, try changing this to Layout_Fluid
