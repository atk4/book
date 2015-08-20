*********************
Controller_Data_Clusterpoint
*********************

.. todo:: This page is Under Construction.......
.. todo:: This page is Under Construction.......
.. todo:: This page is Under Construction.......




.. php:class:: Controller_Data_Mongo

    Implements Model Integration with Clusterpoint (http://www.clusterpoint.com/)


This controller implements model access to Clusterpoint::

    $m = $this->add('Model');
    $m->addField('name');
    $m->addField('surname');
    $m->addFiled('email');
    $m->addFiled('type');

    $m->setSource('Clusterpoint', 'user');

    $this->add('CRUD')->setModel($m);

The table name corresponds with the collection. The ID field used
by Mongo is called ``_id``, however the ID in your model will be
set according to :php:attr:`Model::id_field` property.

Connecting
==========

You need three things in order to connect to Clusterpoint database:

- end-point
- username
- password

You can generate username/password for API-only access through Clusterpoint
database console.

You can either specify connectiong arguments when performing setSource()
or you can store arguments in the configuration file.

Connecting to Public Database
-----------------------------

Clusterpoint has several public databases, which can be easily used inside
your existing database. When defining source for Clusterpoint, you can
specify the entire collection parameters. Based on information available
through https://github.com/clusterpoint/osm, we can create
a model to retrieve Open Street Map Points of Interests::

    $m = $this->add('Model');
    $m->setSource('Clusterpoint', [
        'https://api-eu.clusterpoint.com/298/osm_poi/',
            'user'=>'osm', 'pass'=>'openmaps' ]);

The above method specifies all connection details to the driver directly
and this connection overriding your config settings.

Next you can perform various operations with your model::

    $m->addCondition('amenity/type','cafe');
    $m->search('starbucks');

    foreach($m as $point){
        echo $point['lat'].', ' $point['lon'].'<br/>';
    }

Connecting to your Private Database
-----------------------------------

You can create your own database in the cloud with Clusterpoint. To
connect and access data in your own database, we recommend that you
use configuration file for storing your connection credentials. Your
clusterpoint account allows you to create multiple databases and we
Agile Toolkit makes it easy to access them.

First, add this into your configuration file::

    $config['cp']['endpoint'] = 'https://api-eu.clusterpoint.com/298/';
    $config['cp']['user'] = 'myuser';
    $config['cp']['pass'] = 'secret';

Secondly, when you need to access specific database, you can use::

    $m = $this->add('Model');
    $m->setSource('Clusterpoint', 'mydb');

You should remember that Clusterpoint does not define columns so you
would need to define your own database structure using addField()::

    $m->addField('name');
    $m->addField('email');
    $m->addfield('type');

Features
========

Clusterpoint database driver implements basic CRUD features for you,
so you can easily test your connectivity with this::

    $page -> add('CRUD')->set($m);

This will allow you to add, delete, update and save records. You
will not be able to update any of the public databases, as they are
read-only.

You can also operate with model directly by using save(), load(),
delete() methods::

    $cnt = 0;
    foreach($m as $record){
        echo $id.' => '.json_encode($point->get()).'<br/>';
        if($cnt++>20) break;
    }

or::

    $m->load(123123); // load by ID
    $m['foo'] = 'bar';
    $m->save();  // store data (will not reload)


This code will dump first 20 records from your database.

In addition to the basic features, Agile Toolkit allows you to
access vendor-specific features of Clusterpoint.


Searching
---------

Clusterpoint incorporates amazing full-text search capability. When
you setSource() on your model, you will have access to $model->search().
Search operates similar to Condition on a model::

    // Search
    foreach($m->search('starbucks') as $point){
        echo $point['lat'].', ' $point['lon'].'<br/>';
    }

Calling search() multiple times will operate as "AND". Passing
array to search will operate as "OR"::

    $wikipedia->search(['clusterpoint','mysql']);
    $wikipedia->search('php');

    foreach($wikipedia as $article){
        echo $article['title'].'<br/>';
    }

The above example will look for articles in Wikipedia (hosted
and updated through Clusterpoint) which contains "PHP" and
one of "clustepoint" or "mysql" anywhere in title or body.

For security reasons, arguments to search will be escaped, so
if you want to use XML expressions inside search, you would
need ot use::

    $osm->_search("<amenity>[0]</amenity>",[$amenity]);

.. note:: For syntax on search expressions, refer to Cluserpoint docs.

Searching in Shapes
-------------------

Clusterpoint allows you to look for points based on geographical locations.
Agile Toolkit model implements shapes() method that will allow you to
define shapes that will later be used for matching. The order in which
you deifne search() and shapes() is not important::

    foreach(
        $m
            ->search('>< polygon')
            ->shapes(['polygon'=>$point_array])
            as $point
        ) {
            //
        }

Advanced Conditions
-------------------

Clusterpoint allows you to specify conditions through some means. You
must remember that 2nd argument to addCondition is always escaped for
XML characters::

    $m->addCondition('type', 'cafe'); // exact match.
    $m->addCondition('type', '1..4'); // binary match to 1..4, quoted to avoid injection

If you want to supply your own XML (be sure to do it safely) you can also use::

    $m->addCondition('<type>1..4</type>'); // range match

Finally, if second argument is passed as an array, then you can use expression.
This is preferred way to specify range, to avoid any XML injection::

    $m->addCondition('<type>[0]..[1]</type>', [1,4]);
    $m->addCondition('<type>[from]..[to]</type>', ['from'=>1, 'to'=>4]);

Driver will look for the first argument to start with '<' as XML should. If
you specify array, then it will be cosider to be list of allowed values (OR)::

    $m->addCondition('type', ['cafe','bench']); // exact OR match.

If you specify one array argument to addCondition() then the contens of
this array will be converted into XML (using encode_xml). That allows you to
build any structure here::


    $m->addCondition(['type'=>'1..4']); // again - binary match, auto-quoted
    $m->addCondition(['type'=>$m->expr('1..4')]); // range match, similar to dsql->expr()

All of the above rules also apply to loadBy() and tryLoadBy().

Aggregation
-----------

Clusterpoint query can contain aggregation part that will summarize the result.
When calling aggregate(), this will automatically set limit to 0 and further
iterating the model will give you aggregation results. If you call limit()
on such a model, it will instead limit aggregation results::

    $m->aggregate('group by ...');

Information
-----------

Along with the query results, clusterpoin returns some useful information.
You can access it through $m>-info;

- number of results
- execution time

Efficient use
=============

Clusterpoint being a cloud database has some latency when you load records.
Sometimes you want to load multiple records or save multiple records::

    $m->preload([ids, ..]);

This will use loadMultiple and execute it as a background operation. You
will still need to execute $m->load() later, however if "id" was previously
preloaded, then the record will be loaded from cache::

    $m->preload([34,35]);

    // do something else

    $m->load(34);
    // do something
    $m->saveLater();

    $m->load(35);
    // do something
    $m->saveLater();


Clusterpoint model will automatically use updateMultiple() before your
application terminates to store results above. As a result, your code
above will execute only 2 requests instead of 4. Additionally it will
impact minimally performance of your application as you can do other
things.

In Agile Toolkit we recommend you to put preload() inside your init()
methods and then use load() inside render() if it's possible.

Transactions
============

Two additional methods, which may be useful for your model are beginTransaciton()
and commit()::

    $m->beginTransaction();
    $m->load();
    // do something
    $m->save();
    $m->commit();

This code ensures than other connections will not be able to modify your
record while you are in the "do something" block. If something was changed,
then commit() will throw Exception_Conflict. Here is the code to handle
exception correctly::

    try {
        $m->beginTransaction();
        $m->load();
        // do something
        $m->save();
        $m->commit();
    }catch(Exception_Conflict $e){
        // handle exception
    }

.. note:: Clusterpoint exceptions are not associtated with the requests,
    so you can even create transaction between loading and storage of the
    form;

::

    $form = $this->add('Form');
    $m_cp = $form->setModel('Cpmodel');
    $form->addField('hidden', 'cp-transaction')->set($cp->beginTransaction());
    $m_cp->load($this->app->stickyGET('cpmodel_id'));

    $m_cp->onSubmit(function($form){

        $form->save();
        try {
            $m_cp->commit($form['cp-transaction']);
            return 'Form Saved';
        }catch(Exception_Conflict $e){
            return $form->error('Data changed in database. Refresh page');
        }

    });


Actual Fields
=============

Agile Toolkit supports concept of actual fields. if you specify
this, clusterpoint will respect them and will only load data for
those fields::

    $m->setActualFields(['title', 'category', 'tags' => [ 'amenity' ]]);
    // will load only requested fields. If saved, will replace only those fields.

You can also use setActualFields(true) which will load use defined field
list to load data::

    $m->addFiled('lat');
    $m->tryLoadAny();
    echo $m['lon'];   // will output <lon> even though it wasn't defined


    $m->addFiled('lat');
    $m->setActualFields(true);
    $m->tryLoadAny();
    echo $m['lon'];   // will raise exception, as field is not defined

You should be aware, that if field is not defined, it won't be saved::

    $m->addFiled('lat');
    $m->tryLoadAny();
    $m['lon'] = $m['lon'] + 1;
    $m->save();     // will NOT store lon, as it wasn't defined as a field.






