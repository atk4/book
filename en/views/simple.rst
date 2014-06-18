*******************
Simple Usable Views
*******************

Agile Toolkit has many objects which you as developer can use out of the box.
This chapter will focus on listing the most simple views.

Headers
=======

The following classes can be used to easily emulate markup of headings. All of
those objects are extended from :php:class:`View`, so you should remember
to call :php:meth:`View::set`.


.. php:class:: H1

    Creates ``H1`` header object

.. php:class:: H2

    Creates ``H2`` header object


.. todo: add more.


Boxes
=====

Box represents an outlined text.

.. php:class:: View_Box

    Outlined text

.. php:class:: View_Info

    Info box

.. php:class:: View_Warning

    Warning box

.. php:class:: View_Error

    Error box


.. php:class:: View_Hint

    Box containing a hint

Icons
=====


.. php:class:: View_Icon

    Will display icon without text. Use set() to set name of the icon. See http://css.agiletoolkit.org/ for a list of usable icons.



