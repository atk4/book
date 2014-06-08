Event Hooks
===========

What Are Hooks?
---------------

Hooking is `a well know development
pattern <http://en.wikipedia.org/wiki/Hooking>`__ that enables you to
register callbacks with an object.

This is very similar to the "bind" and "trigger" methods in jQuery. In
other frameworks hooks may be called "actions", "handlers" or
"callbacks".

In Agile Toolkit, hooks are implemented in ``AbstractObject`` so all
classes have access to hook functionality.

If you're not familiar with this pattern, here's the basic idea...

First, an object offers a "spot" where you can register callback code.
So in ``Model_Table``, for example, you'll find:

::

    function save()
    {
        $this->hook('beforeSave');
        // ...
    }

Then in your own code you might have:

::

    $model->addHook('beforeSave', function($m) {
        $m['card_mask'] = substr($m['card_num'], -4);
    });

The Model class's ``save()`` method will now automatically call your
callback method.

You can place hook anywhere in your code, even inside Model's ``init()``
method:

::

    class Model_Customer extends Model_Table {
        function init(){

            // ...

            $this->addHook('beforeSave',function($m){
                $m['updated_at'] = date('Y-m-d G:i:s', time());
            });
        }
    }

You can add more than one hook to the same spot, and by default they run
in the order they were registered.

This technique enables you to add behaviours to a class without
overriding methods. Many objects in the Toolkit offer hooks, and you can
use this functionality to add flexibility to your own objects as well.

How To Create A Hook Spot
-------------------------

Adding a hook spot to your class is as simple as calling
``$this->hook('hookName')`` where you want the callback handlers to run.

::

    $this->hook('failedTransaction');

How To Add Hooks To A Spot
--------------------------

You hook your callback code into a hook spot with ``addHook()``:

::

    addHook($hook_spot, $callable, $arguments = array(), $priority = 5)

The first argument is the name of the spot, and the second is a callable
PHP type that will be called when the hook runs.

As we've seen, the typical callable is a closure:

::

    $this->addHook('beforeModify', function($m){
        // ...
    });

You can also pass in an array, with the callback handler object as the
first value, and the name of the method to be called on the handler as
the second value.

::

    function addRequest()
    {
        // ...

        $this->addHook('requestComplete', array($this, 'requestComplete'));
    }

    function requestComplete($request, $response)
    {
        // ....
    }

Finally, if you pass in an object rather than an array, the hook will
call a method on the handler with the same name as the hook spot:

::

     $this->addHook('requestComplete', $this);

     // ...is shorthand for:

     $this->addHook('requestComplete', array($this, 'requestComplete'));

Hook Arguments
--------------

Both ``hook()`` and ``addHook()`` can pass arguments to the callback
handler.

The object containing the hook spot always passes itself into your
callback handler as the first argument:

::

    $my_obj->addHook('beforeModify', function($my_obj){

        // ...
    });

The ``hook()`` method can pass values to the callback handler as an
array in its second argument:

::

    $val = 'test-1';

    $this->hook('test', array($val, 'test-2'));

The arguments set in the array will be passed to the callback handler in
order, starting with the second argument:

::

    $obj->addHook('test', function($obj, $first_arg, $second_arg){

        // Outputs "test-1 :: test-2"
        echo("$first_arg :: $secong_arg");
    });

The ``addHook()`` method can pass additional arguments as an array in
its 3rd argument. These are passed to the callback handler after the
arguments passed by ``hook()``;

::

    $val = 'test-3';

    $obj->addHook('test', function($obj, $first_arg, $second_arg, $third_arg, $fourth_arg){

        // Outputs "test-1 :: test-2 :: test-3 :: test-4"
        echo("$first_arg :: $secong_arg :: $third_arg :: $fourth_arg");

    }, array($val, 'test-4'));

If you add the same handler to multiple objects, the first argument
passed to the handler always points to the object that owns the hook.

::

    $model1->addHook('test', $obj);
    $model1->api->addHook('test', $obj);

    $model->hook('test');               // Executes $obj->test($model1);
    $otherobject->api->hook('test');    // Executes $obj->test($api);

Hook priorities
---------------

By default callbacks are assigned a priority of 5 and are executed in
the order they are assigned. You can change the order by passing a
different priority as the 4th argument of ``addHook()``. Lower numbers
are executed first.

::

    // Outputs: "2 def 10"
    $obj->addHook('test',function(){ echo "def "; }); // priority 5
    $obj->addHook('test',function(){ echo "2 "; }, null, 2);
    $obj->addHook('test',function(){ echo "10 "; }, null, 10);

    $obj->hook('test');

If you specify a negative priority then hooks are executed in reverse
order:

::

    // Outputs: rev2 rev1 def1 def2
    $obj->addHook('test',function(){ echo "def1 "; });
    $obj->addHook('test',function(){ echo "def2 "; });
    $obj->addHook('test',function(){ echo "rev1 "; }, null, -3);
    $obj->addHook('test',function(){ echo "rev2 "; }, null, -3);

    $obj->hook('test');

