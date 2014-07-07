****
Grid
****

.. php:class:: Grid

What is Grid?
=============

.. figure:: /figures/grid2-zebra.png

Grid is a View of Agile Toolkit which allows presentation of multi-column data in a table with columns. The main purpose of a Grid is to give you a simple and usable way of presenting data to the user. When you build your application we recommend to start by using Grid initially whenever you need to list your data then replace it with :php:class:`Lister` and :php:class:`CompleteLister` with custom templates.

Adding grid is very simple::

  $grid = $this->add('Grid');
  $grid->setModel('Book');

.. php:method:: importFields

.. note::
    Grid implementation is divided into 2 classes: Grid_Basic and Grid_Advanced. The basic portion implements all the core functionalities of the grid, while Advanced adds more formatters and integrations with other views and extensions. You should always use ``Grid`` class which inherits
    ``Grid_Advanced`` unless you really know what you are doing.

Grid internally is broken down into 2 classes: Grid_Basic and Grid_Advanced but because they are interlinked we will simply refer to them both with Grid
class. (Grid extends Grid_Advanced and Grid_Advanced extends Grid_Basic).

As with most objects, you can use Grid with Model or with generic source. The use of setModel will also populate visible columns (see :ref:`actual fields`)

Adding Columns to Grid
======================

.. php:method:: addColumn
.. php:attr:: columns
.. php:method:: hasColumn
.. php:method:: removeColumn


This method adds new column to your grid. Even if you are using Model, you can still add few extra columns and use custom formatters.

Here is how you can create Grid for Custom Data::

  $data = [
    [ 'name' => 'John', 'surname' => 'Smith' ],
    [ 'name' => 'Joe', 'surname' => 'Blogs' ],
  ];

  $g = $this->add('Grid');
  $g->addColumn('name');
  $g->addColumn('surname');
  $g->setSource($data);

Specifying column formatters
----------------------------

Each column of a Grid can have one or several formatters. Formatters are designed to change the look of the field somehow. For example "money"
formatter will convert a regular number field into a standard currency format.

When using addColumn with at least 2 arguments, the first one is considered to be a formatter(s)::

  $g->addColumn('money', 'salary');


.. php:method:: setFormatter
.. php:method:: addFormatter

You can add more formatters to a field or specify both formatters initially::

  $g->addFormatter('salary', 'link');  // add formatter to existing field

  $g->addField('money,link', 'tax');   // add field with two formatters


Using Template Formatters
-------------------------

While there are many interesting formatters, one stands out - "template".

.. php:method:: format_template

Example::

  $g->addColumn('template', 'details')
    ->setTemplate('{$name} {$surname} earning {$salary}')


Ready to use formatters
-----------------------

.. php:method:: format_html

.. php:method:: format_number

.. php:method:: format_real

.. php:method:: format_money

.. php:method:: format_boolean

.. php:method:: format_date

.. php:method:: format_time

.. php:method:: format_datetime

.. php:method:: format_timestamp

.. php:method:: format_fullwidth

.. php:method:: format_nowrap

.. php:method:: format_wrap

.. php:method:: format_shorttext

.. php:method:: format_password

.. php:method:: format_image

.. php:method:: format_checkbox

.. php:method:: format_link


Interractive Columns
--------------------

.. php:method:: format_button

.. php:method:: format_prompt

.. php:method:: format_confirm

.. php:method:: format_delete


Defining your own formatters
----------------------------

If there is no formatters that you like you can extend Grid class and add it::

    class MyGrid extends Grid {
        function format_smiley($field) {
            $this->current_row[$field] = str_replace(':)', '☺', $this->current_row[$field]);
        }
    }

When you will now be using Grid, you can use smiley formatting to substitute smileys with unicode character::

    $grid = $this->add('MyGrid');
    $grid->addColumn('smiley', 'my field');


Inside your formatter you can access two properties of ``$this``:

- :php:property:`Lister::current_row` - each formatter must alter key of this hash.
- :php:property:`Lister::current_row_html` - if key exist for the column, it will be
  used instead, however no HTML escaping will be done.
- :php:property:`AbstractObject::model` - you can access this to get un-formatted values.

Example which makes column bold::

    function format_bold($field) {
        $this->current_row_html[$field] = '<b>'.
            ($this->current_row_html[$field] ?: htmlsecialchars($this->current_row[$field]))
            '</b>';
    }

.. php:method:: ssetTDParam

Using the setTDParam method it's possible to implement bold without the extra element::

    function format_bold($field) {
        $this->setTDParam($field, 'class', 'atk-text-bold');
    }

There are two optional methods for each formatters prefixed with ``format_totals_`` and ``init_``. The init_ is called when the field is initialized initially and only once. The format_totals_ is called when totals column needs to be formatted after regular formatter is applied. This is useful for some types to output blank space in totals (such as link, checkbox, etc)

Formatting individual grid
--------------------------

There is also way to format without resorting to the formatters and it's explained in :php:class:`Lister::formatRow`


Other interesting Methods
=========================

.. php:method:: setNoRecordsMessage


.. php:method:: addPaginator

This implements a more convenient way to add :php:class:`Paginator` inside Grid. Example::

    $g = $this->add('Grid');
    $g->setModel('People');
    $g->addPaginator();

