Template Engine
###############

Agile Toolkit Applications are HTML based. The HTML is collected from various
templates. To actually manipulate the template, the class GiTemplate is used
by a View objects.

.. php:class:: GiTemplate

    A lightweight template engine using a simple ``{tag}`` syntax without logic.
    The class provides template content manipulation operations and rendering.

    The template object is usually accessed at the :php:attr:`AbstractView::template` attribute.


Template Syntax and Features
============================

Agile Toolkit uses it's own lightweight template engine, which reads and
parses tempaltes like this::

    {Greeting}
    Hello, {$user}.

    Your account have been {action}activated{/}.
    {/Greeting}



Opening Tag — alphanumeric sequence of characters surrounded by ``{``
and ``}`` (example ``{elephant}``)

Closing tag — very similar to opening tag but surrounded by ``{/`` and
``}``. If name of the tag is omitted, then it closes a recently opened tag.
(example ``{/elephant}`` or ``{/}``)

Empty tag — consists of tag immediately followed by closing tag (such as
``{elephant}{/}``)

Self-closing tag — another way to define empty tag. It works in exactly
same way as empty tag. (``{$elephant}``)

Region — typically a multiple lines HTML and text between opening and
closing tag which can contain a nested tags. Regions are typically named
with CamelCase, while other tags are named using ``snake_case``::

    some text before
    {ElephantBlock}
      Hello, {$name}.

      by {sender}John Smith{/}

    {/ElephantBlock}
    some text after

In the example above, ``sender`` and ``name`` are nested tags.

Region cloning - a process when a region becomes a standalone template and
all of it's nested tags are also preserved.

Top Tag - a tag representing a Region containing all of the template. Typically
is called _top.

Manually template usage pattern
-------------------------------

Template engine in Agile Toolkit can be used independently, without views
if you require so. A typical workflow would be:

1. Load template using :php:meth:`GiTemplate::loadTemplate` or
   :php:meth:`GiTemplate::loadTemplateFromString`.

2. Set tag and region values with :php:meth:`GiTemplate::set`.
3. Render template with :php:meth:`GiTemplate::render`.


Template use together with Views
--------------------------------

A UI Framework such as Agile Toolkit puts quite specific requirements
on template system. In case with Agile Toolkit, the following pattern
is used.

 - Each object corresponds to one template.
 - View inserted into another view is assigned a region inside parents
   template, called ``spot``.
 - Developer may decide to use a default template, clone region of parents
   template or use a region of a user-defined template.
 - Each View is responsible for it's unique logic such as repeats, substitutions
   or conditions.

As example, I would like to look at how :php:class:`Form` is rendered. The template of form
contains a region called "FormLine" - it represents a label and a input.

When an input is added into a Form, it adopts a "FormLine" region. While the
nested tags would be identical, the markup around them would be dependent on
form layout.

This approach allows you affect the way how :php:class:`Form_Field` is rendered
without having to provide it with custom template, but simply relying on template
of a Form.


+---------------------------------------------------+-------------------------------------------------------+
| Popular use patterns for template engines         | How Agile Toolkit implements it                       |
+===================================================+=======================================================+
| Repeat section of template                        | :php:class:`Lister` will duplicate Region             |
+---------------------------------------------------+-------------------------------------------------------+
| Associate nested tags with models record          | :php:class:`View` with setModel() can do that         |
+---------------------------------------------------+-------------------------------------------------------+
| Various cases within templates based on condition | cloneRegion or get, then use set()                    |
+---------------------------------------------------+-------------------------------------------------------+
| Custom handling certain tags or regions           | :php:meth:`GiTemplate::eachTag` with a callback       |
+---------------------------------------------------+-------------------------------------------------------+
| Filters (to-upper, escape)                        | all tags are escaped automatically, but               |
|                                                   | other filters are not supported (yet)                 |
+---------------------------------------------------+-------------------------------------------------------+
| Template inclusion                                | Generally discouraged, but can be done with eachTag() |
+---------------------------------------------------+-------------------------------------------------------+

Using Template Engine directly
==============================

Although you might never need to use template engine, understanding
how it's done is important to completely grasp Agile Toolkit underpinnings.


Loading template
----------------

.. php:method:: loadTemplateFromString(string)

    Initialize current template from the supplied string

.. php:method:: loadTemplate(filename)

    Locate (using :php:class:`PathFinder`) and read template from file

