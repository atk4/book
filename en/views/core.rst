*********************
Core UI View Features
*********************


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

Another significant difference is that :php:meth:`View::setModel` will
automatically fill out template tags with model values if they are loaded, but
AbstractView will not do such a thing.

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


.. php:method:: defaultSpot

    Default spot. 'Content'

.. _template definition:

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

.. seealso:: To better understand template behaviours, see excercise: :doc:`/excercises/view-envelope`


.. _recursive rendering:

Recursive Rendering
-------------------

When all objects are initialized in Agile Toolkit it continues with the recursive
rendering phase. It starts with the application and calls recursiveRender method.

.. php:method:: recursiveRender

    Will render all children views by placing their output inside respective
    spots of this object's template. Then it will call render of this object
    which would ``output()`` data into the owners template.

.. php:method:: render

    Method responsible for converting all the dynamic data related to the
    current view (such as model) into a HTML representation and passing it
    to output()

.. php:method:: output

    A supplied argument (HTML string) will be appended to a spot within
    owners template.

Objects contained within the render tree will recursively render and output
themselves producing a fully functional HTML page. Next figure illustrates how
the objects are structured in the render tree. While some of those objects are
non-visual (Models, Contollers), Page relies on Menu, Crud, Form and it's own
template. Form relies on Field and Button objects as well as it's own template
and so forth.

.. figure:: /figures/compose-principle.png

AbstractView js() method
------------------------

.. php:method:: js

    Creates JS chain for a view using event binding.

.. php:method:: on

    Creates JS chain for a view using jQuery on() binding.

.. php:method:: getJSID

    Returns a safe identifier to be used as HTML element ID property
    based on objects name.


Please see :doc:`/js` for documentation on js() method.


.. _cutting:

Cutting Output
==============

Each request to Agile Toolkit goes through a full initialization cycle. It
initializes the routing, finds the page and initializes all objects on that
page.

On large systems the number of pages may be large, yet contents of each
indidividual page will still be manageable, so this technique scales well
in practice.

In some situations however only a part of the page needs to be rendered. In
desktop frameworks this behaviour is called "repaint". In Agile Toolkit
we refer to it as :ref:`cutting`.

To experience cutting in the basic form, try adding ?cut_page=1 to
your request.

Agile Toolkit will respond with HTML of the page only and will not render
menu, header or footer.

Cutting Page
------------

When you have page A and page B open side-by-side in the browser, normally
they would share quite a lot:

- Header and Footer markup
- Navigation menu
- Other parts of :ref:`Layout <Layout>` (except Content)
- JavaScript libraries
- CSS dependencies

You can save quite a lot of processing if user could jump from A to B
without reloading common components.

In Agile Toolkit this is very easy to achieve. Add the following two pages::

    // page/one:

    $this->add('Button')->set('Go to page 2')->js('click')
        ->univ()->page($this->app->url('two'));

    // page/two:

    $this->add('Text')->set('You are now on page 2');

Now if you open first page in your browser and click the button, only HTML
for second page will be loaded and substituted approprietly.

Agile Toolkit will also propely load JavaScript dependencies for second page
and initialize all the events.

:ref:`univ_page` will properly use cut_page argument to render page object only.

Cutting Objects
---------------

With ``cut_object=`` argument it's possible to specify a name of a particular
object and only that object will be rendered. This technique is automatically
used when objects reload themselves. If you look inside borwsers ispector::

    $this->add('Button')->set(rand(1,99))->js('click')->reload();

This button would reload itself by cutting it's own HTML only. You can
observe another behaviour - only :ref:`JavaScript Chains <javascript chain>`
of that particular object and it's children will be included in resulting
output.

This is done specifically to avoid binding actions multiple times to elements
which are not being replaced with reload().

Cutting Regions
---------------

This technique is currently obsolete, but used to output only specified region
of view template.
