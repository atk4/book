Menu
====

Agile Toolkit has several menu implementations currently bundled.

- Menu - the preferred menu to use.
- Menu_Objective - relies on ul/li objects
- Menu_Basic - backwards compatible menu

Menu is also backwards compatible with Menu_Basic which was used before 4.3. The
main reason for new menu was a more powerful HTML features for :doc:`/css/menu`.

Menu_Objective operates without templates and relies on Views for an elements.

I advise you to use :php:class:`Menu` when in doubt and only look into other
ones if you are super confident and you need a custom markup for your code.

It's also possible that Menu_Basic will be removed from Agile Toolkit into
an add-on in the next version.

.. php:class:: Menu_Vertical

    See: Menu_Advanced

.. php:class:: Menu_Horizontal

    See: Menu_Advanced

.. php:class:: Menu

    See: Menu_Advanced

Class Hierarchy
---------------

* Menu_Advanced (extends View)

  - Menu_Horizontal

    + Menu

  - Menu_Vertical

It's better if you use Menu_Vertical or Menu_horizontal directly. While
both classes are mostly identical, the only thing different is a template.

This documentation will refer to the menus as :php:class:`Menu_Advanced`.

.. php:class:: Menu_Advanced

    Main Agile Toolkit Menu implementaiton.

Adding Menu Items
-----------------

You can call the following methods to populate the menu:

.. php:meth:: addTitle

    Adds menu title

.. php:meth:: addItem

    Add menu item

.. php:meth:: addMenu

    Adds submenu

.. php:add:: addSeparator

    Adds separator

All of the above are quite similar as they both create a new View and return it.
addSeparator does not take any arguments.

Other menthods accept first argument as a label text. This argument supports
:ref:`Component Definition Array` format, enabling you to pass ``icon`` for instancee.

addItem() extends this format to also include ``icon2`` (which will be placed)
on the right and ``badge``, which is also placed on the right (but can't be used
together with icon2). addMenu and addTitle does not support icon or badge, but
you can still use ``icon``.

addItem can have second argument - page. This argument can also be a hash (see
:ref:`URL array hash`).

addMenu second arguent can be either 'Vertical' or 'Horizontal' and defaults
to Horizontal.

Next is an example::


    $this->menu = $this->add('Menu_Vertical');
    $this->menu->addItem(['Dashboard', 'icon'=>'gauge'], 'index');

    $m=$this->menu->addMenu(['Customers', 'icon'=>'smile']);
    $m->addItem(['Users', 'icon'=>'users'], 'users');
    $m->addItem(['Purchases', 'icon'=>'money'], 'purchases');
    $m->addItem(['Subscribers', 'icon'=>'chart-line'], 'subscribers');
    $m->addItem(['Plans', 'icon'=>'basket'], 'plans');

    $m=$this->menu->addMenu(['Installation', 'icon'=>'network']);
    $m->addItem('ATK Installs', 'installations');

Using Menu with Model
---------------------

.. php:method:: setModel

    Associate Menu with Model

Menu object supports use of Model. It will then iterate throug model :ref:`data set`
and initialize menu items. More importantly - Menu support :ref:`Hierarchy Models`
and will automatically create sub-menus as necessary.

The model must have the following fields:

- name - will be used as menu label
- link - destination URL for the page
- icon - if specified, will show icon
