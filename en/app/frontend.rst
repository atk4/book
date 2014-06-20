************
App_Frontend
************

This is a recommended class for your front-end web application. App_Frontend
is minimalistic, it will not add anything unless you request it to.

.. note:: When referring to ``App_Frontend`` and ``Frontend`` those are being
    two different classes. The first of those two is a standard class by
    Agile Toolkit - second is a suggested name of yoru own cusom class
    you create in your application.


Layouts and Page Routing
========================

Frontend will route pages as per a simple association:

+----------------------------------+------------------------+
| URL                              | Page location          |
+==================================+========================+
| http://example.com/user/settings | page/user/settings.php |
+----------------------------------+------------------------+

For a more advanced routing, see :php:class:`Controller_PatternRouter`

App_Frontend also allows you to use :php:class:`Layout`. Place this in Application init::

    $layout = $this->add('Layout_Fluid');
    $layout->addLeftMenu('MyMenu');
    $layout->addUserMenu('UserMenu');



.. php:method:: routePages

.. php:method:: pageNotFound

