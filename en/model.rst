***********
Data Models
***********

In Agile Toolkit Views can interact with the Data Layer through Models.
Model is a class implementing `Active
Record <http://en.wikipedia.org/wiki/Active_record_pattern>`__ and
`ORM <http://en.wikipedia.org/wiki/Object-relational_mapping>`__ as well
as holding meta-data for the objects.

One Model instance

.. figure:: /figures/orm-integration.jpg

Model Capabilities
------------------

Agile Toolkit model implementation can set up their capabilities depending
on Data Source. For example if your model is using ``setSource('Mongo')``,
you will be able to use conditions, iteration and some other features
supported by mongo. If you are using ``setSource('Memcache')``, features
like listing and conditions will not be available.

You must understand that there are two types of models - Relational and
Non-Relational (or NoSQL) and that Model\_Table (relational) extends the
other. To most of the User Interface both classes are compatible,
however both model implementation use a very different way to interact
with data.

Before we can understand Relational Models, we must first look into the
implementation of the DSQL layer. Models are described in the further
chapters.

Connecting to the database
~~~~~~~~~~~~~~~~~~~~~~~~~~

To connect to the database you need to call :php:class:`App_CLI::dbConnect`.
For this method to work without any arguments, you need to specify DSN
(connection info) in config.php file. Alternatively you can also specify
DSN as an argument, although be advised that if you use passwords inside
GET argument, they will be visible on exception screen, when your
database is unreachable. You can disable error output on the screen, see
chapter for "Application" for more information.

-  Specifying DNS
-  Using different connection modes
-  Counting number of queries in application
-  Using multiple connections
-  Connecting to noSQL databases

====================================

.. toctree::
    :maxdepth: 1

    model/core
    model/fields
    model/relation
    model/dsql
    model/sql
    model/mongo

