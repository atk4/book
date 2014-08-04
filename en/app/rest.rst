********
App_REST
********

Implementing a basic RESTful service for your web application. A routing for App_REST
is slightly different as it does not have pages, but it has interfaces instead.

Requirements
~~~~~~~~~~~~

To create API for your application, you must start by creating a new
:ref:`Application Interface`. A minimum set of files are::

    api/
       index.php
       endpoint/v1/book.php
       lib/Api.php
       config.php

Start by populating::

    class Api extends App_REST {
        function init(){
            parent::init();

            $this->dbConnect();

            $this->add('Controller_PatternRouter')
                ->link('v1/book',array('id','method','arg1','arg2'))
                ->route();
        }
    }

For routing see :php:class:`Controller_PatternRouter`

Normally you can add more things here, depending on your application
requirement. In my example, I want to be sure that database connection
is established.

I am also using standard pattern routing which allows me to use a URLs
like that::

    http://api.example.com/v1/book/12

Next you need to define your end-point. By default they would be inside
"endpoint" folder::

    class endpoint_v1_book extends Endpoint_REST {
        public $model_class = 'Book';
    }

The file "index.php" is pretty much a standard one, although I recommend
adding some headers::

    include'../vendor/atk4/atk4/loader.php';
    include 'lib/Api.php';

    $api = new Api('apisrv');
    $api->main();

And in the config file I would simply include the main configuration
file::

    include_once'../config.php';


More info on :ref:`Configuration`

Testing
~~~~~~~

Open ``http://localhost:8888/api/v1/book`` and it should show you list
produced by your book model.

Enabling or Disabling actions
-----------------------------

By default Agile Toolkit allows you to list entities through the REST.
Normally there are 4 actions available:

::

    GET /book        -- Listing all entries
    GET /book/12     -- Load then list one entry
    POST /book       -- Add new book record
    PUT /book/12     -- Update existing book record (or PATCH)
    DELETE /book/12  -- Delete record

You can also add support for additional actions. Each of the actions
above correspond to the method inside your endpoint. Those are already
defined for you inside ``Endpoint_REST``.

::

     function get()
     function post($data)
     function put($data)
     function delete($data)


When the ID is specified in the URL, the class will automatically
attempt to load corresponding model record returned by
``$this->_model()``

While the above actions are defined by a reference RESTful service
implementations, you may want to enable or disable those for your
endpoint. To do that, there are several properties you can specify with
the following default values:

::

    public $allow_list=true;
    public $allow_list_one=true;
    public $allow_add=false;
    public $allow_edit=false;
    public $allow_delete=false;

Using Methods
-------------

When we defined routing then the segment of the URL was considered to be
a "method":

::

    GET /book/12/sales

When method is passed, then a different method of your end-point class
will be called:

::

    function get_sales()
    {
        return $this->outputMany($this->_model()->ref('Sales'));
    }

This is a handy way to return records relevant to your original entity.
Finally - you may pass additional arguments - arg1 / arg2 as they are
defined in your routing. Here is another sample API call:

::

    GET /book/12/sales/confirmed/123

    function get_sales()
    {
        $m=$this->_model()->ref('Sales');
        if(isset($_GET['arg1'])){
            $m->addCondition('type',$_GET['arg1']);

            if(isset($_GET['arg2'])){
                $m->load($_GET['arg2']);
                return $this->outputOne($m);
            } else {
                return $this->outputMany($m);
            }

            return $this->outputMany($m);
        }

    }

Your methods must always return array which then would be encoded into
the requested format by App\_REST.

Result Output
-------------

To handle output of structured data, you must pass results through
``outputMany`` or ``outputOne`` methods of your end-point, which will
perform output filtering as well as pagination of your data.

Pagination
~~~~~~~~~~

A default API of Agile Toolkit supports optional arguments:

-  limit - specifies how many records to return
-  offset - specifies offset for the first record

Those arguments are automatically considered by ``outputMany()`` method.
If you are producing custom data inside your endpoint methods, we
recommend you to pass them through ``outputMany()`` wrapper to apply
those conditions.

Output filtering
~~~~~~~~~~~~~~~~

Inside your endpoint class you may define some additional rules to limit
input / output for certain methods. For example, if your entity contains
confidential information, such as a password hash, you might want to
exclude it from being returned.

Exclusions are defined inside ``outputOne()`` method, but you may want
to extend it to add additional exclusions:

::

    function outputOne($data)
    {
        $data=parent::outputOne($data);
        unset($data['password']);
        return $data;
    }

Output Formats
~~~~~~~~~~~~~~

By default the JSON is used for output. If you wish to output data in a
different format, you should redefine method ``App_REST::encodeOutput``.

::

    function encodeOutput($data){
        switch($_GET['format']){
            case 'xml':
                echo $this->myGenerateXML($data);
                exit;
            default:
                parent::encodeOutput($data);
                exit;
        }
    }

