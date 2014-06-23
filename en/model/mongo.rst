**************
MongoDB Models
**************


MongoDB is one of the most popular NoSQL engine implementing
document-oriented storage. This type of storage can greatly simplify
many areas that are that are a challenge for traditional relational
modelling, such as ecommerce products. Mongo is fast, convenient and
relatively easy to learn: if you have a suitable application, I strongly
encourage you to try it.

Agile Toolkit offers a full-featured Mongo Model which integrates your
Mongo data smoothly with CRUD, Grid and other View components using the
syntax you already know from ``Model_SQL``.

This document will help you get started quickly even if you're new to
Mongo. See also:

- :php:class:`Controller_Data` - introduction to Data Controllers
- :php:class:`Model` - introduction to base models

Installing Mongo
----------------

Mongo is easy to install.

On Lunux you should be able to install Mongo from a distro package. Then
run: ``$ pecl install mongo`` to set up the PHP driver, add
``extension=mongo.so`` to your php.ini file, restart PHP and you're good
to go.

On Windows you can find detailed guidance on the `MongoDB
website <http://docs.mongodb.org/manual/tutorial/install-mongodb-on-windows/>`__
or use a WAMP stacks such as `Z-WAMP <http://zwamp.sourceforge.net/>`__
that includes MongoDB.

Once you're set up, you should be able to connect from your
command-line:

::

    bash> mongo
    MongoDB shell version: 2.4.4
    connecting to: test
    >

In PHP, you can run ``phpinfo()`` to confirm that your driver is
available.

Creating a MongoDB based CRUD
-----------------------------

Start by creating a new Model:

::

    class Model_Test extends Mongo_Model {

        public $table='test';

        function init()
        {
            parent::init();

            $this->addField('name');
            $this->addField('gender')->enum(array('M','F'));
            $this->addField('over_18')->type('boolean');
        }
    }

Then add this to your page:

::

    $this->add('CRUD')->setModel('Test');

And finally, in your config file:

::

    $config['mongo']['db']='mydb';

That's all you need to set up a working CRUD! You don't need to manually
create your table structure: Mongo handles this for you on the fly. If
you need an additional field, simply declare it in the Model and you're
done.

Implementation Details
----------------------

MongoDB Model support is implemented using the new Controller\_Data,
therefore if you look into the
"`Mongo\_Model <https://github.com/atk4/atk4/blob/master/lib/Mongo/Model.php>`__\ "
class you'll find that it's pretty short. The majority of the logic
lives inside
`Controller\_Data\_Mongo <https://github.com/atk4/atk4/blob/master/lib/Controller/Data/Mongo.php>`__.

This makes our Mongo implementation compatible with other data
controllers such as
`Array <https://github.com/atk4/atk4/blob/master/lib/Controller/Data/Array.php>`__,
`MemCache <https://github.com/atk4/atk4/blob/master/lib/Controller/Data/Memcached.php>`__
and
`Session <https://github.com/atk4/atk4/blob/master/lib/Controller/Data/Session.php>`__.

Specifying The Model Collection
-------------------------------

A 'collection' in Mongo is roughly equivalent to a 'table' in SQL.

For compatibility, you specify a collection using the ``$table``
property in your Model.

Traversing references
---------------------

Mongo doesn't support SQL joins, so you can't use SQL-specific
``join()`` syntax.

But Mongo does support referencing between documents, allowing us to
traverse references in both directions as you would with relations in
``SQL_Model``:

::

    // In Model_Book:

    $this->hasMany('Chapter');
    $this->hasOne('Author');

So just as with SQL Models, you can do this in your code:

::

    $author = $book->ref('author_id');
    $chapters = $book->ref('Chapter');

Conditions
----------

Mongo\_Model supports conditions in the format you're used to:

::

    $model->addCondition('over_18', true);

This will affect new records you create as well as limit records which
your Model can access.

Mongo provides support for advanced conditions, which you can access
like this:

::

    $model->addCondition('age', array('$gt'=>18, '$lt'=>65));

Or you can access the full power of the Mogo query language like this:

::

    $model->addCondition('$where',
       "function() { return this.name == 'Joe' || this.age == 50; }");

    $model->addCondition('search_index', new MongoRegex("/^$prefix/"));

    $email->addCondition('$or', array(
            array('from_id'=> new MongoID($user->id)),
            array('to_id'=> new MongoID($user->id))
        ));

You should refer to
`PHP <http://php.net/manual/en/mongocollection.find.php>`__ and
`MongoDB <http://docs.mongodb.org/manual/reference/operator/>`__
documentation for more information.

