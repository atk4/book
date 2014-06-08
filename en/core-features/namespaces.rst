Using namespaces / add-ons
~~~~~~~~~~~~~~~~~~~~~~~~~~

Agile Toolkit allows you to add objects from add-ons. Each add-on
resides in it's own namespace, so to add object you should use slash to
separate namespace and class name:

::

    $page->add('myaddon/MyClass');

Note: as you move code from your core app to add-on the format of using
add() from within a namespace remains the same.

.. seealso::
    :ref:`Addons`
    More infromation about Add-ons.
