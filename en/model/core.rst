**********
Model Core
**********

.. php:class:: Model

    Implementation of Model paradigm in Agile Toolkit

Introduction
============

In your software you will be dealing with structured data:

+----+--------+---------+--------+
| id | name   | surname | gender |
+====+========+=========+========+
| 1  | John   | Smith   | m      |
+----+--------+---------+--------+
| 2  | Betty  | Shark   | f      |
+----+--------+---------+--------+
| 3  | Alfred | Hitch   | m      |
+----+--------+---------+--------+

This data is stored in a special dedicated location we call Data Source.
As you see - data structure is consistent throughout the set (table) and
can be broken into 4 fields. One of those fields is a unique identifier (id)

If your data has the above properties, then you can take advantage of a
Model class to access the data.

Models offer syntactic sugar and use unification for various data sources.
With a unified interface various UI objects can safely access and modify data.

Example Usage
-------------

The following example demonstrates simple data manipulation, to give you
better feeling of what model is::

    $person->load(2);
    echo $person['name'];    // outputs 'Betty'

    $person['surname']='Smith';
    $person->save();         // Updates surname to 'Smith'

Features
--------

In Agile Toolkit model class have the following features:

- Defining column structure and types
- Creating one model by extending another
- Loading one row at a time, manipulating and saving it
- Defining custom methods dealing with data
- Iterating through available records (:ref:`model dataset`)
- Callbacks (e.g. afterLoad or beforeSave)
- Reference traversal

Additionally with the help of Data Source capabilities more features
can be available:

- Adding conditions (filters) on models
- Executing actions on all of the Data Set (update all) without iterating
- Defining skip / count (limit) for records
- Storing complex values in model

A relational database managers (RDBMS) or SQL Servers are capable of
more features and Agile Toolkit provides ways to take advantage of those
features without manually writing queries:

- Joining tables
- Using expressions
- Using sub-selects based on model
- Applying action with existing conditions
- Operating with "actual" field subset

Agile Toolkit standard Data Controllers try to provide you with access to
the features of underlying Data Source, however they will not emulate
features lacking in the Database.

- One primary Data Source per model
- Several secondary Data Sources (caches) per model
- Knowledge of Data Source capabilities

Class Structure
---------------

I have already introduced the main class - :php:class:`Model`, which can
operate with any Data Source::

    $m = $this->add('Model', [ 'table' => 'user' ]);
    $m->setSource('SQL');
    $m->addField('name');
    $m->addField('surname');

However this Model implementation may not support all the features of the
Data Source. A more advanced Data Sources will have a dedicated model class
you can use::


    $m = $this->add('SQL_Model', [ 'table' => 'user' ]);
    $m->addField('name');
    $m->addField('surname');
    $j = $m->join('contact_info','user_id');
    $j->addField('address');
    $m->addCondition('gender', 'm');
    $m->addExpression('full_name')->set('concat(name, " ", surname)');

Limitations and Recommendations
-------------------------------

In order to make working with model more predictable, you must remember
that you must follow these rules:

- Each record must have an ``id`` (numeric or alphanumeric)
- Each ID must correspond to hash of values (by fields), where key is (alphanumeric)
- Model should have field defined (and field types/properties)
- One field is a Title Field (normally "name")
- Model can only access items within data-set (matching conditions)
- Model can only create items which will match match data-set conditions


Creating Data Controllers
-------------------------

Data Controllers implement :php:meth:`Model::load` / :php:meth:`Model::save`
method and some other extensions to the model. If you would like to learn
more about Data Controllers, see :php:class:`Controller_Data`. The rest
of this chapter will focus on defining and using models with existing
controllers.

If you are interested in specific data source features, see:

- :php:class:`Controller_Data_Array` - static array access for models
- :php:class:`Controller_Data_Session` - storing data in Session
- :php:class:`Controller_Data_Mongo` - Accessing MongoDB collections
- :php:class:`Controller_Data_SQL` - PDO-based SQL access. See :php:class:`SQL_Model`
- :php:class:`Controller_Data_Memcache` - Memory Cache
- :php:class:`Controller_Data_RESTful` - Accessing remote API through Model


Model Data
==========

PHP objects are an ideal container for both the data and the set of
methods which can be applied on the data. Agile Toolkit Models enhance
basic objects with some other handy methods. Model hides the Data Controller
from you and lets you simply interact with data without need to know where and
how data is stored.

.. figure:: /figures/model.png

- Each record have unique ID which can be number or string.
- Each record may have value of a String or a Hash
- Model object may have one record ``loaded``
- Model may have several ``conditions``.
- Only records matching conditions may be loaded
- All records which can possibly be loaded are called ``dataset``

Despite model being associated with "table" or "collection" it's dataset
may match a sub-set of available data in table due to conditions.

.. _model dataset:

The Dataset
-----------

Dataset is determined by 3 things: 1) Driver 2) Table 3) Conditions.

+-------------------------+-------------------+--------------------------------------------------+
| Driver                  | Table             | Condition                                        |
+=========================+===================+==================================================+
| SQL + Database/Schema   | Table Name        | set of "where" conditions joined by AND clause   |
+-------------------------+-------------------+--------------------------------------------------+
| Memcache                | Key Prefix        | Sub-prefix                                       |
+-------------------------+-------------------+--------------------------------------------------+
| MongoDB                 | Collection Name   | Conditions                                       |
+-------------------------+-------------------+--------------------------------------------------+
| Redis + Object Type     | Object name       | Prefix                                           |
+-------------------------+-------------------+--------------------------------------------------+

