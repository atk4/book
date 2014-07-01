***********
Form Fields
***********

When using :php:meth:`Form::addField` it will initialize an approirate field class
and add associate it with the form (with :php:attr:`Form_Field::form` property).

.. php:class:: Form_Field_Line

    Regular line of input

.. php:class:: Form_Field_Text

    Textarea input



.. php:class:: Form_Field

    Implements generic Form Field.

Form Field can operate in two modes. If you simply add the field into
any object which does NOT have form as an owner, then field appears as
a stand-alone view::

    $menu->add('Form_Field_Line', 'search');

In most cases, however, you do add a field into a form::

    $form->addField('Line', 'search');

.. note:: you can also add a search field inside any view within a Form,
    see :doc:`form/layout` for more info.


.. php:method:: setForm

    Automatically called when Field appears to be inside Form. This
    will enable validation hook and will allow field to read data on POST.

When field is added into Form directly, it also surrounds itself with a
field_row which often contains a label (See :doc:`form/layout`).

.. php:method:: setCaption($label)

    Changes label of the field. $label is localized.

.. php:method:: displayFieldError($message)

    Will execute field error assuming that js_widget is used. Use
    :php:meth:`Form::error` instead.

.. php:method:: set($value)

    Set default value for the field.

.. php:method:: setNoSave

    Will make sure that field value is not saved when you call save()

Adjacent elements to the field
------------------------------
Here are several methods, which can be used to add other elements relatively
to the field.

.. php:method:: addButton($label, $position)

    Will add button eihter position='before' or position='after' the field.
    If $position is a hash, then 'position' key will be used for positioning
    and the rest will be passed down into ``add('Button', <here>)``.

Additionally the button will automatically be linked with the field in the
following way - every time the field value changes, it's set as a "val"
data attribute of the button. This allows you to easily capture value
of a linked field when button is clicked (see :php:meth:`Button::onClick`)::

    $field = $form->addField('name');
    $field->addButton('Greet')->onClick(function($b, $data){
        return "Hello, ".$data['val'];
    } );


.. php:method:: beforeField()

.. php:method:: afterField()
.. php:method:: aboveField()
.. php:method:: belowfield()

Those methods return a new view, which is located just to the proper side
of the field. You can chain this method with ``add()`` and drop another view
in there::

    $form->getElement('name')->beforeField()->add('Text')->set('Mr:');

.. php:method:: setFieldHint

    Displays a hint under a field (usually with gray text).

.. php:method:: addIcon

    Adds icon at the right side of the field. Second argument can be a link,
    or JS chain which will be executed when icon is clicked.

Field properties
----------------

.. php:method:: get()

    Returns value of the field. Use ``$val = $form['field']`` unless
    you call this method from within a Field class.

.. php:method:: setAttr($attr, $value)

    Set attribute on the ``<input>`` such as ``setAttr('maxlength',50)``



Buiding HTML
------------


Field takes a slightly different path in building HTML it outputs. instead
of using a template, it converts all the attributes into a tag with getTag() method.

.. php:method:: getTag

This method takes many differet argument variations and produces an HTML tag
with properties.

.. php:method:: getInput

    Will return HTML only for the <input> element.

.. php:method:: render

    When inside form, will return whole form line, along with label (if form template supports)
