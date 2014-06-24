**********
Validation
**********

Agile Validation Library
========================

This is a powerful validation library for your Agile Toolkit
applications and more.

Goals of Agile Validation
-------------------------

There are very few validation libraries with a simple and consise
syntax, so I have made this Addon.

Agile Validation continues on the design principles of the framework,
and is very similar in it's design to jQuery Chains or DQSL chaining.
Here is a typical validation query:

::

    $validator->is('name|len|<5?Name is too short');

The library allows us to create number of rules, which are sequentally
checked in a consistent pattern. Here are some of my goals which I
achieve with Agile Validation Library:

1. Minimalistic syntax
~~~~~~~~~~~~~~~~~~~~~~

Developers avoid validation, because it requires them to write a lot of
code. Validation should be easy to define and easy to update. Current
libraries require the use of complex structures and require a lot of
writing.

Agile Toolkit already provides exception-based validation, where your
model can perform any validation you can think of inside beforeSave or
afterLoad hooks. The mechanism of displaying errors, checking the fields
is there, but this approach often requires developer to write many lines
of code instead of short set of rules.

With Agile Validation Library you can define the rules in a short and
smart way:

::

    $model->add('Validator')
    ->is([
        'name,surname!|len|5..100',
        'addr1_postcode|to_alpha|to_trim|uk_zip?Not a UK postcode',
        'addr2_postcode|if|has_addr1|as|addr1_postcode'
       ])->on('beforeSave');

It might seem that the above code is cryptic, but once you learn to read
the rules, you will love using them in your application.

Let me quickly guide you through the example above.

-  the example features 3 rule-sets, each applied and checked
   individually.
-  the rules are applied on the fields of a $model object
-  rules are checked before model is saved
-  first rule-set applies to 2 fields - name and surname:
-  "!" will require both fields (??? or only last field ???) to be
   non-empty
-  "len" convertor will apply any further rules on the length of the
   field
-  5..100 specifies allowed range of length of name and surname
-  if name or surname is missing or their length is outside of specified
   range, appropriate error message will be shown
-  second rule-set is applied on addr1\_postcode field and it uses some
   "normalizers":
-  field will be filtered for alpha characters and trimmed down
-  after normalization it's checked against format of UK post code
-  "uk\_zip" rule is using a custom error message, which will appear
   underneath a form's field
-  third rule is applied on addr2\_postcode field (post code for 2nd
   address):
-  rule is applied only if field "has\_addr1" is true
-  rule inherits all rules from the addr1\_postcode field - therefore
   will be normalized too

2. Great Unique Concept
~~~~~~~~~~~~~~~~~~~~~~~

The validation is implemented as an add-on, therefore you don't have to
use it if you don't like this approach. The concept of a validation
engine is based on few key concepts which you must understand before
using.

-  Field selector - specifying fields
-  Rule processing - how rules are applied
-  Rule types - filters, convertors and others
-  Using rules to normalize field value
-  Aliases and shorter syntax
-  Logical operations
-  Using closures/callbacks
-  Adding your own rules
-  Extending validator for special fields (e.g. upload image validation)

3. Integration
~~~~~~~~~~~~~~

Validator can work on it's own but is neatly integrated with the rest of
Agile Toolkit. You have control over when validator actions are
performed. By default validation occurs before saving model, but this
can be changed.

You can also use validator with a simple array (hash) or any other
object which supports array-access, even objects from outside of Agile
Toolkit. In such case, to specify where data is stored, you can use
``with()`` method.

4. Error Messages
~~~~~~~~~~~~~~~~~

Writing meaningful error-messages is a huge problem. That's why one of
the goals of Agile Validator is to automatically generate reasonable
error messages. (??? Localization of error messages ???)

Getting Started
---------------

To create your first validation, use the following inside your model's
init method:

::

    $this->add('Validator')
        ->is('age|numeric|>10')
        ->is('email|email');

Next create a CRUD with your model and try adding values:

