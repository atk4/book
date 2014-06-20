**********
SQL Models
**********

Introduction
============

Relational Model base class (Model\_Table) extends standard models and
enhances them with various features. If you are already familiar with
other ORM (Object Relational Mapper), you might wonder why it was
necessary for Agile Toolkit to have its own Model implementation.

The fact is that Agile Toolkit is incredibly agile in the way how you
are able toco

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

Fundamentals of DSQL
====================

To understand DSQL you should realize that it works in a very similar
way to how other objects in Agile Toolkit working. This consists of two
parts:

1. Query parameters are set and are saved inside a property.
2. Query is rendered, parameters are converted and query is executed.

Here is the example of a sample query template::

    select [myexpr]

DSQL can have only one template at a time. Calling some methods will
also change the template. You can explicitly specify template using
``expr()``.

Before query is executed, DSQL renders the template. It searches for the
regions in brackets and executes method ``render_<text in brackets>``::

    function render_myexpr(){
        return 'now()';
    }

now to try and put this all together you would need to use this code::

    class MyDSQL extends DB_dsql {
        function render_myexpr(){
            return 'now()';
        }
    }

    $this->api->dbConnect();
    $q = $this->api->db->dsql('MyDSQL');
    $q->expr('select [myexpr]');
    echo $q->getOne();

In this example, we define our own field renderer and then reference in
the expression. The resulting query should output a current time to you.

DSQL Argument Types
-------------------

When you build the query by calling methods, you arguments could be:

1. Field Names, such as ``id``
2. Unsafe variables, such as ``:a``
3. Expressions such as ``now()``

::

    $q=$this->api->db->dsql()->table('user');
    $q->where('id',$_GET['id']);
    $q->where('expired','<',$q->expr('now()'));
    echo $q->field('count(*)')->getOne();


The example above will produce query::

    select count(*) from `id`=123 and `expired`<now();

Value from ``$_GET['id']`` is un-safe and would be parametrized to avoid
injection. "``now()``\ " however is an expression and should be left as
it is.

It is a job for the renderer to properly recognize the arguments and use
one of the wrappers for them. Let's review the previous example and see
how to use different escaping techniques::

    class MyDSQL extends DB_dsql {
        function render_test1(){ return 123; }
        function render_test2(){ return $this->bt(123); }
        function render_test3(){ return $this->expr(123); }
        function render_test5(){ return $this->escape(123); }
        function render_test6(){ return $this->consume(123); }
    }

Those are all available escaping mechanisms:

1. ``bt()`` will surround the value with backticks. This is most useful
   for field and table names, to avoid conflicts when table or field
   name is same as keyword ``("select * from order")``
2. ``expr()`` will clone DSQL object and set it's template to first
   argument. DSQL may surround expressions with brackets, but it will
   not escape them.
3. ``escape()`` will convert argument into parametric value and will
   substitute it with sequential identifier such as ``:a1, :a2, :a3``
   etc.
4. Finally, ``consume()`` will try to identify object correctly and adds
   support for non-SQL objects such as Model Fields.

Exercise
--------

*This exercise is only for your understanding. For normal usage of DSQL
you don't need to do this.*

Let's review how the "set" operator works in "update" query. First,
let's use the following simplified template::

    update [table] set [set]

To help users interface with our template, we must have the following
two methods::

    funciton table($table){
        $this->args['table']=$table;
        return $this;
    }

    function set($field,$value){
        $this->args['set'][]=array($field,$value);
        return $this;
    }

It's important that you don't use any escaping mechanisms on the
arguments just yet. They may refer to expression which can still be
modified from outside. The arguments are packed into an internal
property "args". Next, let's review the rendering part of the arguments.
This time I'll be using different escaping mechanisms in different
situations::

    funciton render_table(){
        return $this->bt($this->args['table']);
    }
    function render_set(){
        $result=array();
        foreach($this->args['set'] as list($field,$value)){
            $field=$this->bt($field);
            if(is_object($value)){
                $value=$this->consume($value);
            }else{
                $value=$this->escape($value);
            }
            $result[]=$field.'='.$value;
        }
        return join(', ',$result);
    }

