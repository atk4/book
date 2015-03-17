*****************
Database Migrator
*****************

.. note:: available in Agile Toolkit 4.3.1 or later.

Migrator is a class which is capable of applying patches to your database.
The migrator in Agile Toolkit is implemented in a most basic way. It is
designed to persist migration statistics of MySQL inside database itself.
This way you no longer need to rely on filesystem files and can avoid many
problems with cloud deployments.

.. php:class:: Controller_Migrator_MySQL

    Implements a generic migration controller. This controller will find
    folder containing "dbupdates", read all the .sql files from there and
    sequentially apply them on top of your database::

    $app->add('Controller_Migrator_MySQL');

    or with a custom database::

    $app->add('Controller_Migrator_MySQL', ['db'=> $dbcon]);

    Database migration statistics is stored in _db_update table which is
    created if necessary.

Usage::

    $migrator = $this->add('Controller_Migrator_MySQL');
    $migrator->migrate();

.. php:method:: migrate

    Find migrations and execute them in order.

    .. warning:: Use ";" semicolon between full statements. If you leave empty statement between two semilocons MySQL ->exec() seems to fail.

.. php:method:: getStatusModel

    Produces and returns a generic model you can supply to your Grid
    if you want show migration status::

    $this->add('Grid')->setModel($migrator->getStatusModel());
