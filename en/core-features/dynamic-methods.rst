Dynamic Methods
~~~~~~~~~~~~~~~

.. note:: Dynamic methods are like "traits" but for older versions of PHP.

PHP has a great way of extending object methods through a catch-all.
Agile Toolkit implements catch-all for all of its objects which is then
wrapped through some hooks to bring the functionality of addMethod(). It
allows you to register method in any object dynamically. Note, that if
such method already exists, you wouldn't be abel to register a method.
This is typically used for compatibility, when some methods are
obsoleted and controller re-adds those methods back.

::

    $obj = $this->add('MyClass');
    $obj->addMethod('sum', function($o, $a, $b) { 
        return $a + $b;
    });

Agile Toolkit also adds hasMethod() into all object as a preferred way
to check if method exists inside object;

::

    if ($obj->hasMethod('sum')) {
        $res = $obj->sum(2, 3); // 5
    }

You can also remove dynamically added methods with removeMethod();

::
    
    $obj->removeMethod('sum');
    
    echo $obj->hasMethod('sum'); // false
