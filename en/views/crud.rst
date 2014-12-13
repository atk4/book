CRUD
====

The CRUD is an acronym fro Create Read Update Delete - universal editing solution
for structured data.

Agile Toolkit approaches CRUD with the use of compositions and abstracts away
all the complexity::

    $this->add('CRUD')->setModel('Book');

This will display a list of Books as well as buttons to add new book, edit individual
book or delete a book.

.. php:class:: CRUD

    View allowing user to add, edit, and delete records in a table.

CRUD however goes beyond this and adds more features:


- CRUD can operate in list, edit, delete and add modes but you can add more modes to CRUD
- Allows addition of custom actions either as a column-action or toolbar action
- Add frame triggered by the button for any cutsom UI code
- Extend existing editing / adding functions
- Add expanders for traversing between model relations

Using advanced CRUD features
----------------------------


.. php:method:: setModel($model, $default_fields, $grid_fields)

    Associates CRUD (and contained Grid and Form) with a model. If default default_fields
    property is specified, it's used for both Grid and Form as :ref:`Actual Fields`.
    If you also specify grid_fields, then default_fields will be used for model only.

    CRUD accepts extra argument for setModel - if third argument is specified then
    instead of $default_fields it will be used as actual field for the Grid mode.

    If you would like to make "name", "details" fields editable but only show "name"
    in grid mode, use::

    $this->setModel('User', ['name','details'], ['name']);

.. php:method:: addRef($relation, $optoins)

    Assuming that your model has a $relation defined, this method will add a button
    into a separate column. When clicking, it will expand the grid and will present
    either another CRUD with related model contents (one to many) or a form preloaded
    with related model data (many to one).

The format of $options is the following::

    [
      'view_class' => 'CRUD',  // Which View to use inside expander
      'view_options' => ..     // Second arg when adding view.
      'fields' => array()      // Used as second argument for setModel()
      'extra_fields' => array() // Third arguments to setModel() used by CRUDs
      'label'=> 'Click Me'     // Label for a button inside a grid
    ]

.. php:method:: addFrame($name, $options)

    Adds button to the crud, which opens a new frame and returns page to
    you. Add anything into the page as you see fit. The ID of the record
    will be inside $crud->id


.. php:method:: addAction($method_name, $options)

    Assuming that your model contains a method $method_name, this allows
    you to create a button, which would allow user to execute that method.
    By default the button will be present as a column, click-able for
    any records (just like edit or delete).

When clicked, it will automatically load the model record and execute
the action. If method requires arguments, then user will see a form
prompting for the arguments to supply to the method.

$options can be set to 'toolbar', 'column' or be an array::

    [
      'descr'=>'Trigger Manual Recalculation', // will appear on toolbar
      'icon'=>'heart', // icon to use on buttons
      'toolbar'=>true, // should the button be in the toolbar (no record preloading)
      'column'=>true,  // should the button be in the column (each record)
    ]


.. php:method:: addButton($label)

    Will add a button to Grid's toolbar. Returns Button object (or other class
    which you can specify as 2nd argument). Label is localized.


Methods to override
-------------------

Some methods of CRUD are well suitable for overriding:

.. php:attr:: allow_add

    Boolean property allowing CRUD to add new records.

.. php:method:: configureAdd($fields)

    Will populate $crud->model into $crud->form. Used to display
    "Add" form. Also will add "Add" button into toolbar. Called
    if allow_add is true.

.. php:attr:: allow_edit

    Boolean property allowing CRUD to edit existing records.

.. php:method:: configureEdit($fields)

    Will populate $crud->model into $crud->form. Used to display
    "Edit" form. Also will add editing column into grid.

.. php:attr:: allow_del

    Boolean property allowing CRUD to delete existing records.

.. php:method:: configureDel

    Will implement the column for deleting records and handle
    deletion.

.. php:method:: configureGrid($fields)

    Will populate $crud->model into $crud->grid. Used to display
    the default listing interface.