.. php:method:: addQuickSearch

Wrapper for adding :php:class:`QuickSearch`.

    $g = $this->add('Grid');
    $g->setModel('People');
    $g->addQuickSearch(['name', 'surname']);


Expanders
=========

.. php:method:: format_expander

Expander is a special formatter (``format_expander``), which will create a clicable column. If format is added to existing column, then the original
text will be placed on the expanding button.

The work of Expander relies on ui.atk5_expander.js widget, which will add additional row after the clicked one and load data through AJAX.

Exander will open a sub-page of a current page with the same name as the column name::

    function page_book(){
        $grid = $this->add('Grid');
        $grid->setModel('Book');
        $grid->addColumn('expander', 'details');
    }

    function page_book_details(){
        $this->add('View_ModelDetails')
             ->setModel('Book')
             ->load($this->app->stickyGET('book_id'));
    }

As you can see in the example, the expander will also pass the row ID inside an two arguments:

- $GET['id']
- $GET[$table_name.'_id'];

While in most cases you would want just to grab the ``id``, if you have a nested expanders, you will need to use the second format. The table name is taken from :php:attr:`Model::table` property.

We have also used :ref:`stickyGET` just in case my newly opened page will want to perform further reloading.

Making Sortable Fields
======================

You might have already noticed that if you use :php:meth:`Field::sortable` in your model definition, then Grid gains ability to sort by this column.
The sorting is turned off by default, but it's relatively easy to add it. 

We recommend you to always use the Field sortable flag, however you might want to know that there is also a method:

.. php:method:: makeSortable

Please avoid using it directly.

General Grid Usage Guidance
===========================

While Grid is a very convenient UI element you should always consider having a custom-formatted lists of data using CompleteLister. You can still use
tables and buttons inside CompleteLister, but the customization capabilities of CompleteLister are much higher than Grid.



Grid Limitations
----------------

- Grid always consists of columns.
- Grid outputs one table row per record.


Extra HTML Classes
------------------

You can find full list of table decorator components in :doc:`/css/tables`,
but you can apply them to your Grid like this::

    // select the ones you need
    $grid->addClass('atk-table-zebra');

    $grid->addClass('atk-table-outline');

    $grid->addClass('atk-table-bordered');


Final Implementation Notes
--------------------------

-  Values placed for current row but missing from within the next row will keep appearing inside row template. You should must make at least pass NULL as a value of a ``current_row`` hash, to clear out previous values.
-  Unless you specifically want to output safe HTML, you should store your value inside ``current_row`` property - this property is automatically escaped when the row is rendered.
-  Avoid sub-selects inside your formatters as it will have major performance impact.
-  Grid automatically selects all ``visible`` fields of the model.
-  When using ``setModel()`` you can override list of field and order in which they are displayed.
-  Automatically assigns appropriate formatters for your fields.
-  ``addPaginator`` integrates ``Paginator`` which breaks results into pages with defined number of rows.
-  ``addQuicksearch`` adds ``QuickSerarch`` form in the corner for filtering result by one of the specified columns or using model's
   custom ``like()`` function.
-  Allows you to use custom ``Iterator`` with ``setModel`` but you would need to call ``addColumn`` manually.
-  ``addFormatter`` can use multiple formatters per field, e.g. "money" and "nowrap".
-  Scalable: grid does not create object or perform queries on every row.
-  automatically keeps up ``totals`` which can be appended as yet another row at the bottom of grid.
-  ``sortable`` grids automatically get sorting control.
-  ``addButton`` is a wrapper for adding buttons into button set on top of grid.
-  handles situations with no records.
-  Change TD-styling with ``setTDParam``.
-  Support for Column add-ons.
-  integrates with ``selectable`` and ``ui.atk4_checkboxes`` with ``addSelectable``.

Obviously Grid inherits all the features of the View as well as you can manipulate ``$grid->model`` in any way model can be manipulated making
it possible to change styling, positioning, size and rendering of a grid, conditions, order or other properties of a select query.

.. todo:: move this to :php:class:`Model_Field`

How to create and use Formatters?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

While most situations can be solved with generic grid, it does need to display data in a reasonable formats. But how does Grid determine a type
of your column?

Field ``type()``
^^^^^^^^^^^^^^^^

Model's field ``type()`` method can be used to specify the type of the column. There are defined set of supported types, and the primary
purpose of this setting is to define how data is stored.

When you call ``grid->setModel`` it creates a new object - ``Controller_MVCGrid`` which is responsible for populating columns
inside Grid and also matches data-types into grid column types. For example ``list``, ``int`` type is displayed as ``text``, ``money`` uses
formatter also named ``money`` and ``text`` is displayed using ``shorttext`` formatter, which shows just a fragment of your long text
fields.

If you want to implement your own type and it's associations, look into `Creating your own Grid Controller <TODO>`__

Field ``display()``
^^^^^^^^^^^^^^^^^^^

Allows you to specify formatter exactly. Does not affect the behavior of the model. Can either target all views with ``type('password')`` or only
grid with ``type(array('grid'=>'password'))``.

You can specify a display type either inside your model definition or inside presentation logic, like this:

::

    $m = $this->add('Model_Book');
    $m->getElement('author')->display('link');
    $this->add('Grid')->setModel($m);
