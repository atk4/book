************
Model Basics
************

.. php:class:: Model

    Model class improves how you interact with the structured data. Extend
    this class (or :php:class:`SQL_Model`) to define the structure of your
    own models, then use object instances to interact with individual records.

Introduction
============

In Software Computing most of the time you deal with structured data, like
this table below:

+----+--------+---------+--------+
| id | name   | surname | gender |
+====+========+=========+========+
| 1  | John   | Smith   | m      |
+----+--------+---------+--------+
| 2  | Betty  | Shark   | f      |
+----+--------+---------+--------+
| 3  | Alfred | Hitch   | m      |
+----+--------+---------+--------+

Data of this table can be stored in SQL or Non-SQL engine,
however some of the common characteristics are pertained e.g.:

- Data is stored as individual records (row or schema)
- All the records have identical or nearly similar structure
- Each record has unique identifier (id)
- Each column has type

Different data engines would introduce a different ways to interract, for example
if you wish change Betty's surname to "Hitch", you would do it differently::

    SQL: update person set surname='Hitch' where id=1;

    MongoDB: ....

Concept of models proposes a common format for any data source, which looks
like this::

    $person->load(1)->set('surname', 'Hitch')->save();

Once we have a single unified way to interact with data, our User Interface
Views can now seamlessly load and save data using the model you specify.

Defining the Model
==================
Similarly to how you need to create a database structure before you store data
in SQL, you would also need to define a model in Agile Toolkit before you can
use it.

.. note:: While there are automated ways to convert model into SQL and back, I
    recommend you to initially do this manually. Later in this documentation
    I'll give my reasons as to why.

.. note:: Some databases can offer non-structured data. For that purpose Agile
    Toolkit Models allow you to access fields even if those fields are not
    defined. You can also use advanced PHP logic to dynamically define
    fields depending on your situation.

Model in Agile Toolkit is an object of a "Model" class. To create your own model
you need to create your own class by extending Model::

    class Model_Person extends Model {
        public $table='person';
        function init() {
            parent::init();

            $this->addField('name');
            $this->addField('email');

            $this->setSource('MongoDB');
        }
    }

The ``setSource`` part here is optional and it's linking your model with
a physical data storage. It enables you to load and save data seamlessly.

After you have defined yoru model, you can create objects of 'Model_Person'
as you normally do it with any other object in Agile Toolkit::

    $person = $this->add('Model_Person');

.. php:attr:: table

    Contains name of table, collection, file or session key, depending on
    data controller you are using.

Core Concepts
=============

Before continuing with explanation of all the other features and capabilities of the
model in Agile Toolkit, you would need to understand some of the core concepts
and terminology used by the framework:


Record Loading
--------------
If you have seen ORM in other frameworks, you might be familiar with the
principle of havign a separate objects for each record. Agile Toolkit uses a
different principle of record loading.

A single Model Object may have record "loaded" inside itself at any given time.
The record data can then be "unloaded" and the same object can be re-used
for another record.

.. php:method:: tryLoad

    Ask the data controller to load the model data from source by a given $id

.. php:method:: load

    Like tryLoad method but if the record not found, an exception is thrown

.. php:method:: unload

    Forget loaded data

.. php:method:: loaded

    Returns true if the model is loaded

Here are a few examples of loading and unloading data::

    $person = $this->add('Model_Person');
    echo $person->loaded();  // false, not loaded

    $person->load(1);
    echo $person->loaded();  // true now

    $person->load(2);        // no need to explicitly unload
    echo $person->loaded();  // still true

    $person->unload();
    echo $person->loaded();  // false now

    $person->tryLoad(12313123); // no such record
    echo $person->loaded();  // will be false

    $person->load(12313123); // generates an exception

You will see a common pattern in Agile Toolkit pages, where
models are loaded with the data passed through the GET parameters::

    $this->person  = $this->add('Model_Person')->load($_GET['id']);

If the specified ID passed here is not found in the database, then
exception is generated and is handled by API.

Generic models
----------------
While a general rule says that all your business models needs to be defined
as classes extending from Model or SQL_Model, you can, however, have a
generic model defined like this::

    $m = $this->add('Model', ['table'=>'person']);
    $m->addField('name');
    $m->setSource('Array', ['John', 'Peter', 'Joe']);

The short notation demonstrated here is good if you are simply willing to
test model functionality and do not require comprehensive model definition.

Accessing and Changing field values
-----------------------------------
Model contains the information loaded from the Data Source and
there are several ways to access it.

.. php:method:: get($name = null)

    Get the value of a model field. If field $name is not specified, then
    returns associative hash containing all field data