Table would need to be back-ticked and we don't really need to worry
about expressions. For the "set" rendering things are bit more complex.
We allow multiple calls to ``set()`` and then we need to produce the
equation for each field and join result by commas. The first argument,
the field, needs to be back-ticked. Second argument may be an object,
but if it's now, it most probably would contain an unsafe parameter, so
we use ``escape()`` to convert it into parametric value.

Consume here would recursively render the expression and join the
parameters from the subquery to our own. In some situations we would
need to surround consume with a brackets, if SQL syntax requires it.

NOTE: This exercise is mentioned here only to help you understand DSQL.
You must not try to re-implement rendering of table or set arguments as
it is already implemented properly in DSQL. Let's take look at other
templates DSQL supports.

DSQL Use
========

With DSQL you can set arguments first and then decide on the template
for your query. DSQL recognizes a number of SQL query templates::

    'select'=>"select [options] [field] [from] [table] [join]
        [where] [group] [having] [order] [limit]"

    'insert'=>"insert [options_insert] into [table_noalias]
        ([set_fields]) values ([set_values])"

    'replace'=>"replace [options_replace] into [table_noalias]
        ([set_fields]) values ([set_value])"

    'update'=>"update [table_noalias] set [set] [where]"

    'delete'=>"delete from [table_noalias] [where]"

    'truncate'=>'truncate table [table_noalias]'

To render the templates, all the respective renderers exist (such as
render\_options, render\_field, render\_from, etc). Data is typically
stored in property "args" until execution. Below is the list of
functions you can call on the ``dsql()`` object to affect the queries::

    option("ignore");
    field($field, $table, $alias); // uses multiple call formats
    table($table, $alias); // uses multiple call formats
    where($field,$condition,$value); // uses multiple formats
    join($table, $master_field, $join_type); // uses multiple formats
    group("field");
    order($field, $desc); // uses multiple formats
    limit($count,$offset);
    set($field,$value); // used by insert, replace and update to set new field values

When using any of the mentioned templates, the property "mode" of dsql
object will be set to the name of the template (such as 'update' or 'insert').

Handling DSQL Object
--------------------

I have already mentioned property "template", "args" and "mode", but
there are more properties. All of them are defined public and you can
access them for reading, but avoid changing them.

params
    During rendering of the query, PDO parameter values are stored here.
stmt
    Contains PDO object, in case you need to use some custom PDO values or properties
main_table
    When using joins, this contains the name (or alias) of main table, which is
    sometimes prefixed to the fields.
default_field
    Contains name of the field to use if you don't set any fields. Asterisk by default.
default_exception
    will throw Exception\_DB by default
param_base
    prefix used when creating params for PDO
debug
    can be switched on by calling debug() method. Will echo query before executing it.
sql_templates
    contains array with pre-defined modes and templates
id_field
    can be set if you wish that DSQL would return id as key when iterating.

Cloning and duplicating DSQL
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Any DSQL object can be safely cloned. It will NOT however clone any
arguments.

Alternatively you can call ``dsql()`` method, which will create new
instance of the same class, very similar to ``newInstance()``.

If you are using ``expr()``, it will return a new object with the
template as you specify in the argument. This is done to leave your
original query intact. If don't want new object, ``useExpr()`` will
change template without cloning.

Overloaded arguments
~~~~~~~~~~~~~~~~~~~~

Similarly how you could specify different argument types is jQuery
methods, you can specify different argument types to the methods. Keep
your code clean by using most effective one, e.g. use where('id',123)
instead of where('id','=',123)

Setting data
------------

By calling various methods, you record relevant data in DSQL object for later
execution.

Specifying Table (table())
~~~~~~~~~~~~~~~~~~~~~~~~~~

Use this argument to set a primary table or even multiple tables for
your query. When specifying multiple tables, you would most likely need
to have where condition. Call like this::

    table('user')
    table('user','u') // aliases table with "u"
    table('user')->table('salary') // specify multiple tables
    table(array('user','salary')) // identical to previous line
    table(array('u'=>'user','s'=>'salary')); // specify aliases and tables

