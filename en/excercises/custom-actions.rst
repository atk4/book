***********************************
Grid Custom Action Column Excercise
***********************************

In this excercise I will guide you through customizing Grid's columns and
also creating some AJAX interactivity. We will be looking to achieve the
following look:

.. image:: /figures/excercise-action/final-look.png

Setting up the project
======================

You will need a working Agile Toolkit, so go to the website, click Download
and install it as a new project on your local computer. After you are
done go inside "Admin". You will see the default page which will prompt
you to opening admin/page/index.php in your editor.

When you do, you will see something like this::

    class page_index extends Page {
        public $title='Dashboard';
        function init() {
            parent::init();
            $this->add('View_Box')
                ->setHTML('Welcome to your new Web App Project. Get started by opening '.
                    '<b>admin/page/index.php</b> file in your text editor and '.
                    '<a href="http://book.agiletoolkit.org/" target="_blank">Reading '.
                    'the documentation</a>.');


        }
    }

In this tutorial I will mostly work on my ``index.php`` file. If you wish, you can
create new files for each class (and that is what you should do in big projects),
but I will just cram everything into my page file.

First, I need to create a model to work with. I don't want to use database for
this example, so I'll create a model with some static data only::

    class Model_Test extends Model {
        function init(){
            parent::init();

            $this->addField('name');

            $this->setSource('Array',['John','Peter','Joe','Steve']);
        }
    }

This model automatically populates itself with 4 records from a supplied array.
It will contain ``id`` and ``name`` fields (:php:class:`Model_Field`) only, which should be enough.

Next, I'm going to add a default :php:class:`Grid` and link it with my new model.
Add the following code inside your page::init::


    $grid = $this->add('Grid');
    $grid->setModel('Test');

You should now be able to see table with two columns. We are done for the basic setup. The rest of
my tutorial I'll divide into several steps.

Step 1: Adding a custom column to Grid
======================================

(See documentation at :ref:`grid_custom_formatters`).

First I'll need to create my own Grid class by extending Grid::

    class MyGrid extends Grid {

        function init_actions($field){

        }
        function format_actions($field){
            $this->current_row_html[$field] = "hello <b>world</b>";
        }
    }

To make use of this new column type, I'll have to manually add it by using
:php:meth:`Grid::addColumn`::

    $grid->addColumn('actions','actions');

First argument is type, and second will be used as field name and label.

You can now refresh the page and you will see new column populated with "hello world".
Using :php:attr:`Grid::current_row_html` allows me to specify HTML tags. Some
of you might argue, that it's not nice to have markup inside PHP file, so I'm
going to move it to a designated template next.

Step 2: Creating and using custom template with markup
======================================================

Create file ``admin/template/column/actions.html``::

    <div class="atk-inline">
        <div class="atk-buttonset">
            <a href="javascript:void(0)" class="atk-button-small do-set-default">
                <span class="icon-flag"></span>
            </a>
            <a href="javascript:void(0)" class="atk-button-small do-delete atk-swatch-red">
                <span class="icon-trash"></span>
            </a>
        </div>
    </div>

.. note:: I will then compress all of the HTML into a single line, because
    spaces inside HTML markup between elements can cause undesired glitches
    inside your interface. I recommend you to look into JADE.

The above markup uses Agile CSS to create a button-set with two buttons. Each
button will have only icon and the second button will be red. I have taken
the markup example directly from http://css.agiletoolkit.org.

I also added two classes ``do-set-default`` and ``do-delete``, which will help
me binding triggers to those buttons.

Next, I need to load the template and the best place would be inside init_action()::

    function init_actions($field){
        $this->columns[$field]['tpl']=$this->add('GiTemplate')->loadTemplate('column/actions');
    }
    function format_actions($field){
        $this->current_row_html[$field] = $this->columns[$field]['tpl']->render();
    }

I am making use of :php:attr:`Grid::columns`, which would store individual template
for each column of type ``action``. The template is loaded and parsed inside
``init_action()`` however it is rendered for each row. This is so that you could
use :php:meth:`GiTemplate::set` to change some tag values on row-to-row basis (although
i'm not using any yet).

You should now see a nice two buttons for each field on the Grid.

Step 3: Binding actions to buttons
==================================

The important thing here is to keep efficiency in mind. We must not create
object during each row iteration to maximize scalability of our application.
I will be taking advantage of :php:meth:`AbstractView::on` to bind action
to a button. This needs to be added inside ``init_action`` method::

    function init_actions($field){
        $this->columns[$field]['tpl']=$this->add('GiTemplate')->loadTemplate('column/actions');

        $m=$this->model;

        $do_flag = $this->add('VirtualPage')->set(function($p)use($m){
            $name=$m->load($_GET['id'])['name'];

            // $m->flag();

            return $p->js()->univ()->alert('You have flagged '.$name)->execute();
        });

        $this->on('click','.do-set-default')->univ()->ajaxec([$do_flag->getURL(), 'id'=>$this->js()->_selectorThis()->closest('tr')->data('id')]);
    }


As you see, I am creating :php:class:`VirtualPage` with a call-back method. The
call-back I'm defining will make use of the Grid's :php:attr:`AbstractObject::model` and will potentially
call a low-level custom-added method "flag", which I would have to implement
later. I must stress that it's very important that the same model instance
is used for both Grid and action here, to make sure that regardless of passed
``id`` GET attribute, user will only be able to load records which would
physically appear in the Grid.

The call is done through :ref:`ajaxec <ajaxec>` and therefore requires me to finish with
:php:meth:`jQuery_Chain::execute` to function properly.


The array I pass to ajaxec is actually a :ref:`url component array` and I'm
relying on the closest ``<tr>`` element which has ``data-id="1"`` attribute
set to the ID of each record.

The final thing for me to add to ``format_actions`` is a custom width of the column::

    $this->setTDParam($field, 'width', '100'); // in pixels

Although alternatively I can add class ``atk-expand`` to one of the other fields with
the similar effect, but because I may be using this column in various places with
different columns, I need to be sure it looks ok.


Things to try
=============

You can continue yourself and try some other things:

- find this project on my github and fork it as you work on it
- implement "delete" action
- adjust CSS to make field visible only on mouse-over
- add nicely looking jQuery UI tooltips on the buttons
- convert this column into Grid column class