Returning hook values
---------------------

Method ``hook()`` will return an array containing the values returned by
all the handlers added, listed in the order they were called. If at
least one hook executed calling ``hook()`` will return a non-empty
array, so you can also use this syntax to detect if there were any
hooks:

::

    if( ! $this->hook('test')){

        echo "No hooks!";
    }

Here is an example where we call multiple hooks and use the returned
values:

::

    $obj->addHook('foo', function($o){ return 1; });
    $obj->addHook('foo', function($o){ return 2; });

    $res = $obj->hook('foo'); // Returns array(1, 2);

Breaking The Chain Of Execution
-------------------------------

It's possible for a callback to stop execution of further call-backs.
When called from inside a hook callable, ``breakHook()`` will break the
chain of execution, and the ``hook()`` method will return the value
passed to ``breakHook()``:

::

    $obj = $this->add('MyClass');

    $obj->addHook('foo',function($o){ return 1; });

    $obj->addHook('foo',function($o){

            $o->breakHook('override-value');
        });

    $obj->addHook('foo',function($o){ return 2; });

    $res = $obj->hook('foo'); // returns 'override-value';

Removing All Hooks From A Spot
------------------------------

To remove all hooks from a spot:

::

    $obj = $this->add('MyClass');

    $obj->addHook('foo', function($o){ return 1; });
    $obj->addHook('foo', function($o){ return 2; });

    $obj->removeHook('foo');
    $res = $obj->hook('foo'); // returns array();

Useful Hooks
------------

Here are some of the most commonly used hooks, to give you an overview
of what's on offer:

Model Hooks
~~~~~~~~~~~

In many applications the most common use of hooks is in Models:

-  **beforeLoad(\ :math:`model, `\ query)** – called before an SQL
   SELECT query is executed. This is called for both ``$model->load()``
   and for iterating through a result set with foreach($model). This
   hook is handy for applying extra options to your SQL query.