-  age as string
-  age value 10 or less
-  email in incorrect format

You should see a proper error message below the field when trying to
save these incorrect values.

Understanding Rules Format
--------------------------

Ruleset is a set of rules and consists of 2 parts:

-  field selector
-  one or multiple rules

In the most basic form, rule-sets look like this:

::

    $validator->is('field|rule|rule|rule');

or

::

    $validator->is('field','rule','rule','rule');

While using pipes sometimes is easier to read especially if you know how
pipes work in UNIX. You can also specify unlimited number of arguments
to the ``is()`` method and this approach is slightly more flexible
because in such case You can use pipe as part of the value too.

1. Rule Processing
~~~~~~~~~~~~~~~~~~

Validator stores all the rules you have defined, but does not apply them
until certain call-back occurs. If you have added validator inside
``Model`` then default call-back is ``beforeSave``. If you have added
validator inside ``Form`` then rules are checked during ``submit``
event.

You can manually process the rules too by calling ``now()`` method or
specify a different hook with ``on()``.

Until rules are processed you can add as many rules as you like. You can
also group rules inside an array and feed them inside the validator:

::

    $validator->is(array(
       'field|rule|rule',
       'field|rule|rule'
      ))

or if would like to avoid pipes:

::

    $validator->is(array(
        array('field','rule','rule'),
        array('filed','rule','rule')
      ));

2. Argument Consumption
~~~~~~~~~~~~~~~~~~~~~~~

Some rules take arguments. For example ``in`` rule is used to check if
the value exists in an array. The next argument after ``in`` is
considered to be list of possible values instead of a rule:

::

    $validator->is('state','in','paid,draft,new,old');

or

::

    $validator->is('state','in',array('paid','draft','new','old'));

You must check documentation of a specific rule if you want to know how
many arguments it takes.

3. Aliases
~~~~~~~~~~

To make syntax shorter, a number of special rule formats are used. For
instance:

::

    $validator->is('age','>18');

is the same as

::

    $validator->is('age','gt',18);

You must remember that every short syntax also have a long-syntax behind
the scenes.

Field Definition
----------------

First argument always defines field or fields. The validators method
``expandField`` is responsible for converting the notation into list of
fields.

Examples:

-  including fields:
-  single field: "email"
-  multiple fields: "email,name,surname"
-  wildcards: "\*\_date" or "user\*"
-  all fields: "\*" (special case of wildcards)
-  excluding fields - starts with dash "-":
-  all fields except name and surname: "\*,-name,-surname"
-  all fields matching "**date" exclude matching "accept*":
   "**\ date,-accept*\ "
-  excluding takes precedence

Validator will process this during the designated time (such as
beforeSave). Use of asterisk or wildcard assumes that your data source
is either extended from Model, has method ``getAllData()`` (??? what is
getAllData()? maybe getRows() ???) or can be passed to ``array_keys()``.

NOTE: You can use only alpha-numeric and underscore symbols for field
names!

Specifying array of fields
~~~~~~~~~~~~~~~~~~~~~~~~~~

You may specify a list of fields using array. Next example will create
one rule-set which will be applied on 2 fields and require both to be
specified.

::

    $validator->is(array('name','email'),'!');

Next example is to remind you that ``is()`` may also take first argument
as "multi-array" (??? is it really multi-array ???):

::

    $validator->is(array('name!','email!'));

In this case two rules will be created, each on one field and they would
require that field to be specified. Further on I will no longer point
out different ways to specify rulesets except where it's important, so
keep in mind all the possibilities.

Model field groups
~~~~~~~~~~~~~~~~~~

Models supports field groups:

::

    $model->addField('has_addr')->type('boolean');
    $model->addField('address')->group('addr');
    $model->addField('zip')->group('addr');

You may now specify fields by group:

::

    $validator->is('[addr]|if|has_addr')

You can use asterisk or wildcard symbol too:

::

    $validator->is('[*addr]|if|has_addr')

Exclamation sign
~~~~~~~~~~~~~~~~

