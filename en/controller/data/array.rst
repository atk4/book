*********************
Controller_Data_Array
*********************

.. php:class:: Controller_Data_Array

    Allows accessing array through a model interface.


Basic Set-up
============

In the most simple set-up a regular array of strings can be used as a source
for array model::

    $array = ['John', 'Peter', 'Steve'];

    $m = $this->add('Model');
    $m->setSource('Array', $array);

    foreach($m as $id => $row) {
        echo $row['name'];
    }

A ``name`` is the default name for the title column and if array records
does not have hash, then it's considered to have only one column - "name".
The ID of this array would be 0, 1 and 2.

Array of Hashes
===============

I'm going to use aray of hashes to define my Menu structure through array::

    $menu_data = [
        [ 'name'=>'Dashboard', 'page'=>'dash' ],
        [ 'name'=>'Settings', 'page'=>'user/settings' ],
        [ 'name'=>'Log Out', 'page'=>'logout' ],
    ]

    $this->add('Menu_Vertical')
        ->setModel('Model')
        ->setSource('Array', $menu_data);

Hash of Hashes
==============

You are more than welcome to specify a key for the array too. Agile
Toolkit will try to determine which data you are using by looking at
the first record. You can also omit some column values in your data::


    $data = [
        1 => [ 'name'=>'John', 'surname'=>'Lush' ],
        3 => [ 'name'=>'Betty', 'surname'=>'Smith' ],
        8 => [ 'name'=>'Peter', 'surname'=>'Cute' ],
    ]

    $m = $this->add('Model');
    $m->setSource('Array', $data);
    $m->addField('name');
    $m->addField('surname');

    $m->tryLoadAny();
    $this->add('H1')->set('First user name: '.$m['name'].' and the rest are:');

    $this->add('Grid')
        ->setModel($m, [ 'id', 'name', 'surname']);

