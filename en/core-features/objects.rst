Adding objects
==============

The principle of presenting the Web UI through a nested set of flexible
objects is a fundamental concept in Agile Toolkit.

But the Toolkit is a full stack framework so there are invisible objects
too. For example, when a Form object is submitted, it communicates via
Ajax with a Model object nested inside it. When you add a Model object,
your form knows which fields to display and how to store the data.

.. php:class:: AbstractObject

  A base class for all objects/classes in Agile Toolkit.
  Do not directly inherit from this class, instead use one of
  AbstractModel, AbstractController or AbstractView

.. php:method:: add($class, $options, $template_spot, $template_branch)

  Creates new instance of `$class` as a child.

.. php:method:: init()

  Perform object initialization

.. php:attr: short_name

  Unique object name within parent's $element array

.. php:attr: name

  Unique object name. Consists of $owner->name . "_" . $this->short_name

.. php:attr: elements

  Array containing references to all objects which have been added into
  this class. Instead of some references, there might be "true" value. This
  is to improve work of garbage collector.

.. php:attr: auto_track_element

  If this is true, then owner object will contain reference inside it's
  element array. If false, then "true" will be stored instead. For Views
  this is set to true, so that recursive rendering could be done. For
  Models, this is false, therefore you loose pointer to your model,
  it will be garbage collected.


Direct Adding Of Objects
------------------------


When you add a new object, you use :php:meth:`AbstractObject::add` in
the following form::

    $view = $page->add('LoremIpsum');

Here is what happens next:

#. :php:class:`PathFinder` is used to locate :php:class:`LoremIpsum` class
#. Class `LoremIpsum` is loaded and instance is created using default constructor.
#. Property :php:attr:`AbstractObject::$owner` of a new object is set to point to `$page` object
#. Property :php:attr:`AbstractObject::$app` of a new object is set to point to
   `$page->app`;
#. Property :php:attr:`AbstractObject::$name` and :php:attr:`AbstractObject::$short_name` is set.
#. Owner's :php:attr:`AbstractObject::elements` is updated to contain either a
   link to new object or a `true` value (depending on :php:attr:`auto_track_element`)
#. Other properties passed through 2nd argument of `add()` are set.
#. If a new object is a :ref:`View`, then
  a) :ref:`Template` initialization is taking place and stored in :php:attr:`AbstractView::$template`
  b) :php:attr:`AbstractView::$spot` is set as per 3rd argument of :php:meth:`AbstractObject::add`

#. Hook `$app @ beforeObjectInit` is called.
#. Method :php:meth:`AbstractObject::init` is called for `LoremIpsum`.
#. Hook `$view @ afterInit` of a new object is being called.
#. Reference to new object is returned to and stored in `$view`

When you create a new object, instead of using constructor, you should re-define
init() method instead, because object will be linked with the parent and application
as well as other properties will be already set for your object.


Many objects are designed to reside within parent objects of a certain
type. So if you add an obviously incompable object, such as a Grid
paginator to a database Model, expect to see errors.

Indirect Adding Of Objects
--------------------------

Objects may deﬁne wrapper methods for adding certain types of object –
this `syntactic sugar <http://en.wikipedia.org/wiki/Syntactic_sugar>`__
helps keep code clean and expressive. For example:

-  ``Form`` has a method called ``addField()``
-  ``Grid`` has a method ``addButton()``;

The methods call ``add()`` for you with useful default arguments, and
may take additional arguments which save you from chaining calls. For
example:

::

    $form->addButton('Click Me');

is shorthand for:

::

    $form->add('Button', null, 'form_buttons')->setLabel('Click Me');

Some shorthand methods also allow you to omit part of the class prefix:

::

    $form->add('Field_Line','name');
    $form->addField('Line','name');  // Use this!


Adding Models with setModel()
-----------------------------


.. php:method:: setModel($model_or_class, ..)

  Associates object with supplied model. If string is supplied as first
  argument, it will create instance of this class. The name of the class
  will be :ref:`normalized` by prefixing Model_ if necessary.

  This method sets $object->model (which you can access directly) and
  returns it.

