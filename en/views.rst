User Interface
##############

Behind the simple facade of the PHP objects, Agile Toolkit hides a flexible
and robust implementation of User Interface based on best web development
practices.

If you have only experienced Agile Toolkit (older versions) for the purposes
of admin system, you will be thrilled to learn that Agile Toolkit is well
suited for the most demanding UI practices.

Consider the following code on your page::

    $menu = $this->add('Menu_Vertical')->addClass('atk-col-2');
    $menu->addTitle('Customer Interaction');

    $m_users = $menu->addMenu(['Customers', 'icon'=>'smile']);

    $m_users->addItem(['Users',       'icon'=>'users'],      'users');
    $m_users->addItem(['Purchases',   'icon'=>'money'],      'purchases');
    $m_users->addItem(['Subscribers', 'icon'=>'chart-line'], 'subscribers');
    $m_users->addItem(['Plans',       'icon'=>'basket'],     'plans');

    $menu->addItem(['Comments','icon'=>'chat-1', 'badge'=>[$new_comments,'swatch'=>'red']]);
    $menu->addItem(['Statistic', 'icon'=>'chart-bar', 'icon2' => 'export-1']);

.. figure:: figures/menu-ui-example.png

   Menu in action

With just about 10 lines of PHP code thanks to :ref:`abstraction`, you achieved
the following:

#. Made use of modern responsive CSS framework
#. Took advantage of Fontello icons
#. Created cross-browser standard-compliant HTML5
#. Assigned click and hover behaviour
#. Placed your widget at the right place in your app
#. Added JS injection protection for $new_comments
#. Ensured that your code will work with any Agile CSS Theme
#. Used solution based on JADE and LESS CSS
#. Cleverly set ID tags to your objects
#. Used template engine
#. Ensured that your menu labels can be localized

.. tip:: Agile Toolkit allows you to use PHP Objects to build user Interface
    out of blocks. You can tweak existing views, create new ones and share.


UI Fundamentals - Hierarchy of detail
=====================================

There are some things which you can do with the menu above, sucha as add sub-menus
and assign icons. Some other things are however not supported by standard menu.
You wouldn't be able to fit 2 icons on the left of menu.

Usually those limitations are set to keep developers from doing poor choices
when it comes to UI design. There are steps however to make any imaginable
change possible and well integrated into the toolkit:

#. Add more classes to your menu with :php:meth:`View::addClass`
#. Use JavaScript binding with :php:meth:`View::js` and jQuery.
#. Supply customized template through 4th argument of :php:meth:`AbstractView::add`
#. Extend view class and redefine :php:meth:`View::defaultTemplate`
#. Manually interract with :php:class:`GiTemplate` through `template` property.
#. Create your own :ref:`Agile CSS Theme` and tweak the look.
#. Write your own HTML and :ref:`create a new view`.

General Techniques of User Interface
====================================

.. toctree::
    :maxdepth: 1

    views/templates
    views/core
    views/custom

Useful Views in Agile Toolkit
==============================

.. toctree::
    :maxdepth: 2

    views/simple
    views/list
    views/menu
    views/layout
    views/grid
    views/form
    views/crud
    views/popover



