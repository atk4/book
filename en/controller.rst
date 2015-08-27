Controllers
###########

In Agile Toolkit controllers are generic. They resemble "helpers" or
"libraries" in other frameworks. Like any other object, you need to "add"
your controller. Most controllers are designed to enhance the functionality of
the object, where you add them.

Some controllers would have ``Controller_`` in their class name, others
are used without prefix. Controllers introduce diversity and their impact
may be very different. For instance :php:class:`Controller_Compat` will
set Agile Toolkit into compatibility mode with previous version, while
:php:class:`Order` will simply allow you to reorder items in array.


Ordering Controller
===================

Although not very significant, Ordering Controller is a good demonstration
of controller use in Agile Toolkit.

.. php:class:: Order

    Allows to re-order child elements.

Example::

    $this->add('Text')->set('World! ');
    $o2 = $this->add('Text')->set('Hello, ');
    // Would output - "World! Hello, "

    $this->add('Order')->move($o2, 'first')->now();
    // Now order is fixed. - "Hello, World!"

.. php:method:: move

    Schedule item to be moved. Must be either key or descendant
    of :php:class:`AbstractObject`.

Method move has various formats::

    $o = $form->add('Order');

    $o->move('field', 'first');   // will be first
    $o->move('field', 'last');   // will be first
    $o->move('field', 'before', 'otherfield');   // will be first
    $o->move('field', 'after', 'otherfield');   // will be first
    $o->move('field', 'middle');   // puts in the middle
    $o->move('field', 'middleof', 'Form_Field');   // will be first

:php:class:`Form` in Agile contains many different child objects -
model, controller, helpers etc. When you want your field to be
positioned in the middle of other fields, you would not care
for those invisible objects.

The ``middleof`` setting will instruct Order controller to place your
field in the middle relative to :php:class:`Form_Field` class objects.

.. php:method:: now

    Will re-order items right away.

.. php:method:: later

    Will re-order item before render. (Application beforeRender hook)

.. php:method:: useArray

    By default, object will operate on it's owner's
    :php:attr:`AbstractObject::$elements`. You can specify a different
    array to use here.


====================================

.. toctree::
    :maxdepth: 1

    controller/pagemanager
    controller/router
    controller/pathfinder
    controller/logger
    controller/validator
    controller/tmail
    controller/data
    controller/mvc
    controller/auth
    controller/db
    controller/migrator
    controller/storage
    controller/url
    controller/process
    controller/dummy


