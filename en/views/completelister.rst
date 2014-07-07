CompleteLister
==============


.. php:class:: CompleteLister

    While similar to :php:class:`Lister`, this class will use region {row} in
    its template and after iterating it will replace it back into {rows} before
    rendering the rest of its template.

Additionally CompleteLister supports separators, totals row, alternating
tag for odd/even rows and it's default template will use ``<ul><li>.. `` tags
for presenting ``$name`` field of the model.

.. php:attr:: sep_html

The actual name of the tags can be changed through class properties.

.. php:attr:: item_tag
.. php:attr:: container_tag


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

.. php:attr:: row_to


Total Calculation
-----------------

.. php:attr:: totals
.. php:attr:: total_row
.. php:attr:: totals_t

CompleteLister supports calculation of totals. To enable calculation:

.. php:method:: addTotals

While drawing the items, the totals will be automatically calculated. This
calculation method will only be able to calculate totals for the rendered
fields. If Lister is used in conjuction with Paginator, only totals from
the current, visible page will be calculated.

To address this problem you can use addGrandTotals

.. php:method:: addGrandTotals

Total calculation plays a major part inside :php:class:`Grid`.

Row rendering
-------------

.. php:method:: render

CompleteLister uses a more advanced render loop compared to :php:meth:`Lister::render`::

    $iter = $this->getIterator();
    foreach ($iter as $this->current_id=>$this->current_row) {

        if($this->sep_html && $this->total_rows) {
            $this->renderSeparator();
        }

        // calculate rows so far
        $this->total_rows++;

        // if onRender totals enabled, then update totals
        if ($this->totals_type == 'onRender') {
            $this->updateTotals();
        }

        // render data row
        $this->renderDataRow();

    }

The enhancements include total calculation:

.. php:method:: updateTotals

.. php:method:: renderTotalsRow

After the loop, render continues to output the totals or grand totals::

    // calculate grand totals if needed
    if ($this->totals_type == 'onRequest') {
        $this->updateGrandTotals();
    }

    // set total row count
    $this->totals['row_count'] = $this->total_rows;

    // render totals row
    $this->renderTotalsRow();



Here are the few key ingredients to this rendering process:

.. php:method:: getIterator
.. php:attr:: current_id
.. php:attr:: current_row
.. php:method:: formatRow
.. php:method:: rowRender

