******************
Core View Features
******************

Core features of all View objects are implemented in classes :php:class:`AbstractView`
and :php:class:`View`.


Difference between View and AbstractView
========================================

- AbstractView could be associated with any template.
- View assumes, there would be one HTML element in your template, to which I'll
  refer as "main element".
- View has methods to assign class, style or attribute to main element.
- View assumes there wolud be a {$Content} tag inside it's template.
- You can use "View" without inheriting it, and it's advised to inherit AbstractView.

When you are creating your own view, think if if you would need an functionality
of modifying class, style of your view in your PHP code. If that's not necessary,
use AbstractView.

If unsure - use View.


In this chapter we will look into AbstractView class first, and then into the
View.

AbstractView core features
==========================

.. php:class:: AbstractView

    A base class for all Visual objects in Agile Toolkit. The
    important distinctive property of all Views is abiltiy
    to render themselves (produce HTML) automatically and
    recursively.

.. php:attr:: template

    Object containing indexed HTML template. Property is initialized by
    initTemplate method.

.. php:attr:: spot

    Tag within parents template where render will send output.

.. php:method:: initTemplate

    Method called during initialization of the View to load the
    defaultTemplate (or cutom one), parse it and verify existance of
    spot.

.. php:method:: add

    Add for Views can use 4 arguments. Third argument specifies a default
    value for $spot property. Fifth is a template definition.

.. php:method:: defaultTemplate

    Returns a default template definition, same as 4th argument do
    :php:meth:`AbstractView::add`.


Template Definition
-------------------

This parameter can be defined in several formats:

- String - would be used as a name of the region within owners template. The
  template would be cloned.
- Array with single string element. This element will be used as a name of the
  template. A file will be located (:php:class:`PathFinder::locate`) and parsed.
- Array with two elements. The first element still contains name of the template
  file. Second element would contain a region which would cloned right after loading
  the template.
- GiTemplate object. You can always pass a :php:class:`GiTemplate` object which
  then would be used as view's template.

Rendering Behaviour
-------------------

When you add object inside a child and specify a custom region, then the contents
of this region will be deleted after object is added. The contents will be then
repopulated during the rendering.

If you define child view tempalte as a string, it will be cloned right before it
is emptied.

.. seealso:: See excercise: :doc:`/excercises/view-envelope`


