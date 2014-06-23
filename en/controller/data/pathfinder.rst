**************************
Controller_Data_PathFinder
**************************

.. php:class:: Controller_Data_PathFinder

    Implements access to PathFinder resources through model.

With this you can easily create a list of all the accessible templates::

    $m = $this->add('Model');
    $m->addField('name');
    $m->addField('base_path');


    $m->setSource('PathFinder', 'template');

    $this->add('Grid')->setModel($m);
