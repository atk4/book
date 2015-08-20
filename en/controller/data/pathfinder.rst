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

You can also specify additional argument to PathFinder controller if you
pass array as a source::

    $m->setSource('PathFinder', ['template', 'path_prefix'=>'page');

This will make sure that only page/* tempaltes are returned.

.. note:: PathFinder controller will not descend into directories.

Supported Options
=================

- 'path_prefix' - will start searching withing this path
- 'strip_extension' - will remove extensions form files. You can set this
    value to true (will strip any extenison) or define which extension to
    use explicitly ``strip_extension=>'html'``. If you define it explicitly,
    then files which does not have this extension will be skipped.
