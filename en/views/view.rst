
**********
View Class
**********

.. php:class:: View

Core features
=============


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

.. php:method:: addComponents

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

If you wish that icon is placed to the right from the text, use ``icon-r``.

Setting the text to ``false``, will only display icon.

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


Using with Model
================

.. php:method:: setModel()

    This method will not only associate view with the model, but will
    auto-fill values of the model inside template tags just before
    rendering itself.