.. php:method:: set($name, [$value])

    Set value of fields. If only a single argument is specified
    and it is a hash, will use keys as property names and set values
    accordingly.

To complement the example below, I'll also use :php:meth:`Field::defaultValue`
inside field definition. In this example, I'm using generic class for the model,
instead of extending it and creating a separate model::

    $m = $this->add('Model', ['table'=>'person']);
    $m->addField('name');
    $m->addField('age')->type('int')->defaultValue(18);
    $m->setSource('Array', ['John', 'Peter', 'Joe']);

    $m->load(1);
    echo $m->get('name');
    $m->set('age', 25);

    var_dump($m->get());   // outputs [ id=1, name=Peter, age=25 ]

You can also use model as Array, instead of set / get use square brackets::

    $m['age'] = 25;
    echo $m['name'];

.. note:: You can't use ``$m['age']++`` due to some PHP limitation.

.. php:attr:: data

    This is a actual property of the model which contains current field
    data. Avoid using this property directly and use get/set or array-access
    instead.

.. note:: It is recommended that you keep field types in their native form
    for PHP. For example if you operate with boolean type field, use ``true``
    and ``false`` instead of "Y" / "N" or 1 / 0 values.

    Setting ``->type()`` on the field will actually help data controller
    to properly convert data types. For example MongoDB controller will
    convert "date" from PHP format into MongoDate when storing data.

.. _model dataset:

The Dataset
-----------
In a traditional database design, the underlying database engine would
group all the data into tables for optimisation purposes. One table, however,
can contain different "types" of data. "user" table may contain both regular
users and admin users. A boolean field "is_admin" could be used to separate
regular users from admin.

Agile Toolkit heavily uses a term of "DataSet", which is a set of records
from a table possibly restricted by some condition.

In my previous example you have one data-set - 'Model_User' and another,
mach smaller dataset - 'Model_Admin'. The purpose of a data-set is to
make sure that when you work with 'Model_Admin' it can't possibly load
record from outside of it's allowed dataset (e.g. non-admin user).

Another use of DataSet is record ownership. Imagine a system where
users can create orders. Each order would have "user_id" field and
user must only be able to access orders where user_id matches his own ID.

A classical problem which often occurs in software design is when you
individually design queries. It's a very common mistake where developer
forgets to add condition and system can now load order owned by another user::

    select * from `order` where user_id = ? and id = ?

Imagine a separate deletion page with a query like this::

    delete from `order` where id = ?

While our first query would correctly verify user_id and only allow loading
of user's record, the deletion query lacks an extra check and by cleverly
substituting ID, user can now delete orders belonging to anothre user.

Agile Toolkit ORM framework trains you to think in term of DataSets.

    $user = $this->add('Model_User')->load($user_id);
    $orders = $user->ref('Order');
    $orders -> load($_GET['id'])->delete();

The above code in Agile Toolkit correctly creates a DataSet with
a 'Model_Order' which can only load records of a specified user.

By learning to write code like that you will avoid many errors in your
code.


To summarize: Data Set is a collections of records which model is allowed
to load, update or delete. Here is how you can define DataSet conditions::


    class Model_MyOrder extends Model_Order {
        function init(){
            parent::init();

            $this->addCondition('user_id', $this->app->auth->model->id);
        }
    }


    // And then
    $my_order = $this->add('Model_MyOrders');
    $my_order['name'] = 'Test Order';
    $my_order->save();

    // this automatically sets order.user_id to that of a currently
    // logged-in user

The same code can also be written differently::


    $my_order = $this->app->auth->model->ref('Order');
    $my_order['name'] = 'Test Order';
    $my_order->save();


Relation Traversal
------------------
When you define models, you can specify how they relate to other models.
There are 2 types of basic relations: ``hasOne`` and ``hasMany``::

    $user->hasMany('Order');

    $order->hasOne('User');


Note that Agile Toolkit will automatically add Model_ in front the argument
if it's not present.

.. php:method:: hasOne(model_name, field)
    Establishes many-to-one relationship. $order->hasOne('User');

    This also adds 2 fields to the model:

    ``user_id`` field will be used to store "id" of related record.

    ``user`` expression is added which will display title of related record.


.. php:method:: hasMany(model_name)
    Establishes one-to-many relationship. $user->hasMany('Order');

    Does not add any fields to your model.

.. php:method:: ref

    Traverses reference of relation

You can traverse thereference both references using ref, however you must
properly use case to indicate direction::

    $user = $my_order->ref('user_id');

    $user_orders = $user->ref('Order');

