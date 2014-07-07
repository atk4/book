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


CompleteLister
==============


.. php:class:: CompleteLister

    While similar to :php:class:`Lister`, this class will use region {row} in
    its template and after iterating it will replace it back into {rows} before
    rendering the rest of its template.

Additionally CompleteLister supports separators, totals row, alternating
tag for odd/even rows and it's default template will use ``<ul><li>.. `` tags
for presenting ``$name`` field of the model.



Template Preservation Technique
-------------------------------

Sometimes the Designer creates an HTML template and sends it off to the
Developer. The job of Developer now is to make template display actual data.

In other words - a chunk of sample HTML must be enhanced with tags and stored
inside ``template`` folder.

The Template Preservation Technique is ability to leave HTML intact after
the tags are added. For example, developer might send us the following
markup for the article::

    <h2>Header title goes here</h2>

    <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris feugiat
    aliquam malesuada. Sed eget massa metus. Proin adipiscing mi quis enim</p>

    <p>ullamcorper sagittis. Nullam vitae neque a nunc volutpat ullamcorper.
    Integer sed leo sagittis, congue diam nec, semper justo. Etiam id augue</p>

After adding tag, the template would look like this::

    <h2>{title}Header title goes here{/}</h2>

    {descr_html}
    <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris feugiat
    aliquam malesuada. Sed eget massa metus. Proin adipiscing mi quis enim</p>

    <p>ullamcorper sagittis. Nullam vitae neque a nunc volutpat ullamcorper.
    Integer sed leo sagittis, congue diam nec, semper justo. Etiam id augue</p>
    {/descr_html}

The template looks similar to original making it much simpler for the Designer
to change templates directly without developer's intervention.


When it comes to listing things, we would receive this::

    <h4>Interests</h4>
    <ul>
        <li>skiing</li>
        <li>skating</li>
        <li>running</li>
    </ul>

CompleteLister allows you to keep the list intact as you convert it into
template::

    <h4>{title}Interests{/}</h4>
    <ul>
        {rows}{row}
        <li>{name}skiing{/}</li>
        {/row}
        <li>skating</li>
        <li>running</li>
        {/rows}
    </ul>

When this template is used with CompleteLister, it would:

#. Clone tempalte for ``row``
#. Delete region ``rows``
#. Render cloned region for each iteration and append to ``rows``

This also allows you to destroy lister much safer if no elements have
been rendered.

.. todo:: write article about this.
