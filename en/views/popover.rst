Popover
=======


.. php:class:: View_Popover

    Popover View creates a invisible call-out, which can be triggered through a :ref:`JavaScript Chain`.

.. php:method:: showJS

    Returns JavaScript chain which will make popover visible when executed. See :ref:`JavaScript Chain`.

.. figure:: /figures/popover-hello-world.png

Technically popover is created using a jQuery UI Dialog, however Agile Toolkit
adds a lot of usability in PHP::

    $i = $this->add('Icon')->set('window');

    $pop = $this->add('View_Popover');
    $pop->add('HelloWorld');

    $i->js('click', $pop->showJS());

Method ``showJS`` accepts one array-type argument - options, which can contain
the following values:


+---------+-----------------+-------------------------------------------------------------------------------------------+
| key     | default value   | meaning                                                                                   |
+=========+=================+===========================================================================================+
| my      | center top      | see jQuery UI positioning widget                                                          |
+---------+-----------------+-------------------------------------------------------------------------------------------+
| at      | center bottom+8 | see jQuery UI positioning widget                                                          |
+---------+-----------------+-------------------------------------------------------------------------------------------+
| modal   | true            | popup will close when you click anywere outside of the popup                              |
+---------+-----------------+-------------------------------------------------------------------------------------------+
| class   | atk-popover     | which base class to use to make dialog look like popover                                  |
+---------+-----------------+-------------------------------------------------------------------------------------------+
| open_js | null            | Execute certain code when clicked                                                         |
+---------+-----------------+-------------------------------------------------------------------------------------------+
| width   | 250             | width of the popover. Set this to ``false`` if you want to specify width through a class. |
+---------+-----------------+-------------------------------------------------------------------------------------------+
| tip     | top-center      | location of the tip on the call-out: bottom-right, top-right, etc. This does not affect   |
|         |                 | the position                                                                              |
+---------+-----------------+-------------------------------------------------------------------------------------------+

Other options will be passed to the jQueryUI dialog(), so refer to their documentation.

.. php:method:: setURL

    A specified URL will be automatically loaded in the popover after it's clicked every time. If you wish
    to specify a custom code, use ``open_js`` property.

.. php:method:: addClass

    Allow you to add more classes on the pop-over. It's important that you call
    this method before bindJS()



Example using Popover with Virtual Page
---------------------------------------

The following example creates a :php:class:`VirtualPage` which is then loaded
in the pop-over. Virtual page allows us to contain all the code together::

    $i = $this->add('Icon')->set('window');

    $pop = $this->add('View_Popover');
    $pop->add('HelloWorld');

    $vp = $this->add('VirtualPage');

    $vp->set(function($p){
        $p->add('H1')->set(rand(1,100));
    });

    $pop->setURL($vp->getURL());

    $i->js('click', $pop->showJS());

Output
^^^^^^

.. figure:: /figures/popover-green.png

