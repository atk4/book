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

.. php:attr:: current_row

    Hash containing current row

.. php:attr:: current_id

    ID of current record returned by iterator.

The render() method of Lister will read next iteration of source / model inside
:php:attr:`Lister::current_row` and also set :php:attr:`Lister::current_id`.

.. php:meth:: formatRow

    Called after iterating and may be redefined to change contents of
    :php:attr:`Lister::current_row`.

The resulting values in this hash after formatting will be populated into the
template. The template is :php:meth:`GiTemplate::render`-ed and the resulting
string is :php:meth:`AbstractView::output`-ed.

.. tip:: IMPORTANT: if your iterator will return certain field for ROW1, but
will not have that field set for ROW2, the template of a lister will retain
the previous value.

Extensions of Lister
--------------------

Lister is very simple class for iterating. There are also :php:class:`CompleteLister`
and :php:class:`Grid` which further builds on the foundation of Lister:

 - CompleteLister repeats only some part of it's template not all the template like Lister.
 - Grid recognizes structured data and will prepare row template based on columns.

Listers are also serve as a foundation for objecs such as :php:class:`Menu` and
:php:class:`View_Breadcrumb`