.. php:method:: reload()

    Will attempt to re-load template from it's original source.

.. php:method:: __clone()

    Will create duplicate of this template object.


.. php:attr:: template

    Array structure containing a parsed variant of your template.

.. php:attr:: tags

    Indexed list of tags and regions within the template for speedy access.

.. php:attr:: template_source

    Simply contains information about where the template have been loaded from.

.. php:attr:: original_filename

    Original template filename, if loaded from file


Template can be loaded from either file or string by using one of
following commands::


    $template = $this->add('GiTemplate');

    $template->loadTemplateFromString('Hello, {name}world{/}');

To load template from file::

    $template->loadTemplate('mytemplate');

And place the following inside ``template/mytemplate.html``::

    Hello, {name}world{/}

GiTemplate will use :php:class:`PathFinder` to locate template in one of the
directories of :ref:`resource` ``template``.

Changing template contents
--------------------------

.. php:method: set(tag, value)

    Escapes and inserts value inside a tag. If passed a hash, then each
    key is used as a tag, and corresponding value is inserted.

.. php:method: setHTML(tag, value)

    Identical but will not escape. Will also accept hash similar to set()

.. php:method: append(tag, value)

    Escape and add value to existing tag.

.. php:method: appendHTML(tag, value)

    Similar to append, but will not escape.


Example::

    $template = $this->add('GiTemplate');

    $template->loadTemplateFromString('Hello, {name}world{/}');

    $template->set('name', 'John');
    $template->appendHTML('name', '&nbsp;<i class="icon-heart"></i>');

    echo $template->render();


Rendering template
------------------

Ultimately we want to convert template into something useful. Rendering
will return contents of the template without tags::

    $result=$template->render();

    $this->add('Text')->set($result);
    // Will output "Hello, World"

***Why not "echo" the result?***

*Agile Toolkit discourages direct output. You may try using "echo" for
debug purposes, but never leave it in production environment. You should
use objects instead, such as "Text". Text will automatically escape
output for browser output too.*

Setting value
~~~~~~~~~~~~~

After template is loaded, you can change contents of any region:

::

    $template->set('name','Peter');

    // Will contain "Hello, Peter"

You can also specify hash to ``set()`` with tag/new value.

Getting value
~~~~~~~~~~~~~

Often templates are used to get values too. Let's put name of the person
we are greeting into uppercase

$ template->set('name', strtoupper($template->get('name')));

::

    // Will contain "Hello, WORLD"

Template cloning
~~~~~~~~~~~~~~~~

When you have nested tags, you might want to extract some part of your
template and render it separately. For example, you may have 2 tags
SenderAddress and ReceiverAddress each containing nested tags such as
"name", "city", "zip". You can't use set('name') because it will affect
both names for sender and receiver. Therefore you need to use cloning.

::

    <div class="sender">
    <?Sender?>
    <?$name?>
    <?$street?>
    <?$city?> <?$zip?>
    <?/Sender?>
    </div>

    <div class="recipient">
    <?Recipient?>
    <?$name?>
    <?$street?>
    <?$city?> <?$zip?>
    <?/Recipient?>
    </div>


    $template=$this->add('SMlite');
    $template->loadData('envelope');        // templates/default/envelope.html

    // Split into multiple objects for processing
    $sender=$template->cloneRegion('Sender');
    $recipient=$template->cloneRegion('Recipient');

    // Set data to each sub-template separately
    $sender->set($sender_data);
    $recipient->set($recipient_data);

    // render sub-templates, insert into master template
    $template->set('Sender',$sender->render());
    $template->set('Recipient',$recipient->render());

    // get final result
    $result=$template->render();

More operations
~~~~~~~~~~~~~~~

You can also ``del('name')`` to empty contents of the region, or
``append('name',' and Steve')``. You can also call ``is_set('name')`` to
find out if such tag exists. Finally methods ``trySet`` and ``tryDel``
can be used if you are not entirely sure if such tag will exist and
would rather have your code do nothing if tag is missing rather than
raise exception.

Operations on multiple tags
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Same tag can be used multiple times. If you will "set" such a tag, then
all regions will be changed. ``get()`` will return contents of a first
region. ``append()`` will add content to each region.

You can also use ``eachTag()`` to iterate through those tags.

``<?mywiki?>Continue to <?page?>about<?/?> page or <?page?>history<?/?> page<?/?>``

