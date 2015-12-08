**********
SQL Models
**********

Introduction
============

Relational Model base class (Model\_Table) extends standard models and
enhances them with various features. If you are already familiar with
other ORM (Object Relational Mapper), you might wonder why it was
necessary for Agile Toolkit to have its own Model implementation.


Goal: Unified factory and entity
--------------------------------

In a traditional design, you require to have a factory object (or class)
which will produce objects for each data record. As I have already
mentioned in the previous chapter, the Model implementation in Agile
Toolkit unifies both purposes together. The primary reason for this
decision is the nature of a short-lived PHP requests. While the separate
class design is better for threaded Java, in PHP you would need to
create two classes per model on every request.

Unifying everything into a single model comes at a cost and thats why a
single model object is allowed to load different data entries. The same
principle applies to the relational model.

Goal: Simplification of SQL
---------------------------

As you describe the model, it will behind the scenes build the DSQL
object which can be used for accessing, saving and deleting your model
data. The nature of DSQL which allows it to perform multiple queries
perfectly matches the needs of reusable models. At the same time, the
power and flexibility of DSQL can still be accessed, if you want to
optimize your queries.

In typical ORM design, you must either use their limited features or use
completely different way to address database. For example, you normally
are unable to perform action on multiple records through ORM interface,
and if you wish to do so, you would need to come up with a full query
yourself.

Agile Toolkit models can provide you with a pre-made DSQL object for you
which you can extend and modify.

Goal: Integrity of DataSet
--------------------------

Relational models perfectly applies the concept of data-set to your
database. You can define multiple models which access the same table,
but would have a different set of conditions and therefore would have
different data-sets.

Agile Toolkit ensures that you would not be able to accidentally go
outside of the data-set when you query, update, delete or insert data.


Relational Model Fields
=======================

Model uses objects to describe each of it's field. Therefore when you
create a model with 10 fields, it would populate at least 12 objects:

Model's object. One object for each field. DSQL object for a model.
Fields object play a very direct role in creating queries for the model
such as select, update and insert queries.

Normally you would rely on the standard Model Field class, which is
populated when you call ``addField()``, however you should know that
it's possible to extend fields significantly.


Expressions
~~~~~~~~~~~

Normally, when Model builds a query it asks every field listed in the
"actual fields" list to add itself into ``dsql()``. Fields will call the
``field()`` method to update the query of the model, from inside of the
``updateSelectQuery()``. The "Field\_Expression" redefines the method to
insert user-defined expression into the model query.

Additionally expression field will also change number of flags.
``editable()`` will return false.

Expression filed introduces one new property "expr" which stores your
expression which can be expressed either as a string, as a DSQL
expression or as a call-back. Use ``set()`` to specify the expression.
Model have a method ``addExpression()``, which will create expression
field for you::

    $model->addExpression('full_name')
        ->set('concat(name," ",surname)');

When you are building expressions, be mindful that the fields you are
referenced are SQL fields and are not model fields.


Relational Model Use (ORM)
==========================

Let's do a quick review. First, we have created and abstracted SQL
queries through a query builder. Next we have created and abstracted
model fields. Now we need to tie them together through our ORM
implementation and this will give us table abstraction. Lets create
model for "Book"

::

    class Model_Book extends Model_Table {
        public $table='book';
        function init(){
            parent::init();
            $this->addField('title');
            $this->addField('is_published')->type('boolean');
            $this->addField('cost')->type('money');
        }
    }

Manipulating Model Fields
-------------------------

::

    $m=$this->add('Model_Book');

    $m['title']='Jungle Book';
    $m['year']=123;

    var_Dump($m->get()); // shows title and year

The important thing about Agile Toolkit models, is that you can add more
fields dynamically at any time.
:math:`m->addField('abstract');     `\ m['abstract']='Lorem Ipsum ..';

Ability to take an existing model and add more fields allows us to
extend existing models into new ones:

::

    class Model_Published_Book extends Model_Book {
        function init(){
            parent::init();
            $this->addField('issn');
            $this->addCondition('is_published',true);
        }
    }

but not only we can add additional fields, we can also add conditions,
which would permanently change model's data-set.

Loading and Saving Models
-------------------------

::

    $m1=$this->add('Model_Book');
    $m1['title']='Jungle Book';
    $m1->save();
    echo $m1->id;    // will output book's id
    echo $m1['is_published'];    // null

    $m2=$this->add('Model_Published_Book');
    $m2->tryLoad($m1->id);
    echo $m2->loaded();    // false. Condition not met

Let's try this other way around:

::

    $m1=$this->add('Model_Book_Published');
    $m1['title']='Jungle Book';
    $m1->save();
    echo $m1->id;    // will output book's id
    echo $m1['is_published'];    // true
    $m2=$this->add('Model_Book');
    $m2->tryLoad($m1->id);
    echo $m2->loaded();    // true

Models can be loaded by using either ID key or other field:

::

    $m=$this->add('Model_User');
    $m->load(1);    // loads by id

    $m->loadBy('email',$email);

    $m->loadBy('id',$array_of_ids);

    $m->orderBy('title')->tryLoadAny(); // loads first record

So far no surprises. Any model can also produce a DSQL of itself to be
used to build a custom queries:

::

    $m=$this->add('Model_User');
    $m->count()->getOne();    // returns number

    $m->sum('age')->getOne();     // returns age

    $m->dsql()->del('fields')->field('avg(age)')->getOne();
        // custom field query

One of the up-sides of Agile Toolkit ORM is it's support for
expressions. Let's go back to our Book / Chapter example:

::

    $q=$this->add('Model_Book')->dsql()->del('fields')
        ->field('id')->where('is_published',1);
    $c=$this->add('Model_Chapter')
        ->addCondition('book_id',$q);

Now the model $c will have it's data-set dynamically restricted to only
published books. Let's create some data:

::

    $m1=$this->add('Model_Book')->set('name','Jungle Book')
        ->set('is_published',true)->saveAndUnload();

    $m2=$this->add('Model_Book')->set('name','jQuery Book')
        ->set('is_published',false)->save();

    $c->set('name','Jungle Chapter 1')->set('book_id',$m1->id)
        ->save();    // will be successful

    $c->set('name','jQuery Chapter 1')->set('book_id',$m2->id)
    ->save();    // will fail

What about browsing:

::

    echo $c->count()->debug()->getOne();
    // select count(*) from chapter where book_id in
    //  (select id from book where is_published=1)

We can use this technique again for the Section model, however this
time, we will use a method ``fieldQuery()``:

::

    $s=$this->add('Model_Section')
        ->addCondition('chapter_id',$c->fieldQuery('id');

    echo $s->count()->getOne();

    // select count(*) from section where chapter_id in
    // (select id from chapter where book_id in
    //    (select id from book where is_published=1))

This is cool, but too much typing and manipulating with the models. I am
going to show you a small example from the further chapters on how this
can be simplified:

::

    echo $this->add('Model_Book')
        ->addCondition('is_published',true)
        ->ref('Chapter/Section')
        ->count()->getOne();

    // select count(*) from section where chapter_id in
    // (select id from chapter where book_id in
    //    (select id from book where is_published=1))
    // - although may also use "join" if appropriate

As you can see in the examples, you can achieve things on a low level
with some effort, but the low level gets abstracted more and more to
reveal new beautiful syntax.

Model Aliasing
--------------

In some situations you must resort to table aliases to avoid clashes:

::

    // select a.id,a.title,(select count(*) from book b
    // where b.author_id=a.author_id) cnt from book a

The query above shows number of books written by the same author. But
how do you write this code with the ORM? First let's try to build the
query for our expression:

::

    $q=$this->add('Model_Book',array('table_alias'=>'b'))->count();
    $m=$this->add('Model_Book',array('table_alias'=>'a'));
    $m->addExpression('cnt')->set($q
        ->where($q->getField('author_id'),
            $m->getElement('author_id')
    );

This might seem confusing, let's clean it up:

::

    $m=$this->add('Model_Book',array('table_alias'=>'a'));
    $m->addExpression('cnt')->set(
    $m->newInstance(array('table_alias'=>'b'))
        ->addCondition('author_id',$m->getElement('author_id'))
        ->count()
    );

Let's review what's happening there. First we create a model, but we
pass alias property which will affect all the queries generated by that
model to use the specific alias. Next we create a new instance of the
book but this time we use a different alias. At this point we have 2
models, identical, but with different aliases.

Next I'm setting condition on a second model (b) where author\_id field
equals to author\_id of the first model. It's done by passing Field
object into a condition, which will then end up inside a DSQL object and
will be properly expanded.

Finally our (b) model produces query for a count which is then used as
an expression for the 'cnt' field.

Agile Toolkit will properly prefix fields with the table name if more
than one table is used in a query.

Field Expressions
-----------------

Agile Toolkit treats all types of fields with an absolute consistency.
In other words, anywhere where a physical field can be used, you can
also use expression.

Here is an example:

::

    $m=$this->add('Model_Author');
    $m->addExpression('book_cnt')
        ->set($m->refSQL('Book')->count());

    $m->addCondition('age',$m->getElement('book_cnt'));

This will create a model with only authors who wrote same amount of
books as their age is. Someone who is 32 years old and wrote 32 books
would match the condition.

The important point here is that 'age' could be either a physical field
or another expression and regardless of that the model will properly
build the query.

As a developer you have a full abstraction of a field and expressions.

Here is another example, which uses one field expression to build
another field expression:

::

    $m=$this->add('Model_Author');
    $m->addExpression('full_name')
        ->set($m->dsql()->concat(
        $m->getElement('name'),' ',
        $m->getElement('surname')
        ));
    $m->addExpression('full_email')->set(
        $m->dsql()->concat(
            $m->getElement('email'),' <',
        $m->getElement('name'),'>'
    ));

You will begin to appreciate these qualities as your project grows.
Agile Toolkit is not only trying you to help build query. It provides
PROPER abstraction of query fields and expressions.

Any of the models above can be used with views or for addressing
individual records.

Callback and complex subqueries
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When Model is building query, for all the fields which needs to be selected
(actual fields) it is going to generate a field expression and pass
it into :php:meth:`dsql::field`.

You may pass a call back to the :php:meth:`Field_Expression::set` and
create quite complex sub-selects::

    $m->addExpression('last_payment_amount')->set(function($m){
        $m_payment = $m->refSQL('Paments');
        $m_payment->addCondition('is_settled', true);
        $m_payment->setOrder('date', 'desc');
        $m_payment->setLimit(1);
        return $m->fieldQuery('amount');
    });

You can even take the model outside of the callback if you need to have multiple
fileds::


    $m_payment = $m->refSQL('Paments');
    $m_payment->addCondition('is_settled', true);
    $m_payment->setOrder('date', 'desc');
    $m_payment->setLimit(1);

    $m->addExpression('last_payment_amount')->set(function($m) use ($m_payment) {
        return $m_payment->fieldQuery('amount');
    });
    $m->addExpression('last_payment_date')->set(function($m) use ($m_payment) {
        return $m_payment->fieldQuery('date');
    });

While convenient, this approach will generate multiple sub-queries - one for
each field. You should consider having a sub-select instead or you can overload
ref() field.


Reloading of ref() method
-------------------------

By default ref allows user to traverse through relations. It can be used, however,
to add all sorts of advanced traversal techniques.

Here we can associate a complex field type with a model::

    class User extends Model {

        public $table='user';

        function init() {
            parent::init();

            $this->setSource('Mongo');

            $this->addField('name');
            $this->addField('surname');
            $this->addField('interests')->system(true)->defaultValue([]);
        }

        function ref($ref){
            if($ref == 'Interests'){

                $m = $this->add('Model');
                $m->setSource($this['interests']);

                $self=$this;

                $m->addHook('afterSave,afterDelete', function($m_interest) use ($self){
                    $self['interests'] = $m_interest->getRows();
                    $self->saveLater();
                });
            }
        }
    }


Iterating through records
-------------------------

When model is being iterated, it produces the id / data array pair.

::

    foreach($this->add('Book') as $id=>$book){
        echo "$id: ".$book['title']."\n";
    }
    // 1: Jungle Book
    // 2: jQuery Book

The actual model will be also loaded and can be used much more flexibly
than the produced array. Also you must note that Agile Toolkit will
never retrieve a full result set unless you specifically ask for it. The
iterations will happen as you read data. Here is a slightly different
format:

::

    foreach($book=$this->add('Book') as $junk){
        echo $book->id.": ".$book['title']."\n";
    }
    // 1: Jungle Book
    // 2: jQuery Book

Be mindful of the expressions here. Book object is created first, then
it's assigned to
:math:`book variable. Then the object is iterated and stores results in `\ junk
array. We are not really interested in it, but we work with object
directly.

One way is to update records on the fly here or perform traversal:

::

    foreach($book=$this->add('Book') as $junk){
        echo $book->id.": ".$book['title']." by ".
            $book->ref('Author')->get('name')."\n";
    }
    // 1: Jungle Book by Joe Blogs
    // 2: jQuery Book by John Smith

WARNING: traversing references inside a loop is very bad for
scalability. Your previous favorite ORM might left you no other options,
but with Agile Toolkit you can:

::

    $book=$this->add('Book');
    $book->join('author')->addField('author_name','name');
    foreach($book as $junk){
        echo $book->id.": ".$book['title']." by ".
            $book['author_name']."\n";
    }
    // 1: Jungle Book by Joe Blogs
    // 2: jQuery Book by John Smith

You already saw how to create subqueries, but this example adds a join
and pulls field from a join. Note that we are also adding field under a
different alias.

Timestamps
----------

Things like timestamps, soft-delete and other features tend to appear as
a built-in feature in other ORMs. Agile Toolkit has no built-in support
for either, but with the basic features of Agile Toolkit Models you can
implement these features easily.

::

    $model->addField('deleted')->system(true)->type('boolean');
    $model->addCondition('deleted',false);
    $model->addHook('beforeDelete',function($m){
        $m->set('deleted',true)->saveAndUnload();
        $m->breakHook(false);
    });

Timestamps are equally easy to implement:

::

    $model->addField('created')->type('datetime')
        ->defaultValue($model->dsql()->expr('now()'));
    $model->addField('updated')->type('datetime');
    $model->addHook('beforeSave',function($m){
        $m['updated']=$m->dsql()->expr('now()');
    });

How about if we pack this functionality into a nice controller?

::

    class Controller_SoftDelete extends AbstractController {
        function init(){
            parent::init();
            $this->owner->addField('deleted')
                ->system(true)->type('boolean');
            $this->owner->addCondition('deleted',false);
            $this->owner->addHook('beforeDelete',function($m){
                $m->set('deleted',true)->saveAndUnload();
                $m->breakHook(false);
            });
        }
    }

    $model->setController('SoftDelete');    // simple use

Relations
---------

You may define relations between models like this:

::

    $book->hasMany('Chapters');
    $book->hasOne('Author');

Later you can traverse by using ``ref()`` or ``refSQL()``. For example,
let's create a CRUD for editing list of chapters of a current book:

::

    $this->add('CRUD')->setModel(
        $this->add('Model_Book')->load($id)->ref('Chapters')
    );

You might have already noticed one example with refSQL. When you are
defining relations you can also specify which fields to use. The next
chapter will look into Model() in details.


