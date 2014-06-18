
Lister Excercise - pros and cons
================================

In this example we are creating a View to compare different car models. The
goal for us is simple - create a view which hides the complexity and
appears to be very simple to use on our page. This is how we
anticipate to use our view::

    $c = $this->add('Columns');
    $left = $c->addColumn(6);
    $right = $c->addColumn(6);

    $left->add('View_Car')
        ->setModel('Car')->load($_GET['left_car_id']);

    $right->add('View_Car')
        ->setModel('Car')->load($_GET['right_car_id']);

Creating View
-------------

The View_Car should come with it's own template, containing picture
and also list of pros and cons for each car::

    class View_Car extends View {

        function setModel($m) {
            $m = parent::setModel($m);

            $this->add('Lister',null,'Pros','Pros')
                ->setModel($m->ref('Pros'));

            $this->add('Lister',null,'Cons','Cons')
                ->setModel($m->ref('Cons'));
        }

        function defaultTemplate() {
            return [ 'view/car' ];
        }
    }

Next looking inside ``view/car``::

    <div class="atk-cells atk-cells-gutter-large atk-box">
        <div class="atk-cell">
            <h3>{$name}</h3>
            <p class="atk-push-small">{$dsecr}</p>
        </div>
        <div class="atk-cell">
            <h4>Pros:</h4>
            <ul>{Pros}<li>{$name}</li>{/}</ul>

            <H4>Cons:</h4>
            <ul>{Cons}<li>{$name}</li>{/}</ul>
        </div>
    </div>

The important part to see here is how the Pros region can be seamlessly
used inside Lister to repeat it's content necessary amout of times.

Pay attention to {$name} and {$descr} tags. Those are defined in the template,
and are being initialized inside View's render to the values of the
associated model - that is a standard behaviour of View with model.


Creating model
--------------

The above code also relies on Model_Car, so this section will go through
the steps to create one. In this example I want to store car data
in Mongo database (although this can easily be any other data source).

The pros and cons fields will actually contain list of comma-separated items::

    class Model_Car extends Model {
        public $table = 'car';

        function init() {
            parent::init();

            $this->addField('name');
            $this->addField('descr')->type('text');
            $this->addField('pros');
            $this->addField('cons');

            $this->setSource('Mongo');
        }

        function ref($rel) {
            if($rel == 'Pros') {
                $items = $this['pros'];
            } elseif ($rel == 'Cons') {
                $items = $this['cons'];
            } else return parent::ref($rel);

            $items = explode(',', $items);
            $m = $this->add('Model');
            $m->addField('name');
            $m->setSource('Array',$items);

            return $m;
        }
    }

The init() method uses our regular pattern to define some fields and links
our model with Mongo source.

Method :php:meth:`Model::ref` allows you to traverse between object relations.
In our case I'm extending functionality of this method, to create two pseudo-relations:
Pros and Cons. Those will return a model of elements based on field data.

Because Mongo can actually store arrays, I can enhance my model further to
take advantage of that::

    class Model_Car extends Model {
        public $table = 'car';

        function init() {
            parent::init();

            $this->addField('name');
            $this->addField('descr')->type('text');
            $this->addField('pros')->system(true);
            $this->addField('cons')->system(true);

            $this->setSource('Mongo');
        }

        function ref($rel) {
            if($rel == 'Pros') {
                $field = 'pros';
            } elseif ($rel == 'Cons') {
                $field = 'cons';
            } else return parent::ref($rel);

            $self = $this;
            $m = $this->add('Model');
            $m->addField('name');
            $m->addHook('afteSave,afterDelete', function() use($self) { $self->saveLater(); })
            $m->setSource('Array',$this->data['items']);

            return $m;
        }
    }

I have also created some hooks, to trigger the save of the Car model when either
pros or cons have changed.

.. todo: Must test this example.