::

    $tempalte->eachTag('page',function($val){
        return '<a href="'.$val.'.html">'.ucwords($val).'</a>';
    });

    // Will contain "Hello, WORLD"

If your callback function defines second argument, then it will receive
"unique" tag name which can be used to access template directly. This
makes sense if you want to add object into that region. You can't insert
object into SMlite template, however every view in the system will have
it's template pre-initialized for you.

Views and Templates
-------------------

Now that you understand how raw templates work, let's see how views use
them.

Default template for a view
~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default view object will execute ``defaultTemplate()`` method which
returns location of the template. This function must return array with
one or two elements. First element is the name of the template which
will be passed to ``loadTemplate()``. Second argument is optional and is
name of the region, which will be cloned. This allows you to have
multiple views load data from same template but use different region.

Function can also return a string, in which case view will attempt to
clone region with such a name from parent's template. This can be used
by your "menu" implementation, which will clone parent's template's tag
instead to hook into some specific template

::

    function defaultTemplate(){
        return array('greeting');   // uses templates/default/greeting.html
    }

Redefining template for view during adding
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When you are adding new object, you can specify a different template to
use. This is passed as 4th argument to ``add()`` method and has the same
format as return value of ``defaultTemplate()`` method. Using this
approach you can use existing objects with your own templates. This
allows you to change the look and feel of certain object for only one or
some pages. If you frequently use view with a different template, it
might be better to define a new View class and re-define
``defaultTemplate()`` method instead.

::

    $this->add('MyObject',null,null,array('greeting'));

Accessing view's template
~~~~~~~~~~~~~~~~~~~~~~~~~

Template is available by the time ``init()`` is called and you can
access it from inside the object or from outside through "template"
property.

::

    $grid=$this->add('Grid',null,null,array('grid_with_hint'));
    $grid->template->trySet('my_hint','Changing value of a grid hint here!');

In this example we have instructed to use a different template for grid,
which would contain a new tag "my\_hint" somewhere. If you try to change
existing tags, their output can be overwritten during rendering of the
view.

How views render themselves
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Agile Toolkit perform object initialization first. When all the objects
are initialized global rendering takes place. Each object's ``render()``
method is executed in order. The job of each view is to create output
based on it's template and then insert it into the region of owner's
template. It's actually quite similar to our Sender/Recipient example
above. Views, however, perform that automatically.

In order to know "where" in parent's template output should be placed,
the 3rd argument to ``add()`` exists — "spot". By default spot is
"Content", however changing that will result in output being placed
elsewhere. Let's see how our previous example with addresses can be
implemented using generic views.

::

    $envelope=$this->add('View',null,null,array('envelope'));

    // 3rd argument is output region, 4th is template location
    $sender=$envelope->add('View',null,'Sender','Sender');
    $receiver=$envelope->add('View',null,'Receiver','Receiver');

    $sender->template->trySet($sender_data);
    $receiver->template->trySet($receiver_data);

Best Practices
--------------

Don't use SMlite directly
~~~~~~~~~~~~~~~~~~~~~~~~~

It is strongly advised not to use templates directly unless you have no
other choice. Views implement consistent and flexible layer on top of
SMlite as well as integrate with many other components of Agile Toolkit.
The only cases when direct use of SMlite is suggested is if you are not
working with HTML or the output will not be rendered in a regular way
(such as RSS feed generation or TMail)

Organize templates into directories
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Typically templates directory will have subdirectories: "page", "view",
"form" etc. Your custom template for one of the pages should be inside
"page" directory, such as page/contact.html. If you are willing to have
a generic layout which you will use by multiple pages, then instead of
putting it into "page" directory, call it "page\_two\_columns.html".

You can find similar structure inside atk4/templates/shared or in some
other projects developed using Agile Toolkit.

Naming of tags
~~~~~~~~~~~~~~

Tags use two type of naming - CamelCase and underscore\_lowercase. Tags
are case sensitive. The larger regions which are typically used for
cloning or by adding new objects into it are named with CamelCase.
Examples would be: "Menu", "Content" and "Recipient". The lowercase and
underscore is used for short variables which would be inserted into
template directly such as "name" or "zip".

Don't Repeat Yourself (DRY)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You must always remember and your designer must also know the DRY
principle. Avoid having exactly same piece of code on all the pages. If
you must place "disclaimer" on multiple pages, you can use this simple
syntax:

::

    $page->add('View',null,'Disclaimer',array('disclaimer'));

Take advantage of global setTags
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Application (API) has a function ``setTags($t)`` which is called for
every view in the system. It's used to resolve "template" and "page"
tags, however you can add more interesting things here. For example if
you miss ability to include other templates from Smarty, you can
implement custom handling for ``<?include?>`` tag here.

Be considered that there are a lot of objects in Agile Toolkit and do
not put any slow code in this function.





When template is loaded, it's represented in the memory as an array.
Example Template:

::

    Hello <?subject?>world<?/?>!!

SMLite converts the template into the following structure available
under ``$smlite->template``.

Content of tags are parsed recursively and will contain further arrays.
In addition to the template tree, tags are indexed and stored inside
"tags" property.

::

    // template property:
    array (
      0 => 'Hello ',
      'subject#1' => array (
        0 => 'world',
      ),
      1 => '!!',
    )

    // tags property
    array (
      'subject'=> array( &array ),
      'subject#1'=> array( &array )
    )

As a result each tag will actually add two tags. If tag with same name
is added, reference to a region is added inside respective tag
sub-array. This allow ``$smlite->get()`` to quickly retrieve contents of
appropriate tag and it will also allow ``render()`` to reconstruct the
output



 * ==[ About SMlite ]==========================================================
 * This class is a lightweight template engine. It's based around operating with
 * chunks of HTML code and the main aims are:
 *
 *  - completely remove any code from templates
 *  - speed up template parsing and manipulation speed
 *
 * @author      Romans <romans@agiletoolkit.org>
 * @copyright   AGPL
 * @version     2.0
 *
 *
 * ==[ Version History ]=======================================================
 * 1.0          First public version (released with AModules3 alpha)
 * 1.1          Added support for "_top" tag
 *              Removed support for permanent tags
 *              Much more comments and other fixes
 * 2.0          Reimplemented template parsing, now doing it with regexps
 *
 * ==[ Description ]===========================================================
 * SMlite templates are HTML pages containing tags to mark certain regions.
 * <html><head>
 *   <title>MySite.com - {page_name}unknown page{/page_name}</title>
 * </head>
 *
 * Inside your application regions may be manipulated in a few ways:
 *
 *  - you can replace region with other content. Using this you can replace
 *   name of sub-page or put a date on your template.
 *
 *  - you can clone whole template or part of it. This is useful if you are
 *   working with objects
 *
 *  - you can manipulate with regions from different files.
 *
 * Traditional recipe to work with lists in our templates are:
 *
 *  1. clone template of generic line
 *  2. delete content of the list
 *  3. inside loop
 *   3a. insert values into cloned template
 *   3b. render cloned template
 *   3c. insert rendered HTML into list template
 *  4. render list template
 *
 * Inside the code I use terms 'region' and 'spot'. They refer to the same thing,
 * but I use 'spot' to refer to a location inside template (such as {$date}),
 * however I use 'region' when I am refering to a chunk of HTML code or sub-template.
 * Sometimes I also use term 'tag' which is like a pointer to region or spot.
 *
 * When template is loaded it's parsed and converted into array. It's possible to
 * cache parsed template serialized inside array.
 *
 * Tag name looks like this:
 *
 *  "misc/listings:student_list"
 *
 * Which means to seek tag {student_list} inside misc/listings.html
 *
 * You may have same tag several times inside template. For example you can
 * use tag {$title} inside <head><title> and <h1>.
 *
 * If you would set('title','My Title'); it will insert that value in
 * all those regions.
 *
 * ==[ Agile Toolkit integration ]============================================
 * Rule of thumb in object oriented programming is data / code separation. In
 * our case HTML is data and our PHP files are code. SMlite helps to completely
 * cut out the code from templates (smarty promotes idea about integrating
 * logic inside templates and I decided not to use it for that reason)
 *
* Inside Agile Toolkit, each object have it's own template or may have even several
* templates. When object is created, it's assigned to region inside template.
* Later object operates with assigned template.
*
* Each object is also assigned to a spot on their parent's template. When
* object is rendered, it's HTML is inserted into parent's template.
*
* ==[ Non-AModules3 integration ]=============================================
* SMlite have no strict bindings or requirements for AModules3. You are free
* to use it inside any other library as long as you follow license agreements..

