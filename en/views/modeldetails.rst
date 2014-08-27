*****************
View_ModelDetails
*****************

.. php:class:: View_ModelDetails

This views displays contents of a loaded model in a table with two columns. Left
column containing lables and the right column containing values::

    $this->add('View_ModelDetails')->setModel('Book')->load(1);


This view is implemented on top of "Grid" therefore you can add more columns
to it if you wish. The View will not allow user to modify model values.