Using ``setModel()`` will have different results in different contexts.
For example adding a Model to a Page object will set the Model data into
the page's template. Adding the same Model to a Grid object will
populate the grid columns with data. Check out each class's
documentation for details.

If you add a Model with ``setModel()``, you can access it through the
parent's ``model`` property, which is useful if you need to reuse it:

::

    // In a Page class

    $grid = $this->add('Grid');
    $form = $this->add('Form');

    $grid->setModel('User');        // Sets the class Model_User
    $form->setModel($grid->model);  // Reuses the same Model object

The first argument of ``setModel()`` is always either a class name or an
existing model object, and in some classes, ``setModel()`` offers
additional arguments.

For example Grid allows you to specify a list of fields to use as
columns as a second argument to ``setModel()``::

    $grid = $page->add('Grid');

    // Define the columns to display
    $grid->setModel('Customer', array('name', 'email', 'zip'));

The CRUD object is similar, but ``setModel()`` accepts two parameters,
listing columns for viewing and columns for editing.

Adding Controllers With setController()
---------------------------------------

.. php:method:: setController($model_or_class, ..)

  Associates controller with model. Will create object if necessary.

In Agile Toolkit an object can use multiple Controllers. Controllers
enhance the functionality of your object.

In most cases using ``$c = add('Controller_Foo')`` is correct. But some
classes are specifically designed to work with pluggable Controllers and
require you to call ``setController('Foo')`` if you need to change the
default. This will be covered in the class's documentation.


Setting Default Properties
--------------------------

In your object, you might set a number of useful properties::

    class View_MyBook extends View {
        protected $cover_color = 'red';

        function init() {
            parent::init();

            echo $this->cover_color;    // outputs 'red'
        }
    }

Agile Toolkit allows you to change the default value of this property, when
you add the object::

    $this->add('View_MyBook', [ 'cover_color' => 'blue' ]);

This approach is a good substitute to passing arguments into a constructor.

.. chaining

Chaining Object Methods
-----------------------

In the true spirit of jQuery, most object methods will return a
reference to themselves (``return $this;``) so you can chain your method
calls::

     $this->add('FormAndSave')
         ->setModel($model)
         ->loadData($this->api->auth->get('id'));

You can also chain calls to existing objects::

    // Configure an existing customer object

    $m_cust->addCondition('is_active', true)
        ->addCondition('account_type', 'trade_1')
        ->loadAny();

In your own classes, it's good practice to add ``return $this;`` to any
method that configures the object, so you can chain your method calls.

Accessing Added Objects
=======================

``AbstractObject`` provides two methods for accessing objects you have
added into a parent object::

    $view = $page->add('View','myview');

    $v = $page->hasElement('myview');    // Returns $view or false
    $v = $page->getElement('myview');    // Returns $view or exception

.. php:method:: getElement($name)

    Looks for an element with specified short_name and returns it. Throws
    exception if not found. Returns `true` if element exists, but is not tracked.

.. php:method:: hasElement($name)

    Looks for an element with specified short_name and returns it. Returns
    `false` if not found. Returns `true` if element exists, but is not tracked.

These are used frequently to customize objects at runtime. Not all
objects will be accessible like that, however. The behaviour depends on
:php:attr:`AbstractObject::auto_track_element`, if it's set to false,
then the reference is not maintained. This is done to help garbage
collector to get rid of those models you have created.

This method is most frequently used to:
- access Form fields
- access Model fields

In other cases it's adised that you keep reference to your object and use
it if you need to access your object later.

Renaming and Moving
-------------------

.. php:method:: rename($new_name)

  Changes name for existing object. Avoid using this.

Agile Toolkit allows you to rename objects, although it's generally not
recommended to rename your objects after you have added them.

You can also move object from one location to another::

  $grid = $this->add('Grid');
  $grid->addPaginator(5);
  $box = $this->add('View_Box');

  // Move paginator from Grid into the Box
  $box->add($grid->paginator);

.. todo::
  Currently this might result in 2 paginators being displayed. Must address.


Destroying Added Objects
------------------------

.. php:method:: destroy()

    Removes object from it's parent and destroys all child objects. After
    calling this, object detructor will be executed when all references
    to the objects are dropped.

