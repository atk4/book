*********
Form Core
*********

.. php:class:: Form

    Implement HTML form

.. php:method:: addField($class, $options, $caption)

    add form field

A most fundamendal form would have a few fields defined, submit button
and handler.

When adding a field, you must remember that :php:class:`Form_Field` is
very different from Model :php:class:`Field`.

addField can be called in various ways::

    $form->addField('title');
    $form->addField('Line', 'title');
    $form->addfield('Line', ['name'=>'title']);
    $form->addfield('Line', ['name'=>'title'], 'Title');

Although any of the lines above will produce exactly same result.

The form template has an area for buttons. When you use addSubmit()
it adds the button in this area. addSubmit will also highlight the
button so it would appear visually distinctive from other buttons.
addSubmit also assigns icon to the button. You can override this through
:ref:`Component Definition Array`.


When adding a field, method will return instance of :php:class:`Form_Field`.
Chaining additional calls can help you change the way how the field is
working::

    $form->addfield('title')->validateNotNull();

As mentioned before, form will submit itself through AJAX. You can switch
this off with js_widget property:

.. php:attr:: js_widget

    Name of jQuery UI Widget to enhance form into AJAX submission. Setting
    this property to ``null`` or ``false`` will disable the property.

When disabled, form will automatically read and populate POST data into
it's fields.

The other way how you can get some data into a form is using set() method.

.. php:method:: set($field, $value)

    This will call :php:meth:`Form_Field::set` ($value) to set default
    value of the field. Call this before handling submission.

Adding Buttons
==============

Form supports two types of buttons: submit and regular buttons.

Submit buttons will take all the form data and send it back with POST.
Regular Buttons are just that - regular :php:class:`Button`

.. php:method:: addSubmit($label)

    Create a submission button.

.. php:method:: addButton($label)

    Create a regular button.

Submission handling
===================

Adding buttons is not enough. You also need to capture submission event.
Form routes submission request to exactly same page with exactly same GET
data (See :ref:`Sticky GET`), however you can tell the request apart
by with the following two methods:

.. php:method:: isSubmitted()

    returns ``true`` if form is being submitted.

.. php:method:: onSubmit($callback)

    executes callback if form is being submitted passing form itself
    as first argument.


If you have created multiple submission buttons you can use isClicked method
to verify which one was clicked.

.. php:method:: isClicked($submit_button)

    When you have multiple submission button, you can call isClicked
    on any button to verify if it has been clicked.

    .. note:: If user has pressed enter in the form isClicked will always
        return false.

Next example implements dual submit of a form. If the ``Save and Add New`` is
clicked, then after saving, user stays on the same page possibly adding more
records. Just clicking Save button will return user to the parent page
(See :php:class:`URL` and :doc:`/js/univ` for more info on redirects and routes)::

    $form->addSubmit('Save');
    $save_and_add = $form->addSubmit('Save and Add New');

    $form->onSubmit(function($form) use ($save_and_add) {

        $form->save();

        if (form->isClicked($save_and_add)) {
            return $form->js()->univ()->redirect($form->app->url());
        }

        return $form->js()->univ->redirect($this->app->url('..'));
    });

Responding on submission with errors
------------------------------------

If instead of successful submission you want to display an error, form allows
you to do that

.. php:method:: error($field, $message)

    Displays error message under $field.

Example::

    $form->onSubmit(function($form) {
        if($form['age']<10) return $form->error('age','too young');

        $form->save();
        return 'Success';
    });

Form also respects in-model validation and will display error message
next to an appropriate field. See :php:class:`Exception_ValidityCheck`
and :php:class:`Validator`.

Associating with Model
======================

Most commonly you will use Form to edit :php:class:`Model` data. For that
you should use:

.. php:method:: setModel($model, $actual_fields)

    Associate form with model and populate actual fields. :php:meth:`Form::update`
    will save this model.

You can associate form with more models by calling

.. php:method:: importFields($model, $fields)

    Add fields from model into form. Also :php:meth:`Form::update` will save
    this  model.

Example using Form with one model::

    $form = $this->add('Form');
    $form->setModel('Book', ['title','descr']);

If you want form to edit multiple models, you can import fields. Note that
the model for Import Fields must be alerady initialized::

    $author = $form->model->ref('author_id');
    $form->importFields($author, ['name', 'surname']);

.. php:method:: save()

    Copy submitted field data into associated models and call
    :php:meth:`Model::save`.

Form can automatically save fields into model record::

    $form->onSubmit(function($f){
        $f->save();
        return 'Saved Successfully';
    });

If your model was :php:meth:`Model::loaded` with particular record, then
this record will save back modified fields. Otherwise a new record is
created.

Advanced use with models
------------------------

You can associate Form with a model, but do not import any fields initially::

    $form->setModel('Book', false);

This actually initializes a special controller: :php:class:`Controller_MVCForm`,
which is responsible for binding Form with Model and also transitioning Model
field types into Form Field types.

You can access this controler to call :php:meth:`Controller_MVCForm::importField`::

    $form->controller->importfield('title');

This will create field in form with the identical name (if possible). You can then
access :php:class:`Form_Field` as a regular child of a form::

    $form->getElement('title');

Please note that if you want to access Form :php:class:`Field`, you can use::

    $form->model->getElement('title');


