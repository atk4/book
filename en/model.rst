***********
Data Models
***********

In Agile Toolkit Views can interact with various Data Sources through Models.
Model is a class implementing `Active
Record <http://en.wikipedia.org/wiki/Active_record_pattern>`__ and
`ORM <http://en.wikipedia.org/wiki/Object-relational_mapping>`__ as well
as holding meta-data for the objects.

One Model instance

.. figure:: /figures/orm-integration.jpg

Quick Example if you are familiar with ORM principles::

    // Definition

    class Model_User extends Model_SQL {
        function init(){
            parent::init();

            $this->addField('name');
            $this->addField('address')->type('text');
            $this->hasMany('Order');
        }
    }

    class Model_Order extends Model_SQL {
        function init(){
            parent::init();

            $this->hasOne('User');
            $this->hasOne('Item');
            $this->addField('is_shipped')->type('boolean');
        }

        function ship(){

            $address = $this->ref('user_id')['address'];

            // .. do hard work ..

            $this['is_shipped'] = true;
            $this->save();
        }
    }

    // Usage

    $m=$this->add('Model_User')->ref('Orders');

    $m->addCondition('is_shipped', false);

    foreach($m as $order) {
        echo "Shipping order ".$m;
        $m->ship();   // calling custom method
    }


Model Explained
---------------

Model is an object which creates a flexible API for your software to interract
with physical data storage without knowing details about the specifics of
data layer. In other words:

- Models allow you to use most of MySQL features without SQL language.
- Models allow you to use most of MongoDB features without using their API directly.
- Any REST API can be used as a model source.
- Models can work with arrays, session, filesystem etc
- Models support transparent caching
- You can extend by adding new data sources for a model

If you intend to work with any data, you should learn to use Models.

Further Reading
---------------

.. toctree::
    :maxdepth: 1

    model/core
    model/fields
    model/relation
    model/dsql
    model/sql
    model/mongo




Agile Toolkit model implementation can set up their capabilities depending
on Data Source. For example if your model is using ``setSource('Mongo')``,
you will be able to use conditions, iteration and some other features
supported by mongo. If you are using ``setSource('Memcache')``, features
like listing and conditions will not be available.

Model implementation in 4.3 and 4.2
-----------------------------------

Models were initially introduced in Agile Toolkit 3.9 and then they
were greatly improved in the subsequent releases. Most of the time
it has been done without major impact on the usage pattern and

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

-  Specifying DSN
-  Using different connection modes
-  Counting number of queries in application
-  Using multiple connections
-  Connecting to noSQL databases

====================================

