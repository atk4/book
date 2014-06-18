****
Form
****

Form object is the most powerful and flexible form implementation class you
can find in PHP frameworks. It effectively reuses the :doc:`core features </core-features>`
of Agile Toolkit as well as inheriting all the benefits of :doc:`/views` core.

Form stores significant amount of features inside :php:class:`Form_Field` and
descending classes.

Although Form is physically implemented in `Form_Basic` class, you should always
use `Form` class directly. This is important for :ref:`Class Substitution` techinque.


Form in Agile Toolkit will handle all aspects of the web form starting
from form building, display, layout, submission, validation, database or
model integration.

By default form will use AJAX for submission, and will require page reload.


Forms in Agile Toolkit are fully self-sufficient. Once you initialize
the form, it will stand on the page on it's own, it will handle
submission properly, it will automatically validate itself, encode
received output and integrate with JavaScript. Other frameworks tend to
focus primarily on form display, but Agile Toolkit handles submission
and dynamic actions. Forms in Agile Toolkit work quickly, efficiently
and without conflicts with other forms. Form can appear anywhere -
on a page, in a dialog, in popup or in the menu.

Form fields are typically implementing basic fields supported by HTML
and extensions provided by jQuery UI. It's easy to add additional
field types or enhance existing.

Adding Form
~~~~~~~~~~~


To add a form::

    $form = $this->add('Form');
    $form->addField('name');
    $form->addSubmit('Say Hello');

    $form->onSubmit(function($f) {
        return "Hello, ".$f['name'];
    });

Once you add the form as shown above, it will be automatically rendered
by Agile Toolkit, it will have it's submit handler properly configured,
it will perform necessary input data loading, escaping and validation.

Agile Toolkit uses POST to submit all forms through an AJAX request.


.. toctree::
    :maxdepth: 1

    form/core
    form/fields
    form/validation
    form/layouts


