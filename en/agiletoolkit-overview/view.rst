.. _view:

User Interface (Views)
######################

A View is an object in Agile Toolkit which knows how to render itself. Some
examples of Views are "button", "form", "lorem ipsum", "menu" etc.

By default, a View relies on a template engine to produce its output. Views
can contain other views. All Views in Agile Toolkit contain many useful
features.


HelloWorld View
===============
You don't code 'Hello World' in Agile Toolkit. You use existing class.
Placing this code inside your :php:meth:`Page::init` will display
the usual greeting on your page::

    $this->add('HelloWorld');

.. tip::

    This is a good example of Agile Toolkit principle. Create object
    once and then re-use it.

Find ``HelloWorld.php`` file in atk4 folder and open it, you should see this::

    class HelloWorld extends AbstractView
    {
        // message text
        private $message = 'Hello, World';

        /**
         * Set custom message text
         *
         * @param string $msg Message text
         *
         * @return void
         */
        function setMessage($msg)
        {
            $this->message = $msg;
        }

        /**
         * Render message
         *
         * @return void
         */
        function render()
        {
            $this->output('<p>'.$this->message.'</p>');
        }
    }

.. warning ::
    Class "HelloWorld" is probably the only class in Agile Toolkit which
    does not rely on "Template" engine. Although I have included it here
    to help you understand how rendering works, always use template files
    and never store markup in your PHP files.

Reading the class source reveals that you can set a property "message",
which will display a different greeting. Default properties can be
reassigned through the add() call::

    $this->add('HelloWorld', ['message' => 'Hello, Agile Toolkit']);

Another way to change the message is after the object has been initialized::

    $this->add('HelloWorld')->setMessage('Hello once again');

View class
==========

A most popular HTML element is ``<div>``. In Agile Toolkit most objects
will render into a single DIV or multiple HTML elements contained within
that DIV (See ::php:class:`View` class)

This DIV will always have an unique ID. The next example will create a DIV
object but instead of rendering it normally, it will display the HTML
code it generates inside a box::

    $h = $this->add('View')->set('Sample Header');

    $box = $this->add('View_Box');

    $box->set( $h->getHTML() );

.. tip ::
    Agile Toolkit automatically encodes text into HTML, JS, SQL etc
    as necessary. This help you avoid some validations and prevent some
    nasty exploits (such as JavaScript injection)


View with Template
==================

Create file ``shared/templates/view/mytemplate.html``::

    <div id="{$_name}">
    Roses are {rose_color}Red{/}, Violets are {violet_color}Blue{/}.
    I love {$Target}.
    </div>

Then place this UI code::

    $poem = $this->add('View', null, null, ['view/mytemplate']);

Notice how we are re-using the existing View class but specifying
alternative template to it. This can be done with any view. For instance
a :php:class:`Form` class can use template ``form/compact`` to render
your form in a different look.

A specified template is automatically parsed and a special :php:class:`GiTemplate`
object will be available through a ``template`` property of a View::

    $poem->template->set('rose_color', 'Pink');
    $poem->template->set('violet_color', 'Violet')

Finally - when you add one view into another, you may select ``Spot`` where
it will appear::

    $poem
        ->add('Button', null, 'Target')
        ->set('Red Buttotn')
        ->addClass('atk-swatch-red');

When ``Spot`` is ommitted, then it defaults to '{$Content}', which should
be defined in a template.

Composite Views
===============

In Agile Toolkit a view consisting of other views is called a Composite View.
Some examples of composite views are :php:class:`CRUD` (which consists of
:php:class:`Grid` and :php:class:`Form`) and also :php:class:`Form` (consisting
of :php:class:`Form_Field` and :php:class:`Button`)

Controller and Model
====================

As you probably have noticed - the Views we have used so far work pretty
good without models or controllers. In real world application, you are most
likely to link the view with :ref:`data source` (by specifying a Model)::

    $poem->setModel('Book')->tryLoadAny();

.. note::
    You should change name of the {tags} in ``mytemplate.html`` to correspond
    with the fields of the "Book" model.

.. meta::
    :title lang=en: Introduction to User Interface (Views)
    :keywords lang=en: model view controller,model layer,formatted result,model objects,music documents,business logic,text representation,first glance,retrieving data,software design,html page,videos music,new friends,interaction,cakephp,interface,photo,presentation,mvc,photos