:php:class:`AbstractView` is set to track objects when they are added,
this is done to enable recursive pass during rendering. Other objects,
models and controllers will not be tracked automatically. Some classes
such as :php:class:`Field` will override this and will be tracked too::


    function init() {
        parent::init();

        $m = $this->add('Model_Book');

        $this->setModel('Person')->tryLoadAny();
    }

    function render() {

        echo $this->model['name'];   // Shows name of the person

        // Instance of "book" model does no longer exist.

        parent::render();
    }

In this example, instances of two models were created in init() method.
The Book model was destroyed when init() reached it's end, however
the Person model was associated with $this object and was still accessible
in it's render() method.

Here is another example showing the difference::

    $book = $this->add('Model_Book');

    $hello = $this->add('LoremIpsum');

    unset($book);   // will destroy Book
    unset($hello);  // will NOT destroy Lorem, it will still render.

    // $hello->destroy(); unset($hello);
    // Use this instead to destroy LoremIpsum.



But to aid garbage collection, Models can't be accessed. If you call
getElement() to look for a Model, you'll get ``true`` instead of an
object. So to access Models, set a reference into a variable when you
``add()`` it, or use ``$obj->setModel()`` and access the $obj->model
property.

::

    $model = $page->add('Model_Book');
    unset($model);                  // Will destroy $model

    $view = $page->add('View');

    $view->destroy();               // Removes object from parent
    unset($view);                   // Will destroy $view

You don't need to call ``unset()`` if ``$view``\ or ``$model`` is a
local variable inside your method (it will be garbage collected by PHP)
or if you are going to be using it for something else.

Objects With Global Scope
-------------------------

Instead of using PHP's GLOBAL scope, Agile Toolkit gives all objects the
ability to access the Application class through its ``api`` property. If
you want your object to be accessible from any object, add it to the
Application class. This pattern is very similar to how plugins work in
jQuery.

Here's a simple Agile Toolkit app:

::

    include 'atk4/loader.php';

    // Create the API object
    $api = new ApiFrontend();

    // Every object can access the API through the $api property

    $my_object = $api->add('MyClass');
    $my_object->api === $api;            // Is true
    $my_object->api->url('login');       // Using an api object

    // Every object can use any class added to the API

    $api->myclass = $api->add('MyClass2');

    $my_object->api->myclass->doFoo();

Initializing Objects
--------------------

In Agile Toolkit, we don't initialize objects with PHP's
``__construct()`` method. Instead, when you add an object Agile Toolkit
will automatically execute an ``init()`` method in the new object.

This allows us to set properties used by the Runtime Object Tree such as
``owner``, ``api`` and ``name`` before the object is initialized.

Here's a short code extract from the password StrengthChecker Addon. It
checks that you're adding the object to a password field.

::

    class StrengthChecker extends View {

        // This method is always called
        // when the object is created

        function init()
        {
            parent::init();

            if(!$this->owner instance_of Form_Field_Password){

                throw $this->exception('Must be added to a Password field');
            }

            // ....
        }
    }

Locating Your Code Inside Objects
---------------------------------

In addition to the ``init()`` method, any ``render()`` method within a
view will be called as the Runtime Object Tree is rendered.

Here are some rules of thumb:

1. If code is for adding more sub-elements through composability, place
   it inside ``init()``
2. If code needs to iterate through Model data, place it inside a
   ``render()`` method

                                                                                                                                                                               TODO:
                                                                                                                                                                               I
                                                                                                                                                                               don't
                                                                                                                                                                               understand
                                                                                                                                                                               this
                                                                                                                                                                               -
                                                                                                                                                                               setModel()
                                                                                                                                                                               is
                                                                                                                                                                               a
                                                                                                                                                                               method
                                                                                                                                                                               call
                                                                                                                                                                               -
                                                                                                                                                                               how
                                                                                                                                                                               can
                                                                                                                                                                               you
                                                                                                                                                                               put
                                                                                                                                                                               code
                                                                                                                                                                               inside
                                                                                                                                                                               it??

