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

The table name corresponds with the collection. The ID field used
by Mongo is called ``_id``, however the ID in your model will be
set according to :php:attr:`Model::id_field` property.