Authentication and Access Control
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Typically in the multi-user system, your models would belong to a user,
and must not be accessible to anyone else. In your Agile Toolkit
application, there is $app->auth->model which contains the model of
authenticated user.

In APIs some end-points may be public while other end-points may be
user-only. In our example above we didn't specify a User class therefore
no authentication was used.

Using Auth class
^^^^^^^^^^^^^^^^

If you add "Auth" class inside your API then by default authentication
will be required for all end-points. This is similar to the behavior of
Agile Toolkit web application

::

    parent::init();
    $this->add('Auth')->setModel('User');

You do not need to call ``check()`` here. When Auth class is set-up to
bypass the authentication, you should set the property for your
endpoint:

::

    public $authenticate = false;

API will use basic authentication to match username/password.

Access Control
^^^^^^^^^^^^^^

If your model contains field ``user_id`` then endpoint will
automatically set the condition on this field to match authenticated
user. This is done to avoid a possibility of user accessing other user's
records.

If the filed in your model is called differently, you can override it by
setting ``user_id_field`` property. Finally - you can set this property
to ``false`` to turn off automatic conditioning.

Using Custom authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^^

APIs can use other authentication methods, such as token authentication
or secrets. To implement a custom authentication mechanism, redefine
``App_REST::authenticate()`` method.

To customize authentication on per-endpoint basis, redefine
``Endpoint_REST::authenticate()``

Logging Access
^^^^^^^^^^^^^^

If you implement logRequest method in your APP class, then it will be
called to log request. Here is how I approach loging in my application::

    function init(){
        parent::init();

        // Create global model for logging.
        $this->api_log=$this->add('Model_ApiLog')
            ->set('status','init')
            ->save();

        // Saving instantly allows us to have record of ALL the requests
        // even failed ones. You can experement with saveLater here.
    }

    function logRequest($method, $args) {

        // Collect more information about the request
        $this->api_log['interface']=$this->page;
        $this->api_log['status']='request';
        $this->api_log['method']=$method;
        $this->api_log['params']=$args;

        // Don't save right away, wait until we finish executing.
        $this->api_log->saveLater();
    }

    function logSuccess() {

        // Request was completed successfully
        $this->api_log['status']='complete';
    }

    function caughtException($e){

        // When receiving exception, we change request status accordingly
        $this->api_log['status']='exception';
        return parent::caugthException($e);
    }

Like any Agile Toolkit application, you can further extend caugthException
to handle advanced error logging.



Error Control
~~~~~~~~~~~~~

Agile Toolkit relies on PHP exceptions for producing error messages.
Exceptions are raised inside the ``exception()`` method of respective
object. Using the right object can be helpful to define the right error
context.

::

    throw $endpoint->exception('Argument format is invalid');

    throw $app->exception('Database connection failed');

    throw $model->exception('Access to the record is not permitted');

When exception is executed, typically it would result in a 500 Internal
Server Error. If you want to specify a different code, you can do so by
specifying type and/or code.

::

    throw $model->exception('Access to the record is not permitted','AccessDenied',403);

Additional conditioning
-----------------------

If you want to add additional conditions on your model, you can redefine
your ``_model()`` method. This method is executed before the record is
loaded.

::

    function _model()
    {
        return parent::_model()->addCondition('is_active',true);
    }

Input Filtering
---------------

When creating new records or updating existing records, sometimes you
wouldn't want certain fields to be changed.

General Notes on developing end-points
--------------------------------------

Before you start working on your API end-point class, here are some tips
for you:

-  you might create your own common ancestor for your end-point classes,
   especially if you are using versioning.

   ``class endpoint_v1_book extends Endpoint_V1 {``

-  do not put business logic inside your endpoint class. It should be
   inside your model.

-  endpoint is designed to give you full control and flexibility over
   data access. Always think "security!".

-  if you need to share functionality with other parts of your
   application create a "APP Controller" class and initialize it inside
   ``Api`` class.

Different page patterns
~~~~~~~~~~~~~~~~~~~~~~~

Because you are in control of routing, you can design a different
patterns. Usually it's not a good idea to create very deep structures.

Combining data
~~~~~~~~~~~~~~

Because each API request introduces latency to your user application,
sometimes you would want to merge results:

::

    function get()
    {
        if($this->model->loaded()){
            $o=$this->model->get();
            $o['sales']=$this->outputMany($this->model->ref('Sales'),false);
            $o['purchases']=$this->outputMany($this->model->ref('Purchases'),false);
            return $this->outputOne($o);
        }
        return parent::get();
    }

Using without model
~~~~~~~~~~~~~~~~~~~

Endpoint does not necessarily need to have a model. If you don't specify
``$model_class`` you can define your own methods for getting and
updating data.

::

    class endpoint_v1_book extends Endpoint_REST
    {
        function get()
        {
            return 'OK';
        }
    }

.. todo:: allow\_add can also be array. Document!!.

.. todo:: explain that both JSON and FORM data is supported.

