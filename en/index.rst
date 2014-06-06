.. Agile Toolkit Book documentation master file

Welcome
#######

Agile Toolkit contains everything you need to develop web apps. It is a
self-sufficient PHP framework modeled after Desktop UI Toolkits. Agile
Toolkit may have some similar concepts to other Web MVC frameworks, but
you should consider that some of your existing knowledge might be challenged.


Getting Started
===============

Agile Toolkit PHP framework can be divided into 3 major parts:

 - Data Layer - working with the database, APIs, etc.
 - UI Layer - creating fancy UI interfaces and outputing HTML.
 - Extensions - a lots and lots of ways to improve things.

To start it off with a simple example, let's build a very simple
``CRUD`` UI.


Create a Model
------------

Create a file shared/lib/Model/Book.php::

    class Model_Book extends Model {

        public $table='book';

        function init() {

            parent::init();
            $this->setSource('Session');

            $this->addField('title');
            $this->addField('year');
            $this->addField('author');

            $this->addField('is_borrowed')->type('boolean');
        }
    }

In this file we use a PHP method cals of a model to describe model structure,
it's relations and data storage.

Build UI
-----------

UI is build by adding "views" inside other "view". The our case we will
use a "Page" to add our ``CRUD`` there. Open file admin/page/index.php and add::

    class page_index extends Page {
        function init() {
            parent::init();

            $crud = $this->add('CRUD');
            $crud->setModel('Book');
        }
    }

Navigate to admin/public/ and you should see your ``CRUD`` in action.

Extending
-----------
Now that you have got the basic editing, you can pimp-it-up a little.

Open your model file and add a new method inside your model::

    function borrow() {
        $this['is_borrowed']=true;
        $this->save();
    }

Next we need to update UI to reflect. Lets do that by adding more code after
the crud is initialized::


    if ($p = $crud->addFrame('borrow')) {

        $m = $crud->model;
        $m->load($crud->id);;

        if ($m['is_borrowed']) {
            $p->add('View_Error')->set('Book '.$m['title'].' is already borrowed');
        } else {

            $p->add('P')->set('Are you sure you want to borrow '.$m['title'].'?');

            $button = $p->add('Button')->set('Yes')->addClass('atk-swatch-green');
            if ($button -> isClicked()) {

                $m->borrow();
                $p->js()->univ()->closeDialog()->execute();

            }

            $p->add('Button')->set('No')->js('click')->univ()->closeDialog();
        }
    }

If the code seems a bit overwhelming for you, do not worry. We will go
through all the concepts here gradually in this documentation. Do, however,
try it out in the local copy of Agile Toolkit.

.. TODO::

    TODO: insert video / demo


The Coding Style of Agile Toolkit
---------------------------------

When you are writing an application based on Agile Toolkit, you must follow
a coding style of Agile Toolkit. If you will try to incorporate Agile
Toolkit into your existing code structure you might face some difficulties.

For the best experience start a new application and improve it as-you-learn.




.. meta::
    :title lang=en: .. Agile Toolkit Documentation
    :keywords lang=en: doc models,documentation master,presentation layer,documentation project,quickstart,original source,sphinx,liking,cookbook,validity,conventions,validation,cakephp,accuracy,storage and retrieval,heart,blog,project hope
