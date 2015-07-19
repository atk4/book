
Model Relations
===============

Relations can be established between any two models regardless of their
data source. There are 4 types of relations that Agile Toolkit supports:

- hasOne implements many to one relation `$order->hasOne('User')`
- hasMany implements one to many relations `user->hasMany('Order')`
- containsOne implements contained model `user->containsOne('home_address','Address')`
- containsMany implements contained model with multiple records `user->containsMany('basket','Basket_Item')`

.. note:: When using with SQL models, containsOne and containsMany needs to be JSON-encoded. Other
  data stores may already implement nested structures.

Usage
-----

You can use any of the above methods in model definition. Each will have some
assumptions about how models relate and when traversin will add necessary
conditions.



Implementation Details
----------------------

Relationship can be defined by calling one of the following methods:

- :php:class:`Model::hasOne` - implemented by :php:class:`SQL_One`
- :php:class:`Model::hasMany` - implemented by :php:class:`SQL_Many`
- :php:class:`Model::containsOne` implemented by :php:class:`Relation_ContainsOne`
- :php:class:`Model::containsMany` implemented by :php:class:`Relation_ContainsMany`

.. note:: Test cases are here: https://github.com/atk4/atk4/tree/master/tests/atk43_model_relations

Once defined, it adds a new child to the model of a specific relation class. Calling
:php:class:`Model::ref` will call method `ref()` of a corresponding object.


hasOne and hasMany
------------------

.. todo:: todo


containsOne and containsMany
----------------------------

Sometimes we need to keep data within the model when it is saved. This helps
us to keep number of different tables/collections to the minimum and groups
data nicely.

The main question you should be asking - "Can this model exist on it's own?"

A good example for containsOne could be address. Having just "address" on
the system is not benificial. It's always address of the company or
the person. Person can have multiple addresses too::

    $person->containsOne('home_address','Address');

that means that further calling::

    $person->ref('home_address');

will return a Model_Address. Once saved then the data will be stored
inside `$person['home_address']` field as a hash.


Use of `containsMany` is similar, however it is stored as array of hashes::

    $person->containsOne('basket','Basket_Item');

that means that further calling::

    $person->ref('basket')->tryLoadAny();

Defining Inline Model
^^^^^^^^^^^^^^^^^^^^^

In some cases, the contained model will only be used once. In this case
you can define model in-line without additional class. The above example
with a Basket_Item could be a good example, because basket-item will only
relevant to a person and is not needed outside of this model::

    $this->containsMany('basket',function($m){
        $m->hasOne('Item');
        $m->addField('qty');
    });

In this case "Model" object will be created and fields will be populated.

Type of stored data
^^^^^^^^^^^^^^^^^^^

When saving contained model, the data is not written to database just yet,
but it will be placed inside a contained model's field. For example,
if you have 2 orders in your basket, then your `$user->get()` would
return structure like this::

    'id' => 4,
    'name' => 'John Doe',
    'basket' => [
      '55ab87ad2e546' => [ 'item_id' => 273, 'qty' => 2 ],
      '55ab87ad2e66e' => [ 'item_id' => 270, 'qty' => 1 ],
    ]

It's also useful to use contained model when you retrieve data from
API and simply need to use standard views to format the data.

Options and Filters
^^^^^^^^^^^^^^^^^^^

When defining contained model, you can specify options and filters::

    $user->containsMany(['basket','json'=>true], 'Basket_Item');

In this case the data will be stored like this::

    'id' => 4,
    'name' => 'John Doe',
    'basket' => '{"55ab87ad2e546":{ "item_id":273, "qty":2 },"55ab87ad2e66e": { "item_id": 270, "qty": 1 }}'

which is suitable to be saved into SQL.

There are other parameters you can pass in addition to `xml` above:

- 'encode', 'decode' - use custom methods when storing or loading data. This is useful if we want to encrypt data.

Here are some ideas for future implementation (NOT IMPLEMENTED YET)

- 'lazy' can use to prevent field from loading by default. It will be excluded form actual fields, but will be
  loaded selectively if you decide to traverse into it. Caling get() on the field will return null.
- 'model_opts' specify some options for model initialization
- 'model_class' override default class, which is ‘Model’ when using second argument as a callback.
- 'no_hooks' will not set save hooks
- 'relation_class' - use a custom relation class