Here are some examples:

+-------------------------+-----------------------------+---------------------+------------------------------+
| Use Case                | Driver                      | Table               | Condition                    |
+=========================+=============================+=====================+==============================+
| Model\_Admin            | MySQL                       | user                | is\_admin=1, is\_deleted=0   |
+-------------------------+-----------------------------+---------------------+------------------------------+
| Model\_ShoppingBasket   | Controller\_Data\_Session   | basket              |                              |
+-------------------------+-----------------------------+---------------------+------------------------------+
| Model\_BasketItems      | MySQL                       | item, join basket   | basket.user\_id=123          |
+-------------------------+-----------------------------+---------------------+------------------------------+

Relational Model
----------------

A significant segment of the database implementations are so called
RDBMS - Relational Database Management Systems. Notable for their
flexibility in data querying they utilize a standardized query language
- SQL. Agile Toolkit takes advantage of the powerful features of RDBMS
(joining, sub-selects, expressions) and has a significantly enhanced
model class to work directly with the database through DSQL.

You can find a detailed description of relational models further in this
book. Even through the relational models are significantly enhanced,
they still retain the functionality of regular models, so everything
described in this chapter would also apply to relational models.

setSource - Primary Source
~~~~~~~~~~~~~~~~~~~~~~~~~~

A non-relational models can use ``setSource()`` method to associate
themselves with a driver. Driver is an object of class extending
Controller\_Data. Model will route some of the operations to the
controller, such as loading, saving and deleting records.

Model can only have one source and because relational models already
using SQL you cannot specify a different source.

addCache - Caches
~~~~~~~~~~~~~~~~~

A single model can have several caches associated with it. For example a
relational model may have Session cache.

When loading model with associated cache - the first attempt is made to
load the model from the cache directly. If model is not found in
cache(s), the primary source is used as a fall-back.

When saving model data, it will be also saved into all the associated
caches.

The data controllers typically can be used as either primary source or
as a cache.

Model data and methods
~~~~~~~~~~~~~~~~~~~~~~

In a typical ORM implementation, model data is stored in model
properties while reserving all the property names beginning with
underscore. Agile Toolkit stores model data as array in a single
property called "data". To access the data you can use ``set()``,
``get()`` or array-access (square brackets) format.

Before you can access the data, however, you must define some fields.
Below is a typical implementation of a model in Agile Toolkit. Please
note that model is defined using PHP language and it's always defined as
a class.

::

    class Model_User extends Model {
        function init(){
            parent::init();
            $this->addField('name');
            $this->addField('surname');

            $this->addField('daily_salary');
            $this->addField('due_payment');
        }
        function goToWork(){
            $this['due_payment'] = $this['due_payment']
                +$this['daily_salary'];
            return $this;
        }
        function paySalary(){
            echo "Paying ".$this['name']." amount of ".
                $this['due_payment'];
        }
    }

    $m=$this->add('Model_User');
    $m['name']='John';$m['daily_salary']=150;

    for($day=1;$day<7;$day++) $m->goToWork()
    $m->paySalary();

As you see in the example, model User's model combines definition of the
fields with the methods to perform business operations with the model.
When you design model methods, it's important that you follow these
guidelines:

-  Never assume presence of UI.
-  Avoid addressing "owner" object.
-  Keep object hierarchy in mind. Extend "User" model to create
   "Manager" model.
-  All field names must be unique

By following these guidelines, you can design a model which can work
with magnitude of data sources.

Loading and Saving models
~~~~~~~~~~~~~~~~~~~~~~~~~

You can save your model data to a primary source driver or load data if
you know the "id" of the record. The "id" is not necessarily a number,
but it uniquely defines a data within source / table.

Let's extend our user model by adding "Session" source.

::

    class Model_User extends Model {
        public $table='user';
        function init(){
            parent::init();
            $this->setSource('Session');

Once source is set, you can use a number of additional operations:

::

    $m['name']='John';$m['daily_salary']=150;
    $m->save();
    echo $m->id;    // will contain a generated ID

    $m->load($other_id);    // load different record into model

Model objects in Agile Toolkit are not tied in with any particular
record. They can load any (but one) record from the data-set and save
it. A single object can also iterate through the data-set by loading
each individual record.

There are only two properties which are affected when you load model:
"data" and "id". Next example demonstrates how to display list of all
the users and their respective "due\_payment" field:

::

    foreach($m as $row){
        echo "Please pay ".$row['daily_salary']." to ".
            $row['name']."\n";
    }

When iterating, the
:math:`row becomes automatically associated with the "data" property, however if you are willing to change the content of the model, you should use the `\ m
instead:

::

    foreach($m as $row){
        $m->paySalary();
    }

Model's method ``loaded()`` will return true if model have been loaded
with any data from the source and false otherwise.

::

    $m=$this->add('Model_Table');
    $m->loaded();    // false
    $m->load(1);
    $m->loaded();    // true
    $m->unload();
    $m->loaded();    // false


Deleting model data
~~~~~~~~~~~~~~~~~~~

You can delete a single record of data by calling delete() method or you can
remove all data by calling deleteAll(). If you do not pass id to delete()
method, then loaded record will be deleted.