NOTE: Validation rules are only there for validation. They will NOT
affect presentation of the form. That's why you can still specify field
types, display options and other flags inside field definition.

Exclamation sign may appear at the end of field or any rule:

::

    $validator->is('name!');
    $validator->is('username|to_alpha!');

The use of exclamation sign as shown above will convert into the
following rules:

::

    $validator->is('name|trim|required');
    $validator->is('username|to_alpha|trim|required');

Trim will remove initial, trailing and duplicate space. If you don't
wish to trim the value, then you should use full-formatted 'required'
rule:

::

    $validator->is('name','required'); // will allow you to use "  " as name

Field comparison
~~~~~~~~~~~~~~~~

You may use equation sign inside field definition to compare two fields.
Here is short example and resulting rule:

::

    $validator->is('pass2=pass1');    // same as:
    $validator->is('pass2','eqf','pass1');

Question mark
~~~~~~~~~~~~~

If you finish the field with a question mark, then it's considered to be
a mandatory field with a user-defined error message:

::

    $validator->is('name?type your name here');   // same as:
    $validator->is('name|trim|required?type your name here');

??? Localization ???

Use with Model's Fields
~~~~~~~~~~~~~~~~~~~~~~~

Agile Toolkit models can invoke your validator of choice if you:

1. Define property $validator\_class in your model. By default it's
   "Validator", but you may use your own class.
2. Call ->validate() method on a field.

The following code:

::

    $model->addField('age')->validate('int|>20?You must be over 20');

is identical to this code:

::

    $model->add($model->validator_class)->is('age|int|>20?You must be over 20');

NOTE: validate() method return field object for chaining purposes and
not Validator object.

Rule Definition
---------------

As you learn more about validator, you must understand one important
concept about how it works:

1. When new rule is processed, the value is copied from the data-source
   into a temporary variable, which I'll call ``accumulator``.
2. Rules have access to ``accumulator``, and name of the field.
3. Rules may "read" next ruleset arguments and use them as an arguments
   for validation.
4. Rules may access other fields of current data-source record.

Filter rule
~~~~~~~~~~~

If rule looks at ``accumulator`` and then throws exception based on
condition, then it's called a **Filter Rule**:

::

    function rule_int($acc)
    {
        $v = $acc;
        if (!is_int($v)) {
            throw $this->exception('Must be int');
        }
        return $acc; // always return original value
    }

Some of the filter rules are: ``int``, ``regexp_match``, ``email``,
``alpha``

NOTE: Filter rule don't change original and ``accumulator`` values.

Convertor rule
~~~~~~~~~~~~~~

If rule looks at ``accumulator`` and returns non-null value, then this
new value is stored inside ``accumulator`` for the next ruleset
operation. Rule like that is called **Convertor Rule**:

::

    function rule_len($acc)
    {
        return strlen($acc); // return changed value
    }

Some of convertor rules are: ``trim``, ``len``, ``date_diff``

NOTE: Convertor rule don't change original value. It change only
``accumulator`` value if needed.

Normalization rule
~~~~~~~~~~~~~~~~~~

Often neglected by developers, normalization makes sure your user-input
looks nice and clean. For example, when users enter email addresses,
they often leave spaces around it or when specifying number may
accidentally paste some character, such as enter or tab along with the
number.

It's better that those values are cleaned up before they are saved. Many
of the rules can be used with the "to\_" prefix. This will cause
validator to update the data source with the value of ``accumulator``
after rule processing is complete.

For example (??? both work the same ???):

::

    email|to_trim|to_email

or

::

    email|trim|to_email

If you add a trailing pipe (??? I don't like trailing pipe idea. There
should be another symbol used in this case. Pipe are for separating
elements and let it be so ???) to your validation rule, then this will
copy ``accumulator`` back into the data source:

::

    email|trim|email|

I highly encourage you to use normalization in your software. But You
must use it with caution, as use of normalization can sometimes cause
undesired results:

::

    email|trim|len|>5|

This will replace email with it's length because of trailing pipe.
Probably not what you wanted. The correct rule would be:

::

    email|to_trim|len|>5

NOTE: Normalization rule can change original and ``accumulator`` values.

Multi-field operations, copy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sometimes you would like to perform operation between multiple fields,
such as storing length of one filed inside another or splitting a field
into two fields. This can be done by applying convertors carefully and
using rule ``copy``. This rule will copy value of ``accumulator`` from
the field you specify.

::

    ->is('name|copy|full_name|trim|s/.* //|')
    ->is('surname|copy|full_name|trim|s/ .*//|')
    ->is('name_length|copy|name|len|')

NOTE: Basically ``copy`` will change original field value to
``accumulator`` value which is passed as an infinitely long argument
from sub-rule.

Aliases
~~~~~~~

It's only safe to use alphanumeric characters, digits and underscores
for rules and values when you use pipe-delimited rule format. Other
characters are generally reserved for aliases. For rules that's OK,
because rules are using PHP-method names anyway.

When rule contains any other characters, it is considered to be an alias
and validator will try to convert it into a regular rule. Aliases below
are listed in order in which they are verified:

-  ``a-z`` -> alpha
-  ``a-z0-9`` -> alpha\_num (??? case sensivity ???)
-  ``0-9a-z`` -> alpha\_num (??? case sensivity ???)
-  ``!`` -> mandatory
-  ``2..4`` -> between\|2\|4
-  ``>4`` -> gt\|4
-  ``!=5`` -> ne\|5 (??? this can be hard to distinguish from
   ``required`` or ``mandatory`` rule ???)
-  ``b-z`` -> regexp\_match\|/ [1]_\*$/
-  ``/^a/`` -> regexp\_match\|/^a/
-  ``s/a/z/`` -> to\_regexp\|a\|z (??? what is z and why we need
   trailing slash ???)

Error messages
~~~~~~~~~~~~~~

Each rule have an appropriate error message defined. For example, rule
">20" produces message "{{caption}} must be more than {{arg1}}".

If you have used some convertors they may also alter error message:
"length of {{caption}}"

::

    "Length of Name must be more than 20"

You can specify a custom error message if you append it through question
mark to a rule:

::

    >20?Must be over 20

All error messages are passed through exceptions which also implies that
error messages will be localized using ``$this->api->_($error)``. Refer
to localization documentation for further information. (??? there are no
fully working built-in localization support in ATK :) ???)

If you are willing to specify some fancy error message with dangerous
characters you can use the following format:

::

    $validtor->is('age','int?','Must be numeric');

When rule also expects an argument, then the argument for that rule must
come first.

::

    $validator->is('age','!=?','10','Age must be 10');

(??? I don't like !=? because ! and = are two different rules and should
be separated with pipe ???)

Inside error-message you can also use some of the parameters:

-  {{name}} - actual name of the field (e.g. user\_name)
-  {{caption}} - caption of the field (if model), label of form or
   otherwise same as {{name}}
-  {{arg1}} - first argument for a rule
-  {{arg2}} - second argument for a rule
-  {{arg3}} and so on.

Use of closures
~~~~~~~~~~~~~~~