ID Field
--------

++++++++++++++++++++++++++++++++++++++++++++++++++++++ TODO: Can't make
any sense of this: can you clarify?
++++++++++++++++++++++++++++++++++++++++++++++++++++++

While we recommend using an ``id`` field in SQL, Mongo uses the ``_id``
field internally.

If you want to write a portable code, you should rely on the
``$model->id`` property instead.

::

    $model['user_id']=$this->api->model->id;

Field types
-----------

While SQL usually stores only string values in its fields (or something
which can be expressed by a string, such as integers), Mongo can store
practically any data-type in any field. For example:

::

    $model->addField('interests')
        ->defaultValue(array('fishing','reading'))
        ->system(true);

Because array values aren't supported by Form or Grid (yet), I've set
the ``system`` flag on this field to keep it from appearing in the UI.
You can, however, access it easily:

::

    $model->load($id);
    $this->template->set('interests', join(',', $model['interests']));

But there's a gotcha here. Due to limitations in PHP, you can't modify
the Model value with array syntax:

::

    // This won't work
    $model['interests'][]='cycling';

So you need to use a temporary variable:

::

    // This is the way to go
    $tmp = $model['interests'];
    $tmp[]='cycling';
    $model['interests'] = $tmp;

Hopefully this limitation will be fixed in future versions of PHP.

You should also keep in mind, that reference fields (fields containing
IDs) are using special objects of type ``MongoID``. Fortunately, Model
will handle this for you in the background, so if you specify a
reference ID as a string (passed from GET, for example) it will
automatically be converted to ``MongoID`` when your Model is saved.

When you load a Model ``MondoId`` will be cast to a string, so you can
use it in URLs:

::

    $this->api->url('author/info', array('id' => $book['author_id']]));

Hooks And Behaviours
~~~~~~~~~~~~~~~~~~~~

You'll find that ``Mongo_Model`` contains all the same hooks you're used
to in ``SQL_Model``: ``beforeSave``, ``afterSave``, ``beforeInsert``,
``afterUpdate``, ``beforeUpdate``, ``afterDelete``, ``beforeDelete``,
``afterDelete``, etc.

Obviously there is no DSQL query passed to the callback handler, as
there is with ``SQL_Model``, but you can still modify Model settings.
Here is a interesting way to index your collection:

::

    $this->addField('search_index')->system(true);

    $this->addHook('beforeSave', function($m){

        $m['search_index'] = array_map('strtolower', array_map('trim'
            , explode(' ', $m['name'].' '.$m['email'])));
    });

    $this->db()->ensureIndex('search_index');

Accessing The PHP Collection Object
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As you've just seen, Mongo Model makes it easy for you to access the
`MongoCollection class
object <http://www.php.net/manual/en/class.mongocollection.php>`__
directly – simply call ``$model->db()``.

And through ``MongoCollection``, you can access powerful features such
as `the Mongo aggregation
framework <http://www.php.net/manual/en/mongocollection.aggregate.php>`__:

::

    $ops = array(
        array(
            '$project' => array(
                "author" => 1,
                "tags"   => 1,
            )
        ),

        array('$unwind' => '$tags'),

        array(
            '$group' => array(
                "_id" => array("tags" => '$tags'),
                "authors" => array('$addToSet' => '$author'),
            ),
        ),
    );

    $result = $article_model->db()->aggregate($ops);

You can also access the MongoDB class `for features such as text
query <http://docs.mongodb.org/manual/tutorial/search-for-text/>`__:

::

    $results = $article_model->db()->db->command(array(
        'text'=>$article_model->table,
        'search'=>$search
    ));

Using Cursors
-------------

The Mongo driver offers
`cursors <http://php.net/manual/en/class.mongocursor.php>`__, which can
be used elegantly within the Toolkit.

You'll recall that ``Lister``, or ``Grid`` can operate with any object
implementing the ``Iterator`` interface, so you can display your results
like this:

::

    $grid->addColumn('line','name');
    $grid->addColumn('line','age');
    $grid->setSource($cursor);

Other Toolkit Features That Simply "Work"
-----------------------------------------

Many other features of Agile Toolkit play nicely with Mongo:

-  :php:class:`Paginator`
-  Column sorting (:php:method:`Model::sortable`)
-  Limits and setOrder (:php:method:`Model::setLimit`, :php:method:`Model::setOrder`)
-  :php:meth:`CRUD::addRef`
-  :ref:`Model iterating`
-  :doc:`/addons/filestore`
-  And many more.

But if you find anything that doesn't work as advertised, please `submit
a bugreport
