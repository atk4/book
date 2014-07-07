Grid
====

.. php:class:: Grid

What is Grid?
-------------

.. figure:: /figures/grid1.png
   :alt: Grid Sample Image

   Grid Sample Image
Grid is a visual element in Agile Toolkit which is designed for
presenting multi-column data. Relying on a basic HTML TABLE markup, the
main purpose of a grid is to give you a simple and usable way of
presenting data to the user.

As you see more features and want to have a fancier display of data, you
might find Grid un-suitable to accommodate your layout - look further
into `Complete Lister <../lister/overview.md>`__.

What Grid can and cannot do
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Grid always consists of columns. Usually one model's field per column.
Grid is capable of sorting by column as well as letting you format
columns.

Grid relies on a HTML TABLE and flexibly adjusts itself to full width of
HTML container. Grid does not rely on JavaScript for presenting itself.
While there are some enhancements which allow you to edit data directly
inside Grid, it's not a primary purpose of a Grid. Grid is designed to
display one-dimensional data, it does not support tree-structure,
however you can use "Expander" to contain Grid inside another Grid.

Zebra Grid
^^^^^^^^^^

For a slight polish on a Grid's look and feel, you can add class "zebra"
to your grid. This will produce a slightly different look: |Grid Zebra
Image|

Implementation
~~~~~~~~~~~~~~

Grid extends CompleteLister View, which gives Grid the ability to repeat
a certain portion of HTML continuously. When grid is being rendered, it
first prepares it's "row" template. This ultimately impacts how all
those rows will look. The contend of a rows template is determined by
the list of columns you are willing to display on the grid.

Furthermore Grid iterates through it's data source and stores values of
each row inside ``current_row`` property then by calling ``formatRow``
and ``format_<datatype>`` on each column data, it converts values inside
HTML presentation inside ``current_row_html``. The resulting set of data
is inserted into our row template to be rendered once again and appended
to the body of the table. You can still access ``$this->model`` although
this object will not be defined when Grid is used without model.

As a result, several behaviors are observed:

-  Values placed for current row but missing from within the next row
   will keep appearing inside row template. You should must make at
   least pass NULL as a value of a ``current_row`` hash, to clear out
   previous values.
-  Unless you specifically want to output safe HTML, you should store
   your value inside ``current_row`` property - this property is
   automatically escaped when the row is rendered.
-  Avoid `Row-based Sub-Selects <../performance.md>`__ when you render
   and try not to traverse model fields.

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Features                                                                                                                                                                                                                                                                                                                                                     |
+==============================================================================================================================================================================================================================================================================================================================================================+
| Grid implementation is split into 2 classes: Grid\_Basic and Grid\_Advanced. The basic portion implements all the core functionalities of the grid, while Advanced adds more formatters and integrations with other views and extensions. You should always use ``Grid`` class which inherits ``Grid_Advanced`` unless you really know what you are doing.   |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

-  Grid automatically selects all ``visible`` fields of the model.
-  When using ``setModel()`` you can override list of field and order in
   which they are displayed.
-  Automatically assigns appropriate formatters for your fields.
-  ``addPaginator`` integrates ``Paginator`` which breaks results into
   pages with defined number of rows.
-  ``addQuicksearch`` adds ``QuickSerarch`` form in the corner for
   filtering result by one of the specified columns or using model's
   custom ``like()`` function.
-  Allows you to use custom ``Iterator`` with ``setModel`` but you would
   need to call ``addColumn`` manually.
-  ``addFormatter`` can use multiple formatters per field, e.g. "money"
   and "nowrap".
-  Scalable: grid does not create object or perform queries on every
   row.
-  automatically keeps up ``totals`` which can be appended as yet
   another row at the bottom of grid.
-  ``sortable`` grids automatically get sorting control.
-  ``addButton`` is a wrapper for adding buttons into button set on top
   of grid.
-  handles situations with no records.
-  Change TD-styling with ``setTDParam``.
-  Support for Column add-ons.
-  integrates with ``selectable`` and ``ui.atk4_checkboxes`` with
   ``addSelectable``.

Obviously Grid inherits all the features of the View as well as you can
manipulate ``$grid->model`` in any way model can be manipulated making
it possible to change styling, positioning, size and rendering of a
grid, conditions, order or other properties of a select query.

How to create and use Formatters?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

While most situations can be solved with generic grid, it does need to
display data in a reasonable formats. But how does Grid determine a type
of your column?

Field ``type()``
^^^^^^^^^^^^^^^^

Model's field ``type()`` method can be used to specify the type of the
column. There are defined set of supported types, and the primary
purpose of this setting is to define how data is stored.

When you call ``grid->setModel`` it creates a new object -
``Controller_MVCGrid`` which is responsible for populating columns
inside Grid and also matches data-types into grid column types. For
example ``list``, ``int`` type is displayed as ``text``, ``money`` uses
formatter also named ``money`` and ``text`` is displayed using
``shorttext`` formatter, which shows just a fragment of your long text
fields.

If you want to implement your own type and it's associations, look into
`Creating your own Grid Controller <TODO>`__

Field ``display()``
^^^^^^^^^^^^^^^^^^^

Allows you to specify formatter exactly. Does not affect the behavior of
the model. Can either target all views with ``type('password')`` or only
grid with ``type(array('grid'=>'password'))``.

You can specify a display type either inside your model definition or
inside presentation logic, like this:

::

    $m=$this->add('Model_Book');
    $m->getElement('author')->display('link');
    $this->add('Grid')->setModel($m);

addColumn()
^^^^^^^^^^^

This method adds new column to your grid. Even if you are using Model,
you can still add few extra columns and use custom formatters.

Formatter needs to be defined as a method of a grid.

::

    class MyGrid extends Grid {
        function format_smiley($field) {
            $this->current_row[$field] =
                str_replace(':)','☺',$this->current_row[$field]);
        }
    }
    $grid=$this->add('MyGrid');
    $grid->addColumn('smiley','my field');

.. |Grid Zebra Image| image:: /figures/grid2-zebra.png



.. php:method:: addPaginator

This implements a more convenient way to add :php:class:`Paginator` inside Grid. Example::

    $g = $this->add('Grid');
    $g ->setModel('People');
    $g ->addPaginator();

.. php:method:: addQuickSearch

Wrapper for adding :php:class:`QuickSearch`.

    $g = $this->add('Grid');
    $g ->setModel('People');
    $g ->addQuickSearch(['name', 'surname']);