Getting field with table prefix
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a single table is used, you can use method getField($field) to
generate expression with appropriate table prefix and back-ticked field.
Please do not confuse this with ``getElement()`` or
``model->getField();``::

    $q=$this->api->db->dsql();
    $field = $q->table('book','b')->getField('id');

    // will contain expression for: `b`.`id`

Specifying fields (field())
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Can add physical fields or fields expressed with formula (expressions)
to the query. Can also alias fields. Formats::

    field('name')
    field('name','user') // second argument specified table
    field('name','user')->field('line1','address');
    field('name,surname'); // multiple fields in one go
    field(array('name','surname')); // can specify as array
    field(array('n'=>'name','s'=>'surname')); // use aliases for fields
    field(array('n'=>'name'), 'user'); // combine explicit table and
    field alias
    field($q->expr('year(now)-year(birth)',age'); // alias expression
    field(array('age'=>$q->expr('year(now)-year(birth)'))) // multiple expressions
    field($q->dsql()->table('book')->field('count(\*)'),'books'); // subquery
      // if you don't call field() at all, "\*" will be selected.

Adding Conditions (where)
~~~~~~~~~~~~~~~~~~~~~~~~~

Methods we have reviewed till now in details have been using "safe"
arguments. When you are using where() you can use un-safe argument.
While we still advise you to perform input validation from $\_GET(), but
in normal circumstances it's quite safe to use those as the last
argument to where.

Below are execution examples::

    where('id',1) // argument 1 here is unsafe, will use parametric variable
    where('id','>',1) // can use any operation. Second argument must be safe
    where('id>',1) // same as above, but only limited number of operators are supported: <,>,<=,>=,<>,!=,in,not in,like,is and spaces
    where('expired','<',$q->expr('now()')) // expression overrides unsafe variable, will not use param
    where('age',$q->expr('between 5 and 18')) // can be used with safe variables
    where('age',$q->expr('between [min] and [max]')->setCustom('min',$min)->setCustom('max',$max)); // use this with un-safe variables
    where('book_id', $q->dsql()->table('sale')->field('book_id')); // subquery
    where('state',array('open','closed','resolved')); // state in ('...','...')
    where($q->expr('length(password)'), '>', $maxlen);

Expression when you are calling where() multiple times are joined using "and"
clause. DSQL also supports OR, but multiple OR conditions are considered a
single where() condition. For example::

    select user where is_active=1 and (id=1 or id=2)

When you look at the query like this, you should see that area in brackets is
actually an expression. A method orExpr() can help you produce this expression::

    where('is_active',1)->where($q->orExpr()->where('id',1)->where('id',2));
    where('is_active',1)->where(array(array('id',1),array('id',2))); // same, shortcut
    where('is_active',1)->where(array('name is null','surname is null'));

The second and third format here would use orExpr() internally, but
possibly is a little bit more readable (especially if you write it on
multiple lines).

Finally if you want to compare two fields, you can use something like
this::

    where('income','>',$q->expr($q->bt('expense')); // where `income`>`expense`
    where('income','>',$q->getField('expense'));    // better one. Use this.

The second format here will also use a table prefix for expense, which
is nice when you are using multiple joins. You can also use ``getField``
for the first argument::

    where($q->getField('income'),'>',$q->getField('expense'));

Finally, you can also use ``andExpr()``, which works identical to
``orExpr()``::

    $q=$this->api->db->dsql()->table('book');
    $q->where('approved',1)->where($q->orExpr()
        ->where('state','submitted')->where($q->andExpr()
            ->where('state','pending')->where('filled',1)));

Resulting query::

    select * from book where `approved`=1 and
        (`state`='submitted' or
            (`state`='pending' and `filled`=1)
        )


Using having()
~~~~~~~~~~~~~~

The "``having()``" method is described as an alias to ``where()``
except that the result for all ``having()`` clauses will be stored in a
separate location and will render under a separate tag. Anything written
above for ``where()`` also apply for ``having()``.

Joining tables join()
~~~~~~~~~~~~~~~~~~~~~

With this method, you can create all sorts of joins and query data from
multiple tables. Join will use default field names, which follow the
following principle: "If you have "book" and "author" table, joining
them would use ``book.author_id=author.id expression``".

I advise you to use this field naming convention in your database
design, which will simplify your code and make it more readable. The
examples below, assume that you already have a query object and the
primary table is set::

    $b=$this->api->db->dsql()->table('book');

Join syntax::

    join('author'); // join author on author.id=book.author_id
    join(array('a'=>'author')); // same, but sets alias to `a`
    join('chapter.book_id'); // join chapter on chapter.book_id=book.id
    join(array('c'=>'chapter.book_id')); // same but sets alias to `c`
    join(array('a'=>'author','c'=>'chapter.book_id')); // two joins

If you are not using the standard convention you can use second argument to
specify the field in your main table (book)::

    join('author','written_by'); // join author on author.id=book.written_by
    join('info.issn','issn'); // join info on info.issn=book.issn

If you need a further flexibility with the join, you can use expressions.
When combined with orExpr / andExpr it gives you quite powerful tool::

    join(
        'info',
        $q->andExpr()
            ->where(
                'info.issn',
                $q->getField('issn')
                    ->where('info.is_active',1)
            )
    )
    // join info on info.issn=book.issn and info.is_active=1

The third argument can be used to change the join type. It is an unsafe argument::

    join('info',null,'inner'); // inner join. By default "left" join is used

Using grouping (group())
~~~~~~~~~~~~~~~~~~~~~~~~

Group specifies how the results will be grouped (group by)::

    group('sex');
    group($q->expr('year(created)'));
    group('sex,age');
    group(array('sex',$q->expr('year(created)')));

Ordering results (order())
~~~~~~~~~~~~~~~~~~~~~~~~~~

Ordering can be done my one or several fields and in two directions::

    order('age');
    order('age','desc'); // reverse order
    order('age',true); // same as above, reverse oredr
    order('age,sex'); // order by age, sex (multiple fields)
    order(array('age','sex'));
    order('age desc, sex');
    order(array('age desc','sex'));
    order($q->expr('year(birth)'));


Advanced features
-----------------

Options
~~~~~~~

There are two methods for setting options::

    option('calc_found_rows'); // option for select
    option_insert('ignore');   // option for insert

More option methods could be added later.

Using setCustom
~~~~~~~~~~~~~~~

If DSQL template encounters a tag which it can't find method to render,
it will next check ``args['custom']`` to see if it might have been set
directly. This allows you to set a custom values for the expression
tags. If you use one of the pre-defined tags, then your value will
override the default rendering mechanism. ``addCustom()`` uses
``consume()`` escaping on it's argument::

    $q=$this->api->db->dsql()->expr('show table [mytag]');

Examples::

    addCustom('mytag','123'); // produces 123
    addCustom('mytag',$q->bt('user')); // produces ``user``
    addCustom('mytag',$q->expr('foo bar'); // produces foo bar
    addCustom('mytag',$q->escape(123)); // produces :a1 param

Deleting arguments
~~~~~~~~~~~~~~~~~~

By calling method ``del()`` you can get rid of some arguments which have
been set before::

   field('bleh')->del('fields')->field('name'); // select name from ..
   reset(); // will delete all arguments

The most popular use is when you need to change fields on arbitrary
query::

    $y=clone $q;
    $count = $y->del('fields')->field('count(*)')->getOne();

Calling Methods (call() and fx())
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When you call your own methods in SQL the typical syntax is::

    call myfunc(args)

To do this with DSQL, you should use the following pattern::

    $q=$this->api->db->dsql();
    $q->call('myfunc',array($arg1, $arg2));

To call standard functions you can use fx() method::

    $q=$this->api->db->dsql();
    $q->call('myfunc',array($arg1,
        $q->dsql()->fx('if',array($x1,$x2,$x3))
    ));

    // call myfunc($arg1, if($x1,$x2,$x3))

Note: ``call()`` and ``fx()`` will not clone your query, so use dsql()
if necessary.

Using set()
~~~~~~~~~~~

You should use set() for insert and update. In fact, you can use
something like this::

    $q=$this->api->db->dsql();
    $q->set('name','John')->set($user_data);
    $q->where('username',$username)->limit(1);
    if(!$q->get()){
        $q->insert();
    }else{
        $q->update();
    }



Why you shouldn't use DSQL directly
-----------------------------------

Within DSQL you might find the comfort and control over SQL queries, but
you must try to avoid using DSQL directly. Relational Models are higher
level objects which give you much simpler syntax with very minimal
performance hit.

Cross-SQL compatibility methods and shortcuts
---------------------------------------------

Gradually, as Agile Toolkit starts supporting more database engines, it
will implement more compatibility methods. Currently the following
methods are present::

    concat(arg,arg,...) // string concatination
    describe(table) // select table schema
    random() // return random value 0..1
    sum(field) // returns expression for sum(..)
    count(field) // returns expression for count(..). Argument is '\*' if unspecified.

Querying
========

Calling any of the methods below, will execute your query. Those methods
take no arguments, but will render query collecting previously specified
data.

-  execute() // will not change template. Use with your own expression.
-  select()
-  insert()
-  update()
-  replace()
-  delete()
-  truncate()

Additionally there are some combinations of executing query and fetching
data.

-  get() // returns array of hashes containing data
-  getOne() // returns scalar value, first row first column
-  getRow() // return first row only, but as non-associative array
-  getHash() // return first row as a hash
-  fetch() // returns one row as hash. Calling this or previous two
   methods multiple times will return further data rows or null if no
   more data is there.

Iterating through results
~~~~~~~~~~~~~~~~~~~~~~~~~

You can use DSQL as an iterator::

    $q=$this->api->db->dsql();
    $q->table('book')->where('is_active',1);
    foreach($q as $data){
        $qq = clone $q;
        $qq->where('id',$data['id'])
            ->set('data',json_encode(..))
            ->update();
    }

Using preexec
~~~~~~~~~~~~~

Normally, while you build the query you can set an options, but the
query is only executed when you start iterating through it. If you wish
to execute query quicker you can use ``preexec()`` method. This allows
you to have some information about the query even before you start
iterating.

You can also execute multiple queries simultaneously for better
performance::

    $q=$this->api->db->dsql();
    $q->table('book')->where('is_active',1);
    $rows = $q->calc_found_rows()->preexec()->foundRows();
    foreach($q as $data){
        $qq = clone $q;
        $qq->where('id',$data['id'])
            ->set('data',json_encode(..))
            ->update();
    }

In this example, we were able to determine number of $rows in result set
before we started iterating through the selected cursor and avoided
executing query twice.

Debugging
---------

You can use $dsql->debug() to turn on debugging mode for the query. This
will output some data using "echo" when the query is being rendered.
DSQL will use font colors / HTML to highlight different parts of query.

Examples:
========

DSQL - Practical Examples
-------------------------

For this chapter we are going to explore the following database schema:

-  Book ( title, author_id, is_published )
-  Chapter ( book_id, no )
-  Author

I will review number of tasks::

    $db = $this->add('DB')->connect($dsn);

    // Get All Books
    $q = $db->dsql()->table('books');
    $books = $q->get();

    // Get all books of particular author
    $book_by_author = $q->where('author_id', 1)->get();

    // Delete all books by particular author
    $q->delete();

Compared to other query builders - DSQL is more fluid, allows calling
methods in any order and has a straightforward syntax::

    $q = $db->dsql()->table('user');
    $q->where($q->orExpr()
        ->where('id',1)
        ->where($q->andExpr()
        ->where('age','>',25)
        ->where('votes','>',100)
        )
    );
    $users = $q->get();

    $users = $db->dsql()->table('user')
        ->join('phone.user_id')
        ->field('users.email,phone.number');

    $q = $db->dsql()->table('user');
    $q->join('phone',$q->orExpr()
        ->where('user.id=phone.user_id')
        ->where('user.id=phone.contact_id')
    );
    $users = $q->get();

    $min_age = $db->dsql()->table('user')
        ->field('min(age)')->do_getOne();

    $q=$db->dsql()->table('users');
    $q->set('votes',$q->expr('votes+1'));
    $q->update();