Few more rules:

- $my_order->ref('user_id'); in this case $my_order model must be loaded.
- ref('user_id') returns user model which will also be loaded.
- $user->ref('Order') must similarly be called on a loaded model.
- ref('Order') will return model which is NOT loaded.
- ref('Order') will restrict DataSet of returned model to records related to $user

Getting back to our example::

    $user = $this->app->auth->model;
    $order = $user->ref('Order');

    $order->load($_GET['id'])->delete();

First line will give us model of a currently-logged in user. The definition
of Model_User must have relation defined (hasMany('Order')).

Second line traverses through that relation and will return a new model.
This model will not be loaded, but it's DataSet will automatically be restricted
to "user_id = ?" - that of a currently-logged user ID.

When load() and delete is attempted on next line, Agile Toolkit will only
be able to load order of a currently logged-in user.

.. note:: Traversal in Agile Toolkit works even if models are using
    different data sources. For example a model stored in MongoDB can
    relate to model stored in SQL.

.. todo:: deep taversal is a concept allowing you to traverse one-to-many
    relation of unloaded model.

    $user->ref('Order')->ref('Order_Line');

    This is not implemented in Agile Toolkit yet, however you can get
    around this problem with some expression magic for SQL.


Data Source
-----------

.. php:method:: setSource($driver, $table)

    Associate model with a specified data source. The controller could be
    either a string (postfix for ``Controller_Data_..``) or a class. One data
    controller may be used with multiple models.
    If the $table argument is not specified then :php:attr:`Model::table`
    will be used to find out name of the table / collection

Each model may have a source set. The source is set like this::

    $model->setSource('Session');

    or

    $model->setSource('Array', $arr);

    or

    $model->setSource('MongoDB', 'mycollection');

The first argument here is a name of ":php:class:`Controller_Data`" - a special class
which will control loading and saving of data.

.. note:: If you are extending from SQL_Model you do not need to specify
    a data source - it will work with your current database connection. In the
    future versions of Agile Toolkit Modal_SQL will transition in favor of
    setSource('SQL').

The second argument is optional and if it's specified it will override
:php:attr:`Model::table` of the model. The type of this argument
can vary from driver to driver.

.. note:: Calling load() and then save() right after may not actually
    execute save. Model automatically tracks fields which have been
    changed from it's initial values and will only save those into
    data source.

.. todo:: expand section on using is_dirty();

Below are table comparing different drivers and showing how the meaning of table
and condition change.

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


Relational Model
----------------

A significant segment of the database implementations are so called
RDBMS - Relational Database Management Systems. Notable for their
flexibility in data querying they utilize a standardized query language
- SQL. Agile Toolkit takes advantage of the powerful features of RDBMS
(joining, sub-selects, expressions) and has a significantly enhanced
model class to work directly with the database through DSQL.

To take advantage of those features you must use :php:class:`SQL_Model`.
This class extends Model but adds features of a typical ANSI SQL directly
into model. Refer to the documentation of :php:class:`SQL_Model`

You can still use generic Model with SQL driver, such as SQLite, but
both use slightly different implementations. As of version 4.3 I recommend
using SQL_Model as it is much more tested and optimized. You will also
be able to use expressions and joins freely.

Using Caching
-------------

A single model can have several caches associated with it. For example a
relational model may have Session cache.

When loading model by id with associated cache - the first attempt is made to
load the model from the cache directly. If model is not found in
cache(s), the primary source is used as a fall-back.

When saving model data, it will be also saved into all the associated
caches.

The same data controller class can be used as either primary source or
as a cache.

.. todo:: expand this section, write about setCache(), strategies
    and use of multiple caches.

How to write Model Code
-----------------------
Model is an essential part of your application containing business logic.
You must refrain from using any of the following from inside your model:

- GET and POST arguments, which are exclusive to app running in Web environment
- UI objects or pages, which may not be there in CLI application.
- Add method documents and think about use cases when model data is loaded / unloaded.
- Think about transactions and commits.
- Write test-cases for your models.

There is a special rule for relying on authenitcation data. In this documentation
I have given example for MyOrders model, which display orders of the currently
logged-in user. This is a valid usage pattern, but you must use it in a separate
class which implies reliance on user being logged in.

Another example of external dependency being valid is if your application have a
system-wide filter. You might want
to create Model_FilteredOrder which would automatically apply conditions from
the global filter, but you should not do that inside the base model.

With those basic requirements in mind, you can now create methods inside
your model class to wrap up some business logic.

.. todo:: REVIEW beyond this point.

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

