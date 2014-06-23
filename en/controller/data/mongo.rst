*********************
Controller_Data_Mongo
*********************

.. php:class:: Controller_Data_Mongo

    Implements Model Integration with MongoDB


This controller implements access to MongoDB::

    $m = $this->add('Model');
    $m->addField('name');
    $m->addField('base_path');

    $m->setSource('Mongo', 'person');

    $this->add('CRUD')->setModel($m);

