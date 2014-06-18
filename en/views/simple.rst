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

**********************
Buttons and ButtonSets
**********************

Agile Toolkit since 4.3 no longer uses jQuery UI buttons. Instead the ``atk-button``
is used for a ``<div>``. (See :doc:`/css/buttons`)

Button directly extends :php:class:`View` and retains it's methods such as
:php:meth:`View::addClass` or :php:meth:`View::addComponents`.

Button also inherits a JavaScipt binding techniques: :php:meth:`AbstractView::js`
and :php:meth:`AbstractView::on` for custom actions.

.. php:class:: View_Button

    Implements a UI Button.

.. php:method:: link($page)

    Uses ``<a>`` markup for a button and creates a non-javascript link to
    a page. Supports :ref:`URL Array hash`.


.. php:method:: addPopOver

    Adds pop-over to a button, a sort of fall-out menu. Returns
    :php:class:`Popover` object.

.. php:method:: setIcon

    Adds icon inside a button.

.. php:method:: addSplitButton

    Splits button into 2 parts. Visually this appears as two buttons, one
    which would perform an action, and other one wich would display some
    additional options.

    WARNING: migth be refactored into a separate class.

.. php:method:: addMenu

    Similar to pop-over but will link button with vertical menu. Returns
    :php:class:`Menu_Vertical`.

.. php:method:: isClicked

    Creates a binding on a button which will call back to original page.
    The method will return false, initially, but when a callback is used,
    will return true.

    You must execute a :ref:`JavaScript chain` at the end of your code.

.. php:method:: onClick($callback)

    Similar to isClicked, but will execute a callback code. You can
    return a :ref:`JavaScript Chais`, but if you don't a default JS
    action will be executed saying "Success"

    A second argument to this method could be a confirmation message
    which will appear before callback being triggered.


.. php:class:: View_ButtonSet

    This class is a container for buttons, so that they appear visually grouped.

.. php:method:: addButton()

    Adds a new button into set.