-  \*\*afterLoad(\ :math:`model)** &ndash; called after data has been loaded from SQL. You can now access model data with ``\ model->get()\ ``. Called for both``\ model->load()\`\`
   and iterating. This hook is great for performing data manipulation
   and normalization.

-  \*\*beforeSave(\ :math:`model)** &ndash; called when ``\ model->save()\`
   is called. The hook runs inside an SQL transaction, so database
   changes you perform here will be rolled back if the save is
   unsuccessful. This hook is used for performing data modification
   before it's been saved.

-  **beforeInsert(\ :math:`model, `\ query)** – called after the insert
   query object has been created, but before it executes. The query is
   passed as the 2nd argument, and you can change it in your callback
   handler.

-  **afterInsert(\ :math:`model,`\ id)** – called after an insert
   succeeds, but before the Model is re-loaded. You can break out of
   this hook and return a substitute model. Used for overriding how a
   Model is reloaded after an insert.

-  **beforeModify(\ :math:`model,`\ query)** – called before an UPDATE
   SQL query is executed. This hook is great for changing update query
   options.

-  **afterModify($model)** – called after an SQL query is executed but
   before reloading. Note that if you access set() / get() here the
   Model will be reloaded.

-  afterSave($model)\*\* – called after a successful save and reload.
   This is the last hook to execute before the SQL transaction is
   committed. Please note that ``beforeLoad`` and ``afterLoad`` will
   also be called during the reloading of a Model. This hook is great
   for hiding fields from a Model after saving, such as wiping your
   password field.

The other Model hooks work in a similar way:

-  beforeUnload($model)
-  afterUnload($model)
-  beforeDelete(\ :math:`model, `\ query) &ndash you can access the
   record id through $model->id
-  afterDelete($model)
-  beforeDeleteAll($model)
-  afterDeleteAll($model)

Note: Some of those hooks will not supply $query if used in non-SQL
Models.

Useful Hooks
------------

Here are some of the most commonly used hooks, to give you an overview
of what's on offer:

Model Hooks
~~~~~~~~~~~

In many applications the most common use of hooks is in Models:

-  **beforeLoad(\ :math:`model, `\ query)** – called before an SQL
   SELECT query is executed. This is called for both ``$model->load()``
   and for iterating through a result set with foreach($model). This
   hook is handy for applying extra options to your SQL query.

-  \*\*afterLoad(\ :math:`model)** &ndash; called after data has been loaded from SQL. You can now access model data with ``\ model->get()\ ``. Called for both``\ model->load()\`\`
   and iterating. This hook is great for performing data manipulation
   and normalization.

-  \*\*beforeSave(\ :math:`model)** &ndash; called when ``\ model->save()\`
   is called. The hook runs inside an SQL transaction, so database
   changes you perform here will be rolled back if the save is
   unsuccessful. This hook is used for performing data modification
   before it's been saved.

-  **beforeInsert(\ :math:`model, `\ query)** – called after the insert
   query object has been created, but before it executes. The query is
   passed as the 2nd argument, and you can change it in your callback
   handler.

-  **afterInsert(\ :math:`model,`\ id)** – called after an insert
   succeeds, but before the Model is re-loaded. You can break out of
   this hook and return a substitute model. Used for overriding how a
   Model is reloaded after an insert.

-  **beforeModify(\ :math:`model,`\ query)** – called before an UPDATE
   SQL query is executed. This hook is great for changing update query
   options.

-  **afterModify($model)** – called after an SQL query is executed but
   before reloading. Note that if you access set() / get() here the
   Model will be reloaded.

-  afterSave($model)\*\* – called after a successful save and reload.
   This is the last hook to execute before the SQL transaction is
   committed. Please note that ``beforeLoad`` and ``afterLoad`` will
   also be called during the reloading of a Model. This hook is great
   for hiding fields from a Model after saving, such as wiping your
   password field.

The other Model hooks work in a similar way:

-  beforeUnload($model)
-  afterUnload($model)
-  beforeDelete(\ :math:`model, `\ query) &ndash you can access the
   record id through $model->id
-  afterDelete($model)
-  beforeDeleteAll($model)
-  afterDeleteAll($model)

Note: Some of those hooks will not supply $query if used in non-SQL
Models.

Application Hooks
~~~~~~~~~~~~~~~~~

.. raw:: html

   <!--x Remember to add aliases for the new hook names! x-->

These enable you to run code at specific points in the execution flow:

-  Commonly used hooks in ApiCLI

   -  **caughtException(api,exception)** – called when your application
      generates an exception. This is used by the Logger Controller.

   -  **outputWarning, outputDebug($msg)** can be used for error logging
      or to format error output. The Logger Controller uses both of
      these hooks.

   -  **localizeString($str)** – sed to register a localization
      mechanism. All the texts in Agile Toolkit (such as labels) are
      passed through this hook, which must return a localized version.

-  Commonly used hooks in ApiWeb

   -  **preInit, preExec, postSubmit, preRender, postJsCollection,
      preRenderOutput & postRenderOutput** – these hooks are always
      executed at the relevant point in the execution flow. For example
      if you want to output how many database queries executed during an
      Application run, you would output it on 'PostRenderOutput'.

   -  **cutOutput, submitted** – these are conditional hooks.
      ``cutOutput`` is called when only part of the page is being
      redrawn. You can use it to stop execution timer, as
      postRenderOutput wouldn't be called. The ``submitted`` hook is
      called when POST data is received from any Form. (The Form object
      actually uses this hook to process POST data).

.. todo: Review the following section, as it came from different book.

Hooks
~~~~~

Hooks is a callback implementation in Agile Toolkit. Hooks are defined
throughout core classes and other controllers can use them to inject
code.

For example:

::

    // Add gettext() support into the app
    $this->api->addHook('localizeString',function($obj,$str){
        return _($str);
    });

The localizeString hook is called by many different objects and through
adding a hook you can intercept the calls.

::

    $obj = $this->add('MyClass');
    $obj->addHook('foo',function($o){ return 1; });
    $obj->addHook('foo',function($o){ return 2; });
    $res = $obj->hook('foo'); // array(1, 2);

This example demonstrates how multiple hooks are called and how they
return values. You can use method breakHook to override return value.

::

    $obj = $this->add('MyClass');
    $obj->addHook('foo',function($o){ return 1; });
    $obj->addHook('foo',function($o){ $o->breakHook('bar'); });
    $res = $obj->hook('foo'); // 'bar';

You should have noticed that all the hook receive reference to $obj as a
first argument. You can specify more arguments either through hook() or
addHook()

::

    $obj->addHook('foo',function($o,$a,$b,$c){
        return array($a,$b,$c);
    }, array(3));
    $res = $obj->hook('foo',array(1,2)); // array(array(1,2,3));

When calling addHook() the fourth argument is a priority. Default
priority is 5, but by setting it lower you can have your hook called
faster.

::

    $obj = $this->add('MyClass');
    $obj->addHook('foo',function($o){ return 1; });
    $obj->addHook('foo',function($o){ return 2; },3);
    $res = $obj->hook('foo'); // array(2, 1);

Note: in this example, the "3" was passed as 3rd argument not fourth.
addHook automatically recognize non-array value as a priority if array
with argments is omitted. **Argument** omission is often used in Agile
Toolkit methods.

When you are building object and you wish to make it extensible, adding
a few hooks is always a good thing to do. You can also check the
presence of any hooks and turn off default functionality:

::

    function accountBlocked(){
        if(!$this->hook('accountBlocked'))
            $this->email('Your account have been blocked');
    }

Without any hooks, hook() will return empty array.

Finally you can call removeHook to remove all hooks form a spot.

::

    $obj = $this->add('MyClass');
    $obj->addHook('foo',function($o){ return 1; });
    $obj->removeHook('foo');
    $res = $obj->hook('foo'); // array();

Note: If your object implements handlers for a few hooks and sets them
inside init(), then after cloning such an object, it will not have the
handlers cloned along with the object. Use of newInstance() should work
fine.

d