Previously I have explained how rule\_X methods are called and how they
are being passed an ``accumulator``. If you specify a closure as a rule,
then this closure is called. The first argument is a validator object,
second argument is ``accumulator`` and third is name of the field. You
can interface with validator to perform more complex operations. See
below "Validator's Methods". (??? There are no such chapter "Validator's
Methods" ???)

Example:

::

    ->validate('birthdate',function($v,$acc){
        $d = new DateTime($acc);
        return $d->diff(new DateTime())->format('%y')
    },'>=18?Must be at least {{arg1}} years old');

Comparison
~~~~~~~~~~

Naming of comparison rules are inspired by the UNIX bash comparison
operations. ``>5, <5, >=5, <=5, =5, !=5`` are changed into
``gt, lt, ge, le, eq, ne``. All of the methods will consume next
argument and use it as a value to compare with. If the argument is
array, then the contents of this array is considered to be a sub-rule.

Sub-rules
~~~~~~~~~

Frankly - with what have been done so far, sub-rules is a intuitive next
step. Sub-rules will pause the processing of your rule to go through
another rule and then substitute it with resulting ``accumulator``.

::

    $validator->is(
        'password1',
        'len',
        'eq?Password length must be the same',array('password2','len')
    );

You can also call sub-rules explicitly by using ``as`` rule. While
normally the argument for ``as`` is name of the field, from which rules
are collected, it can also read rules from an array argument.

For example: (??? Need example here ???)

Macros / Use of non-existent fields
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can use rule system to create non-existent fields and then reference
them:

::

    $validator->is(array(
        '#myemail|s/.*<//|s/>.*//|to_trim|email',
        'client_email|as|#myemail',
        'billing_email|as|#myemail'
    ));

Let's take this to another level as we usually can with Agile Toolkit:

::

    class MyValidator extends Validator
    {
        function init()
        {
            parent::init();
            $this->is(array(
                '#myemail|s/.*<//|s/>.*//|to_trim|email',
                '#zip|s/.*<//|s/>.*//|to_trim|email',
            ));
    }

Next you can specify this validator for your model and rely on those
nonexistent fields which can now be used as a macro:

::

    $validator->is('email|as|#myemail');

If you specify error message to ``as``, it will use it instead of the
error message generated inside the sub-rule.

::

    $validator->is('email|as?Does not match fancy email format|#myemail');

And and Or
~~~~~~~~~~

You may rely on And / Or logic to define complex dependencies between
multiple fields:

::

    $validator->is(':or', rules1, rules2, rules3)
    $validator->is(':and', rules1, rules2, rules3)

Example:

::

    $validator->is(
        ':or?Must be male over 10y or female over 12y',
        array(':and','gender|=M','age>10'),
        array(':and','gender|=F','age>12')
    )

(??? 1. quite strange syntax. 2. not sure which error message to show -
the one set with "or" rule or the one generated in sub-rule. ???)

Unit conversion
~~~~~~~~~~~~~~~

There are few converor rules to convert your units into ``kb``, ``mb``
or ``gb``. Those convertors would divide ``accumulator`` by 1024
appropriate number of times.

Conditional rule - if (array)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default ``if`` rule consumes next argument and uses it as to see if
the other field is specified. What if you would like to use a more
sophisticated check? If also supports sub-rules

::

    $validator->is('addr','if',array('method','=','deliver'))

You can also use a call-backs as an argument:

::

    $validator->is('addr', 'if?Must specify address if you deliver',
        function($v, $addr, $addr_name, $data) {
            return $data['method'] == 'deliver';
        })

Rule 'if' will consume up to 3 arguments if you specify them. You can
skip argument by supplying null or just empty string. The first argument
can be a call-back or sub-rule. If second argument is not specified,
then the field will simply be validated as mandatory if call-back
function or sub-rule in first argument will return "true". If second
argument is specified it is then used as a rule, which will only apply
when if is true. Third argument is "else"-rule.

::

    $validator->is('delivery_to','if','home','[home_addr]!','[work_addr]!')

The only way how you can omit arguments is by leaving ``if`` as a last
rule. Having ``if`` in your rule-set will not bypass any rules prior to
it.

Comparing fields
~~~~~~~~~~~~~~~~

When you use comparison operatiors either by their alias ('=') or by
using the rule name 'eq', you specify the value:

::

    $validator->is('gender','=M')
    $validator->is('gender','=','M')
    $validator->is('gender','eq','M')

If you want to compare with other field you can either specify it inside
the field or use one of the methods with "f" at the end:
``eqf, nef, ltf, gtf, lef, get``:

::

    $validator->is('pass1=pass2')
    $validator->is('pass1','eqf','pass2');

Member of array
~~~~~~~~~~~~~~~

using "in" and "!in" (or not\_in) you can verify if element is inside
set of allowed values:

::

    $validator->is('gender|in|M,F')
    $validator->is('gender','in',array('M','F'))

The second format allows you to use any value inside array, they can
even contain commas or pipes.

Extension - Validator\_Table
----------------------------

Agile Toolkit implements validator through a number of classes:

-  AbstractValidator - implements only rule logic, but no actual fields.
   Conditions, comparisons and sub-rule logic is implemented. Will also
   support ``Model`` data source.
-  Validator - adds rules which are commonly used for validation.
-  Validator\_Table - assumes that you use ``Model_Table`` and introduce
   number of ORM-based extensions.

``Validator`` rules have been explained above, however
``Validator_Table`` offers the following enhancements:

Avoiding duplicates (unique)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In some cases you would like to avoid to have multiple user accounts
with same email. Rule 'unique' will attempt to find another record with
same email and if it is found, will produce error.

::

    $validator->is('email|unique');

Looking up inside DSQL
~~~~~~~~~~~~~~~~~~~~~~

Using $dsql, you may specify a sub-query which can be used for lookup of
valid elements:

::

    $validator->is('email','in',$dsql);

Validator will perform a query such as: "select 1 if [field] in ($dsql)"
(with proper quotes).

Verify other model
~~~~~~~~~~~~~~~~~~

You may create a rule, which attempts to load a record from a separate
model:

::

    $validator->is('user_id','loadable','User');

You may also avoid the model:

::

    $validator->is('user_id','loadable');

if you define the field through ``hasOne``. With model integration this
rule can be as simple as:

::

    $this->hasOne('User')->validate('loadable');

Extension - Validator\_Image
----------------------------

When you use Filestore add-on, it also introduces additional validator
class, which can be used to perform further checks on image field. Here
is a syntax:

::

    $model->add('filestore/Field_Image')
        ->validate('format|=jpeg')
        ->validate('size|mb|<5');

You can call validate() method several times or specify array with
rules.

-  format - returns format of uploaded image
-  size - returns size of uploaded file in bytes
-  height - returns height in pixels
-  width - returns width in pixels

Validation defined on the image field by default will be using an upload
hook, so that error will be displayed not during form submission, but
when you are still uploading the image.

More Examples
-------------

::

    $validator->is(array(
     'email|to_email|!',           # convert to email and must not be empty
     'base_price|to_int|10..100',  # convert to int, and must be within range
     'postcode|to_upper|to_trim|to_A-Z|postcode',
                                   # clean up postcode then validate
     'pass1=pass2',                # passwords should match
     'country_code|upper|in|UK,US,DE,FR',
                                   # uppercase for comparison only
     'addr2|asif|addr1',           # validate addr2 like addr1 if addr1 is present
     'hobby|s/[^,]//|len|>5?Max of 5 hobbies can be specified'
    ));

Changing hook
~~~~~~~~~~~~~

As I mentioned, by default validation is performed on beforeSave hook of
the model. If used with form, the validation is performed during submit.
It is possible to change the hook for a specific rule by using @hook.
This is converted into "on" rule:

::

    $validator->is('name|to_lower|@afterLoad')
    $validator->is('name|to_lower|on|afterLoad')
    $validator->is('name','to_lower','on',array($api,'post-init'))

This will affect only a single rule and may result in creation of
another copy of Controller\_Validator, so use ->on method of a validator
instead of using this for every single rule.

Further Ideas
-------------

Validator can normalize rule definitions as often explained above.
Although this will not support all of the cases, normalization can be
pretty awesome if you are willing to do client-based validation (in
browser or mobile app)

::

    $validator->getRules('field');

This will return array of rule-sets like this:

::

    array(
        array('rule','rule','rule',$arg,'rule'),
        array('rule','rule','rule')
    );

Each rule name would be expressed using alpha-numeric and underscore.
Argument can be value or array of values.

.. [1]
   b-z
