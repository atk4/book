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


View class core features
========================

.. php:class:: View

View class assumes that you have a HTML element in your template which can
accept class, style and other attributes. Here is a approximate tempalte
for the standad View class::

    <{element}div{/}
        id="{$_name}"
        class="{$class}"
        style="{$style}"
        {$attributes}
    >{$Content}</{element}div{/}>

The meaning of {$_name} tag was already explained in :doc:`templates` section,
and the {$Content} tag is used throughout Agile Toolkit as a default spot for
objects, so that any objects you would add inside a View would output themself
into {$Content} spot.

The other tags here can be populated by methods of a View class.

.. php:method:: setElement

    Change the HTML element which view would output. By defaut it will output ``div``.
    See :php:class:`H1` for an example.


.. php:method:: setClass

    Set an CSS class to a view element.

.. php:method:: addClass

    Add new CSS class to the view without overwriting previously assigned classes.

.. php:method:: removeClass

    Remove one of the assigned classes

.. php:method:: setStyle

    Replace style definition of objects HTML element. Accepts two arguments for
    property and value. ``setStyle('background', 'red')``

.. php:method:: addStyle

    Add new in-line style definition to existing ones. Accepts two arguments for
    property and value.a ``addStyle('background', 'red')``

.. php:method:: removeStyle

    Remove style specified by the property: ``removeStyle('background')``

.. php:method:: setText

    Replaces content with a text. (automatically localized and escaped)

.. php:method:: setHTML

    Replaces content with a HTML string. Will not localize or escape.

.. php:method:: set

    Similar to setText, but can contain array with component definitions and icon.
    See: :ref:`_Component Definition Array`.



.. _Component Definition Array:

Component Definition Array
==========================

View and some other objects based on Views will accept a so called Component
Definition arrays. This allows you to use "label" arguments to define additional
components and elements (such as icons, badges, etc)

To learn more about AgileCSS components, icons and badges see :doc:`/css`

Setting components
------------------

.. php:method: addComponents

    Assign several components to element as defined in supplied hash.

Components in Agile CSS are defined by adding class ``atk-<type>-<value>``. Some
example are: ``atk-size-mega``, ``atk-swatch-red``, ``atk-effect-info``,
``atk-box`` and ``atk-shape-rounded``. When you need to set several of them,
you can use addComponents method::

    $this->add('View')->set('Hello, World')
        ->addComponents( [ 'box'=>true, 'size'=>'mega', 'effect'=>'info' ] );

To save you some time a component definition array format can be used when
calling set::

    $this->add('View')->set( [
        'Hello, World' ,
        'box'=>true,
        'size'=>'mega',
        'effect'=>'info'
    ] );

The string in this hash appearing without key will be assigned to ``0=>``. Other
hash keys will be reconstructed into atk- components.

Defining Icon
^^^^^^^^^^^^^

View allows you to define property 'icon' by setting it to desired name of the icon.
The view will automatically add necessary markup to prepend your text with an icon::

    $this->add('View')->set( [
        'Hello, World' ,
        'icon'=>'heart',
        'box'=>true,
        'size'=>'mega',
        'effect'=>'info'
    ] );

Because icon is implemented through an :php:class:`Icon` view, which is also
inherited from View, you can also specify nested components to the icon::


    $this->add('View')->set( [
        'Hello, World' ,
        'icon'=>[
            'heart',
            'swatch'=>'red'
        ],
        'box'=>true,
        'size'=>'mega',
        'effect'=>'info'
    ] );

Other views, such as :php:class:`Menu_Advanced_Item` will define additional
extensions such as ``icon2``, ``badge``, etc.

Extending Component Definition Array
====================================

If you wish that your object could handle more extensins to the component
definition, you can extend set() method of your view::

    function set($data){
        if(is_array($data)){
            if($data['my_icon']){
                $this->add('Icon',null,'MyIconSpot')->set($data['my_icon']);
            }
            unset($data['my_icont']);
        }
        return parent::set($data);
    }

.. tip:: Always document your extensinos to Component definitions.

