Lister
======

.. php:class:: Lister

    Lister View is a more efficient way to repeating blocks of HTML.

For instance, you might want to create an itemized list 25 element long. You
could repeated the code and inserted 25 objects::

    $data = ['foo', 'bar', 'baz', 'etc', 'etc'];

    for ($i=0; $i<25; $i++) {
        $this->add('View')
            ->setElement('li')
            ->set($data[$i]);
    }

The alternative would be to use a Lister class::

    $this->add('Lister')
        ->setSource($data);

You will notice that this code is not using the ``<li>`` element. You need
to create a proper template for Lister::

    $this=$this->add(
        'Lister',
        null,
        null,
        $this->add('GiTemplate')
            ->loadTemplateFromString('<li>{$name}</li>')
    )->setSource($data);

Although more realistically you would want to store your template in a file
``list/myitems``::

    <li>{$name}{/li}

Then using the template in our code::

    $this=$this->add('Lister', null, null, ['list/myitems'])
        ->setSource($data);

When we actually think about the real-life lister application, then more
often you'll be using it within the other custom template. See :doc:`/excercises/pros-and-cons`.


Iteration Logic
---------------

.. php:method:: setModel

    Lister support iterating through the :ref:`models data range`.

.. php:method:: setSource

    Similar to setModel, however you specify array of data here. setSource is
    actually implemented around :php:class:`Controller_Data_Array`. actually
    you can pass anything iterateable to setSource() as long as elements of
    iterating produce either a string or array.

Examples::
     $l=$this->add('Lister');
     $l->setSource( array('a','b','c') );        // associative array

    or   // array of hashes
     $l->setSource( array(
         array('id'=>1,'name'=>'John','surname'=>'Smith'),
         array('id'=>2,'name'=>'Joe','surname'=>'Blogs')
         ));

    or   // dsql
     $l->setSource( $this->api->db->dsql()
         ->table('user')
         ->where('age>',3)
         ->field('*')
     );

    or   // sql table
     $l->setSource( 'user', array('name','surname'));

.. php:method:: getIterator

.. php:attr:: current_row

    Hash containing current row

.. php:attr:: current_id

    ID of current record returned by iterator.

The render() method of Lister will read next iteration of source / model inside
:php:attr:`Lister::current_row` and also set :php:attr:`Lister::current_id`.

.. php:method:: formatRow

    Called after iterating and may be redefined to change contents of
    :php:attr:`Lister::current_row`. Redefine this method to change rendering
    logic

Example::

    function formatRow() {
        parent::formatRow();

        $this->current_row['name'].='-san';
    }

You can additionally use 'formatRow' hook on the Lister to register your
callback method::

    // Add honorifics to names
    $lister -> addHook('formatRow', function($l) {
        $l->current_row['name'].='-san';
    });

.. php:method:: render

    Renders everything

.. php:method:: rowRender

    Renders single row
    
    If you use for formatting then interact with template->set() directly
    prior to calling parent

.. php:method:: formatRow

     Called after iterating and may be redefined to change contents of
     :php:attr:`Lister::current_row`. Redefine this method to change rendering
     logic

The resulting values in this hash after formatting will be populated into the
template. The template is :php:meth:`GiTemplate::render`-ed and the resulting
string is :php:meth:`AbstractView::output`-ed.

.. tip:: IMPORTANT: if your iterator will return certain field for ROW1, but
    will not have that field set for ROW2, the template of a lister will retain
    the previous value. As a result some values will get "stuck" in the template.

Extensions of Lister
--------------------

Lister is very simple class for iterating. There are also :php:class:`CompleteLister`
and :php:class:`Grid` which further builds on the foundation of Lister:

- CompleteLister repeats only some part of it's template not all the template like Lister.
- Grid recognizes structured data and will prepare row template based on columns.

Listers are also serve as a foundation for objecs such as :php:class:`Menu` and
:php:class:`View_Breadcrumb`


Pagination
^^^^^^^^^^

Pagination in Agile Toolkit is implemented through a helper view. Paginator can
be used on Lister, CompleteLister, Grid or any descendants. Lister can be placed
inside of the CompleteLister or Grid (in a dedicated tab). Due to the way how
Lister repeats all of it's template, you can't place Paginator in it, so you
would need to make it adjacent to the list.

.. php:class:: Paginator

.. php:attr:: ipp
.. php:attr:: skip
.. php:attr:: range

Usage example with Lister::

    $l = $this->add('Lister', null, 'People', 'People');
    $l -> setModel('People');


    $pg = $this->add('Paginator', null, 'PeoplePaginator');
    $pg -> setSource($l->model);

Now interacting with the paginators model will automatically affect the
limit of Lister's model. Here is how to use it with CompleteLister::


    $l = $this->add('CompleteLister', null, 'People', 'People');
    $l -> setModel('People');
    $l -> add('Paginator');     // by default uses spot {$Paginator} and not Content

Finally to add paginator to a Grid you can use a helper method - :php:meth:`Grid::addPaginator`.


Adding Filter Form
^^^^^^^^^^^^^^^^^^

Sometimes you would want to display a Lister, CompleteLister or Grid with a Filter.
A filter may appear either on the side or in the pop-over. For now I will assume
that it's physically located inside the same Page view.

.. note:: While it's technically possible to use Filter Form with Lister, you
    will probably experience problem with :ref:`js->reload() <reloading>` Lister
    because it typically does not have a containing <div>.

While you could use a regular :php:class:`Form` to implement your Form, I recommend
that you use Filter:

.. php:class:: Filter

.. php:method:: useWith

.. php:method:: addButtons

Usage example::

    $l = $this->add('CompleteLister');
    $l -> setModel('Person');

    $q = $this->add('Filter');
    $q -> useWith($l);
    $q -> addField('name');

As you add fields into filter, they will automatically be used as filters inside
model of the lister. Our example above will allow to filter by ``name`` field.

QuickSearch
^^^^^^^^^^^

.. php:class:: QuickSearch

.. php:method:: useFields

While QuickSearch preserves the functionality of a regular field it will come
with one field which will be matched against multiple field in your model.


    $l = $this->add('CompleteLister');
    $l -> setModel('Person');

    $q = $this->add('Filter');
    $q -> useWith($l);
    $q -> addField('age');
    $q -> useFields([ 'name', 'surname' ]);


Using with Iterators
--------------------

You can use lister with any object which supports iteration::

    $page->add('Lister')
        ->setSource(new DirectoryIterator('.'));

If objects supports a non-hash while iterating, be sure to convert current_row
in formatRow.

Hook: formatRow
---------------

If you do not want to re-define a Lister class just to format the data, you
can override formatRow hook instead. This can be cleverly used by controller::


    class Controller_FileLister extends AbstractController {

        function setFolder($folder) {
            $this->owner->setSource( new DirectoryIterator($folder));

            $this->owner->addHook('formatRow', $this);
        }

        function formatRow($l) {
            $file = $l->current_row;

            $l->current_row = [
                'name'=>$file->getFilename(),
                'size'=>$file->getSize(),
                'type'=>$file->getType(),
            ]
        }
    }

And to use the controller above, use this::

    $this->add('Lister', null, 'Files', 'Files')
        ->setController('FileLister')
        ->setfolder('.');

.. todo:: verify this example


