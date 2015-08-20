****************
Data Controllers
****************

.. php:class:: Controller_Data

    Data controllers are used by "Model" class to access their data source.

:php:class:`Model`'s storage in Agile Toolkit is yet another abstraction. Technically
one model object can represent data stored in MySQL database, in
Session or retrieved from another server using HTTP RESTful API.

The Model class on it's own can exist without any "Data Source", however
when you attempt to save() or load() the data, the model will work with
associated data controller to perform the opperation.

.. warning:: The implementation for Controller_Data has changed in 4.3.

The below syntax will associate model with data controller::

    $book = $this->add('Model');
    $book -> addField('title');
    $book -> hasOne('Author');

    $book -> setSource('Clusterpoint', 'book');

    $book -> load($id);   // retrieve data from MongoDB


Standard Data Sources
=====================
Agile Toolkit comes with a number of built-in data controllers:

.. toctree::
    :maxdepth: 1

    data/array
    data/session
    data/pathfinder
    data/restful
    data/clusterpoint
    data/memcached
    data/mongo
    data/sql
    data/dumper

Creating your own Data Controller
=================================

You can create a new data controller by creating class with name matching
``Controller_Data_*`` and extending it from :php:class:`Controller_Data`.

The minimum which you would need to do is to re-define 4 abstract methods
of base class:

* save($model, $id, $data)
* delete($model, $id)
* loadById($model, $id)
* prefetchAll($model)

It's most likely you would also need to set up setSource().

If you are dealing with read-only data source, you should create empty
methods for save/delete and throw exception when an attempt is made to
store data.

.. php:method:: setSource

    Associates model with the collection/table in this data store identified
    by second argument. For instance for Controller_Data_SQL the second
    argument would be a table.

    All necessary data must be stored in $model->_table[$this->short_name]
    to avoid conflicts between different controllers and to allow user to
    use one controller with multiple models.

    You should use method d() to access this property. We recommend
    that you use the following format for specifying argument::

        $m->setSource('YourController', 'table');

        $m->setSource('YourController', ['table', 'arg1'=>'val1', ..]);



.. php:method:: save

    Writes record containing $data into the data store under id=$id.
    If the ID field can vary, consult $model->id_field. You do not have
    to worry about default fileds, because their values will automatically
    be in $data.

    If $id is null then new record must be created. If $id is specified,
    then record must be overwritten. Method must return $id of stored record.
    This method may throw exceptions.

.. php:method:: delete

    Locate and remove record in the storade with specified $id. Return
    true if record was deleted or false if it wasn't found. In most
    cases, the record data will first be loaded and then deleted, just
    to make sure the record is accessible with specified conditions,
    so you can silently ignore error.

.. php:method:: loadById

    Locate and load data for a specified record. If data backend supports
    selective loading of fields, you may call model->getActualFields
    to get a list of required fields for a model. When non-array is
    returned, you should load all fields.

.. php:method:: prefetchAll

    Create a new cursor and load model with the first entry. Returns cursor,
    which will then be passed to loadCurrent() for loading more results.

    If the data store does not support cursors, then fetch all data
    and return array. loadCurrent will automatically array_shift one record
    on each call.

.. php:method:: d(model)

    This methods exists for easily access data, that was passed to setSource.
    and can be used as a short-cut for ``$model->_table[$this->name]``.

    Method returns reference, so you can change it's contents.

Supporting Conditions
=====================

Condition is ability to limit record set by providing a test-criteria
against non-ID field. Here is example use of conditions in models::

    $user -> addCondition('gender','M');

Controller used with this model must advertise it's support for ``conditions``
and ``operators`` by setting the following attributes to ``true``.

.. php:attr:: supportConditions

    Conditions can be set on non-primary field. This capability of
    data controller allows your model to use methods such as
    addCondition, loadBy, tryLoadBy etc. Each of those methods can
    be supplied with two arguments - the field and value and
    exact matching will be used. For three-argument use, see
    property $supportOperators

.. php:attr:: supportOperators

    Operators add ability to use fuzzy-match on conditions.
    Controller is required to support '>', '<', '=', '!=',
    '<=', '>=' operators even if database uses diffirent
    notation, due to consistency.

    If database supports additional operators, you can list
    them as an array in this attribute, e.g.: ['like','is','is not'].

    You may also set $supportOperators='all' which will indicate
    that this controller supports all possible operators, although
    this is not advised.

If supportConditions is ``true``, but supportOperators is set to false,
then the only condition is a full-match on field, with a syntax of::

    $model -> addCondition('name','John');

    $model -> addCondition('suranme','=','Smith');

supportConditions also implies that conditions can be set on multiple fields
simultaniosuly.

Conditions are stored in $model->conditions in format of
``[[$field, $operator, $value], ..]``

Ordering Support
================

If your data engine supports ordering of data in ascending/descending order
on some fields, then this property needs to be set to true:


.. php:attr:: supportOrder

    Controller allows re-ordering data set by non-primary field
    in either ascending or descending order. Models support
    ordering on multiple fields. If controller does not
    support multiple ordering fields, then the last
    field to be used for ordering must take precedence

The order is inside $model->order in format of ``[$field, $is_desc]``, where
second argument is boolean.

Limit Support
=============

If your data engine supports limiting records (skip and count), then set
the following propert true:


.. php:attr:: supportLimit

    Limit makes it possible to specify count / skip for the
    model, which is necessary for pagination.
    bleh

The limit will be found in $model->limit in format ``[$limit, $offset]``.

Testing Data Controller
=======================

.. todo:: must create a test-suite

When you add a new data-controller you must run it against a specially designed
test-suite, which will attempt to perform all operations. The test-suite will
automatically adjust itself by verifying ``supportXX`` flags.

Adding Extra Methods
====================

Sometimes capabilities of the backend are much more extensive than the described
capabilities. For example, MongoDB supports aggregation. Controller should
provide some unique interface to those features for developer.

Any developer who is willing to write portable code will rely on the above
capabilities, however if developer is certain that only specific database will
be used, he can rely on extra methods provided by controllers. You can
define those methods in ``setSource`` method::

    function setSource(...){
        ...
        $this->model->addMethod(['aggregate', 'collection']);
    }
    function aggregate($model, $arg1, $arg2){
        ... // some complex logic
    }
    function collection($model){
        return $model->_table[$this->name];
    }












