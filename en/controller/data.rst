****************
Data Controllers
****************

.. php:class:: Controller_Data

    Data controllers are used by "Model" class to access their data source.

:php:class:`Model` in Agile Toolkit is yet another abstraction. Technically
one model object can represent data stored in MySQL database, stored in
Session or retrieved from another server using HTTP RESTful API.

Bundled Data Sources
====================

Agile Toolkit comes with a set of bundled data sources. You can either use
them directly or extend those classes and form your own data sources.

.. toctree::
    :maxdepth: 1

    data/array
    data/session
    data/pathfinder
    data/memcached
    data/mongo
    data/sql
    data/restful
    data/dumper







TODO: write this