.. php:method:: formSubmit($form)

    Called after on post-init hook when form is submitted.

.. php:method:: formSubmitSuccess($form)

    Returns JavaScript action which should be executed on form successfull
    submission.

If you need further control over the default form and grid, there are two
more properties you can change:

.. php:attr:: grid_class

    By default, CRUD will simply use "Grid" class, but if you would like
    to use your custom grid class for listing, specify it inside associative
    array as second argument to add()

.. php:attr:: form_class

    By default, CRUD will simply use "Form" class for editing and adding,
    but if you would like to use your custom form, specify it inside
    associative array as second argument to add()

.. php:attr:: form

    Points to a form object. use isEditing() instead.

.. php:attr:: grid

    Points to a grid object.

.. php:attr:: virtual_page

    Points to :php:class:`VirtualPage` object used in dialogs.

.. php:attr:: frame_options

    When clicking on EDIT or ADD the frameURL is used. If you want to pass
    some arguments to it, put your hash here.


Understanding CRUD modes
------------------------


Agile Toolkit is a clever framework and often it might appear that things
happen magically. For instance CRUD being able to handle callbacks right inside
itself somehow.

In reality CRUD operates in various modes and it will either populate a Grid
when "listing" or Form when "editing". After initializing CRUD and setting model,
properties ``form`` and ``grid`` will either point to a proper object or to a
:php:class:`Dummy` object::

    $cr = $this->add('CRUD');
    $cr->setModel('Book');

    echo get_class($cr->form);

This will output ``Dummy`` when you observe grid, however if you click on add
or edit, this will use a Form class instead.

You can read the mode with isEditing() method:

.. php:method:: isEditing($mode = null)

    Will return if CRUD is in the editing mode. When argument is not specified
    method will return ``false`` in Grid mode and ``true`` for any other mode
    (add, edit or any action)


    When mode is specified, then ``true`` is returned only if that mode is active.

If we assume you are adding a new record::

    $crud->isEditing();        // will return true
    $crud->isEditing('edit');  // will return false
    $crud->isEditing('add');   // will return true

Features such as Actions create additional modes for the grid.

Tracking Record ID
------------------

If you are in the editing mode, then you can access "id" property of a CRUD:

.. php:attr:: id

    Will contain ``null`` or the ID of the record where user have clicked.

So clicking on Add button will put grid into isEditing() / add mode, yet it
will not set ``id``.

The record is further populated into :php:attr:`CRUD::virtual_page` proprety::

    $crud->addFrame('click me')->set(funciton($p) {
        // $p is a virtual page

        $m=$p->owner->model;
        // $m is CRUD model

        $m->load($p->id);
        // Load corresponding record

        $p->add('H1')->set('Hello, '. $m['name']);
    });

Example with Dynamic Model Method
---------------------------------

Agile Toolkit supports :ref:`Dynamic Methods`. This allows us to create a
call-back powered method in model, then use addAction of grid.

Because Grid attempts to use ``Reflection`` to learn more about the
model and we will be a dynamic model which does not have reflection,
we will need to specify arguments manually::

    $crud = $this->add('CRUD');
    $m = $crud->setModel('Book');

    $m->addMethod('borrow', function($m) {
        return 'Borrowing '.$m['name'];
    });

    $crud->addAction('borrow', [ 'args' => [], 'toolbar' => false ]);

This will add "Borrow" column to Grid and when clicking will respond with
returned text.

Actually because list of arguments is empty anyway - you could have ommitted it::

    $crud->addAction('borrow', [ 'toolbar' => false ]);

Finally, we could have used a string as second argument::

    $crud->addAction('borrow', 'column');

Rewriting the example more compactly we get this::

    $crud = $this->add('CRUD');
    $crud->setModel('Book');
        ->addMethod('borrow', function($m) {
            return 'Borrowing '.$m['name'];
        });

    $crud->addAction('borrow', 'column');

.. tip:: You must rememeber that this is just example. In real application, you
    should keep your business logic inside a proper model methods.