3. If code needs to add more sub-elements but must access database or
   model structure for it - place it inside setModel().

                                                                                                                                                                               TODO:
                                                                                                                                                                               can
                                                                                                                                                                               we
                                                                                                                                                                               clarify
                                                                                                                                                                               please?
                                                                                                                                                                               When
                                                                                                                                                                               is
                                                                                                                                                                               render()
                                                                                                                                                                               called,
                                                                                                                                                                               and
                                                                                                                                                                               when
                                                                                                                                                                               is
                                                                                                                                                                               it
                                                                                                                                                                               not
                                                                                                                                                                               called?
                                                                                                                                                                               I
                                                                                                                                                                               find
                                                                                                                                                                               this
                                                                                                                                                                               confusing
                                                                                                                                                                               as
                                                                                                                                                                               it
                                                                                                                                                                               stands.

Keep in mind that ``init()`` will be called whenever part of your page
is redrawn via AJAX, but ``render()`` won't. If you are executing an
expensive CURL request, for example, it will be much more efficient to
put it inside ``render()``.

Configuring Object Properties
-----------------------------

Many objects have properties with default values. When you are setting
up a new object you can configure it at runtime by passing in an array
of property values as the second argument to ``add()``:

::

    $password->add(
        'StrengthChecker',
        array('default_text' => 'Secure Password Please!')
    );

A common use for properties is overriding a default class name:

::

    // Use CRUD with a custom Grid class

    $page->add(
        'CRUD',
        array('grid_class'=>'MyGrid')
    )->setModel('User');

Where a property is frequently set at runtime, classes often provide a
``set()`` wrapper which you can chain with your ``add()`` call:

::

    $password->add('StrengthChecker')
        ->setDefaultText('Secure Password Please');


Wrappers are also handy when you need to configure a nested object:

::

    $form->setModel('User');
    $form->getElement('email')->setCaption('Your Email');

Cloning Objects & newInstance()
-------------------------------

.. php:method:: newInstance()

  Creates object of same class as this one and add to the same owner. This is
  not same as cloning.


In Agile Toolkit you will frequently be changing your objects after they
are added. For example, you might take your regular Model and add a new
``join`` before using it with a List:

::

    // In a Page or View class

    $book = $this->add('Model_Book');
    $author_join = $book->leftJoin('author');
    $author_join->addField('name')->type('readonly')->caption("Author's Name")

    // Now you can use this Model inside a Grid and it
    // will show the author name for each book

    $this->add('Grid')->setModel($book);

How To Use newInstance()
~~~~~~~~~~~~~~~~~~~~~~~~

If you call ``$book->newInstance()`` it will create a new object of a
same class (``Model_Book``). The new instance, however, will not have
the added author join. This can be useful in some cases when you want to
work with a copy of a model:

::

    >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
    TODO: we need to clarify, I think.  How is
    this different from just using add()?
    >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

    function duplicate()
    {
        return $this->newInstance()->set($this->get())->save();
    }

This example implements a generic duplication method. It will create a
new Model of a same class, copy in the data from the current Model into
new one and call save.

Cloning Objects
~~~~~~~~~~~~~~~

When you use the PHP ``clone`` statement the whole object is duplicated
into an independent variable.

::

    $book_archive = clone $book;

    $book_archive->addCondition('is_archive',true);
    $book->addCondition('is_archive',false);

    $this->add('Grid')->setModel('book');
    $this->add('Grid')->setModel('book_archive');

This code will display two grids - one for regular books and another for
archived. Because objects are cloned, adding conditions to one will not
affect the other.

But be careful – there's a gotcha when you clone hooks.

To continue the example above, say you have a hook inside ``Model_Book``
to check a value before saving:

::

    // In Model_Book

    function init()
    {
        parent::init();

        $this->addField('review');
        $this->hasOne('Author');

        $this->addHook('beforeSave', array($this,'check'));
    }

    function check($m)
    {
        if (strlen($this['review']) < 100) {

            throw $this->exception('Review is too short');
        }
    }


After cloning, ``$this`` will be referencing the wrong object! Saving
our Model with ``$book_archive->save()`` will call ``$book->check()``,
and ``$this`` will validate the value of ``$book`` instead of
``$book_archive``.

