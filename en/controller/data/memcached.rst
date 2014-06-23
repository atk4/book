*************************
Controller_Data_Memcached
*************************

.. php:class:: Controller_Data_Memcached

    Implements read/write access to Memcached

.. note:: The class is called Memcached (not MemCached) to match the case of
    PHP class: http://www.php.net//manual/en/book.memcached.php

While you can use MemCached as a primary source for your model, it's main
purpose is for caching data::

    $m = $this->add('Model');
    $m->addField('name');
    $m->addField('base_path');


    $m->setSource('Pathfinder', 'template');
    $m->setCache('MemCached')

    $this->add('Grid')->setModel($m);

