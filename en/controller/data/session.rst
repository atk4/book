***********************
Controller_Data_Session
***********************

.. php:class:: Controller_Data_Session

    Similar to Controller_Data_Array but the array is physically stored in session.

Use of Session data can quickly give you some persistance::


    $m = $this->add('Model');
    $m->setSource('Session', 'person');
    $m->addField('name');
    $m->addField('surname');

    $this->add('Grid')
        ->setModel($m, [ 'id', 'name', 'surname']);




Session can also be used as a cache for your model::

    $m = $this->add('SQL_Model', [ 'table' => 'person' ]);
    $m->setCache('Session', 'person_cache');
    $m->addField('name');
    $m->addField('surname');

    $this->add('Grid')
        ->setModel($m, [ 'id', 'name', 'surname']);