You can avoid this problem if you use the Model passed in as ``$m``
instead of ``$this`` inside a hook. In the above example, ``$m`` will
point to ``$book_archive``.

Object Naming
-------------

A call to ``add()`` assigns your new object a unique name within the
Application. This is a useful property whenever you need a unique id
such as for HTML elements (``<div id="...">``), GET arguments or session
values.

Typically Agile Toolkit will append the class name to ``$owner->name``
along with a unique number, but if you specify a string as the second
argument to ``add()`` you can alter part of the name.

::

    // Automatic naming
    $my_object = $api->add('myClass');

    // The name property is unique to the Application
    // and is based on the realm and class name
    $name = $my_object->name;

    // The short_name property is unique to the object within its parent
    $short_name = $my_object->short_name;

    // Manual naming (most often used for fields)

    $my_object = $owner->add('myClass', 'foo');

    echo $my_object->name;          // realm_name_of_owner_foo
    echo $my_object->short_name;    // foo

Object Properties
-----------------

As we have seen, ``AbstractObject`` provides a number of useful
properties to every object in your app. Here's a complete reference:

.. raw:: html

   <table>
       <thead>
           <tr>
               <th>

Property

.. raw:: html

   </th>
               <th>

Access\*

.. raw:: html

   </th>
               <th>

Description

.. raw:: html

   </th>
           </tr>
       </thead>
       <tbody>
           <tr>
               <td>

short\_name

.. raw:: html

   </td>
               <td>

Read

.. raw:: html

   </td>
               <td>

Object name unique to its parent's 'element' array.

.. raw:: html

   </td>
           </tr>
           <tr>
               <td>

name

.. raw:: html

   </td>
               <td>

Read

.. raw:: html

   </td>
               <td>

Object name unique to the entire application.

.. raw:: html

   </td>
           </tr>
           <tr>
               <td>

elements

.. raw:: html

   </td>
               <td>

None

.. raw:: html

   </td>
               <td>

Array containing references to child objects for element tracking. Where
tracking are not required, objects may be 'detached' and their
``elements`` value will be ``true``. This helps conserve memory.

.. raw:: html

   </td>
           </tr>
           <tr>
               <td>

owner

.. raw:: html

   </td>
               <td>

Read

.. raw:: html

   </td>
               <td>

Points to the object which created this object through the call to
``add()``

.. raw:: html

   </td>
           </tr>
           <tr>
               <td>

api

.. raw:: html

   </td>
               <td>

Read

.. raw:: html

   </td>
               <td>

Always points to the application object, the topmost object in the
system

.. raw:: html

   </td>
           </tr>
           <tr>
               <td>

model

.. raw:: html

   </td>
               <td>

Read

.. raw:: html

   </td>
               <td>

Points to Model objects set with ``setModel()``

.. raw:: html

   </td>
           </tr>
           <tr>
               <td>

controller

.. raw:: html

   </td>
               <td>

Read

.. raw:: html

   </td>
               <td>

Points to Controller objects set with ``setController()``

.. raw:: html

   </td>
           </tr>
           <tr>
               <td>

auto\_track\_element

.. raw:: html

   </td>
               <td>

Default

.. raw:: html

   </td>
               <td>

Regulates whether adding this object will automatically add a reference
to the owner's ``elements`` array. If set to ``false``, the object will
be 'detached'

.. raw:: html

   </td>
           </tr>
       </tbody>
   </table>

These properties are declared as ``public`` so that they can be read by
Addons, but it's bad style to change them directly. Here are the methods
you should use to work with these properties:

-  \*\*\ :math:`short_name**, `\ name: ``$obj->rename('new_name')``
-  \*\*\ :math:`elements**: ``\ obj->add();
   :math:`obj->getElement(); `\ child->destroy()\`
-  \*\*\ :math:`owner**: ``\ new\_owner->add($object)\`
-  **$api**: none
-  \*\*\ :math:`model**: ``\ obj->setModel()\`
-  \*\*\ :math:`controller**: ``\ obj->setController()\`
-  **$auto\_track\_element**:
   ``add(.., array('auto_track_element'=>true));``

