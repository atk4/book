
.. _univ:

******************
Universal JS Chain
******************

Often developers behave very neatly on the server - all PHP functions arranged
into classes, no global variables; but when it comes to JavaScript functions
are defined in a very messy way, they are scattered across several files and
are defined in global content.

Agile Toolkit relies on jQuery library but it does want number of it's own
functions to be accessible. To implement this, $.univ() function is defined
which returns a copy of Universal chain object.

Universal object defines quite a few functions which you can use in your
application. For example, executing this in JavaScript console (inspector or
firebug) will show a success message on your current page::

    $.univ().alert('Hello World');

Methods defined inside univ() chain by default return reference to the same
chain, so that you can perform multiple actions::

    $.univ().alert('hello').dialogOK('agile','world');

Try some more examples::

    $.univ().dialogURL('test','/learn');

    $.univ().confirm('are you sure').alert('you are');

To see other functions, which are already defined in univ.js chain, see it's source.

Using univ() chain from PHP
===========================

It's super simple to use univ() chain from the PHP code. Here is example
how to bind alert event to a button::

    $this->addButton()->js('click')->univ()->alert('Clicked');

Extending univ() chain with your own Methods
============================================

Create a new file ``my-univ.js`` and place it inside ``public/js`` folder::

    $.each({

      myfunc1: function(a){
        alert(a);
      },

      myfunc2: function(b){
        $(b).fadeOut();
      }

    },$.univ._import);

Next we would need to include your chain::

    $this->js()->_load('my-univ');

You can decide where you want to place this code. If you place it inside
application class, the methods will be available throughout your application.
If you place it inside a view, then the methods only will be available
for that view.

If some of your views uses a heavy JS functionality, you should only include
that JS form that particular view. This will prevent your web application
from loading the JS code when it's not needed.

Let's now use your methods::

    $b1 = $this->add('Button')->set('Trigger Alert');

    $b2 = $this->add('Button')->set('Hide button #1');

    $b1->js('click')->univ()->myfunc1();

    $b2->js('click')->univ()->myfunc2($b1);

Univ chain methods can automatically be chained (but only if your
method does not return any value)::

    $this->add('Button')->set('Do Both')
        ->js('click')->univ()
            ->confirm('Do both actions?')
            ->myfunc1()
            ->myfunc2($b1);


Basic methods from univ chain (atk4_univ_basic.js)
==================================================

A standard univ chain already contains a lot of useful methods you can use.

.. _univ_alert:

alert()
-------

Displays alert

.. _univ_setTimeout:

setTimeout(callback, delay, [ arg, .. ])
----------------------------------------

Same as window.setTimeout(). Will execute callback and pass
arguments to it after ``delay`` ms. Returns id.

.. _univ_setInterval:

setInterval(callback, delay, [ arg, .. ])
-----------------------------------------

Same as window.setInterval(). Will execute callback and pass arguments
to it every``delay`` ms. Returns id.

.. _univ_clearTimeout:

clearTimeout(id)
----------------

Stops timeout

.. _univ_clearInterval:

clearInterval(id)
-----------------

Stops interval. See window.clearInterval


.. _univ_redirect:

redirect(url)
-------------

Redirects browser to a specified URL. If your application is an ajaxified,
then it will attempt to dynamically load the page.

.. todo:: write article about ajaxification


location(url)
-------------

Redirects browser to a specified URL. URL can be passed as :ref:`url component array`

.. _univ_page:

page(url)
---------

Dynamially loads a page (through AJAX without refreshing your browser)


confirm(msg)
------------

Will display a confirmation to user and if he clicks OK, proceed with the rest
of univ chain.



closeExpander
-------------

If called on any element inside Grid / expander, it will collapse expander.

getJQuery
---------

Normally univ() methods will return univ() itself, but aclling this method
will return jQuery object with the current element still selected.


.. _ajaxec:

ajaxec(url, data, fn)
---------------------

Will send AJAX request for the specified URL. Response will be avaluated
as JavaScript code.

The page which handles the response should use :php:method:`jQuery_Chain::execute`::

    $b=$this->add('Button')->set('Randomise');
    $b->js('click')->univ()->ajaxec(
        $this->api->url(null, ['randomise'=>true])
    );

    if($_GET['randomize']) {
        $b->js()->text('Rand: '.rand(1,100))->execute();
    }

If second argument - data is specified as array, it's passed through POST data.
You can also use :ref:`url definition array` to pass GET data::

    $b->js('click')->univ()->ajaxec(
        [ $this->api->url(null, ['randomise'=>true]), 'foo'=>'bar' ],
        [ 'foo' => 'baz' ]
    );

If you specify ``true`` as second argument, then ``data()``
(http://api.jquery.com/data/#data) of the object will be automatically passed.


Third argument will be called right after ``ajaxec`` finises successfully
loading it's work::

    $b->js('click')->univ()->ajaxec(
        [ $this->api->url(null, ['randomise'=>true]), 'foo'=>'bar' ],
        [ 'foo' => 'baz' ],

        $b->js()->univ()->alert('Done')->_enclose()
    );


.. _autoChange:

autoChange(ms)
--------------

Normal behaviour of JS is to trigger ``change`` event when input field looses
focus. Quite often you would want this to happen sooner:

- if you performing JS calculation on the fly immediatelly.
- if you need slight delay on your quick-search field.


calling autoChange will trigger ``change`` event faster. Use this on form field::

    $f_name = $form -> addField('name');
    $f_surname = $form -> addField('surname');

    $f_full = $form -> addField('full_name')->setAttr('disabled');

    $js_concat = $f_full->js()->val(
        $f_name->js()->val()->concat(" ", $f_surname->js()->val())
    );

    $f_name->js('change', $js_concat);
    $f_surname->js('change', $js_concat);

    $f_name->js(true)->univ()->autoChange();
    $f_surname->js(true)->univ()->autoChange();

numericField
------------

Only allows numbers to be entered in the field::

    $form->addField('phone')->js(true)->univ()->numericField();


disableEnter
------------

Will ignore if user presses Enter in this field::

    $form->addField('phone')->js(true)->univ()->disableEnter();


.. _bindconditionalshow:

bindConditionalShow(conditons, tag)
-------------------

Will show / hide fields based on other field current values.

Conditios are described as array. Conditions are checked against current
field and various other fields may appear or be hidden depending on it's value.
Next example will show second address line only if the first one is not
empty::


    $f_ad_line1 = $form->addField('address_line1');
    $f_ad_line2 = $form->addField('address_line2');

    $f_ad_line1 -> js(true)->univ()->bindConditionalShow( [
        '' => [],
        '*' => ['address_line2']
    ]);

Inside conditions you specify field value as a key and array of fields
to be visible during this value on the right. Value ``*`` represents all
unspecified values. All the fields ever mentioned in conditions will be
hidden if they are not explicitly specified.

Conditions work with input fields, radio buttons, checkbuttons, dropdowns
and other field types::

    $interests=['S'=>'swimming','R'=>'running','W'=>'watching birds'];

    $form->addField('dropdown','interest','Your interests')
        ->setValueList($interests)
        ->js(true)->univ()->bindConditionalShow( [
            'S'=> ['swimming_info'],
            'R'=> ['running_info'],
            'W'=> ['birds_which','birds_where']
        ]);

    $form->addField('line','swimming_info','How far can you swim?');
    $form->addField('line','running_info','How far can you run?');
    $form->addField('line','birds_which','What type of birds?');
    $form->addField('line','birds_where','Where do you watch them?');

Do not nest conditions, e.g. do not set conditions on a field, which is
controlled by other conditions. This may result some stray fields remaining
on your form. If you need a more complex logic, create your own javascript
method.

This method only implements JavaScript behaviour. It will not affect form
validation.



Other Methods
-------------

There are other methods in file ``atk4_univ_basic.js`` chain. You can use them
at your own risk.


jQuery UI related methods from univ chain (atk4_univ_ui.js)
===========================================================

If your application is using jQuery UI (see :php:class:`jUI`), then a file
``atk4_univ_ui.js`` will be included adding more methods to univ().

dialogOK (title, text, fn, options)
-----------------------------------

Displays jQuery dialog with OK button. fn callback is called when user
closes dialog (closs or using OK button). options are passed to dialog
init method, see jQuery UI docs.

dialogConfirm (title, text, fn, options)
----------------------------------------

Very similar to dialogOK, but displays OK / Cancel options. Callback is
executed on "ok".

frameURL(title, url, options, callback)
---------------------------------------

Opens a new dialog and loads a specified page in there. (See :ref:`cutting`).
If callback is specified, it's called after loading is finished. Here is action
order:

#. dialog opens
#. atk4_loader starts loading url
#. spinner shows
#. HTML from response are placed inside dialog
#. JavaScript events from responsea are executed
#. spinner removed
#. callback is called

.. todo:: verify if this order is true

.. note:: When dialog is closing, it will look for un-saved atk4_form widgets
    in it's body. If any are found, it will display a confirmation.

dialogURL(title, url, options, callback)
----------------------------------------

This is identical to frameURL, but will also have OK / Cancel buttons
by jQuery. Normally you would probably want to have your own buttons inside
frame instead.


successMessage(msg)
-------------------

Displays a Growl-style message notifying user that some action was completed
successfully.


getFrameOpener
--------------

This is a very interesting function which allows you to have connection between
element which opened dialog and the code inside the dialog. Here is example,
which will display 2 identical buttons opening same dialog. The dialog
will change the label of the button which was used to open it::

    $b1 = $this->add('Button')->set('Button1');
    $b2 = $this->add('Button')->set('Button2');

    $vp = $this->add('VirtualPage')->set(function($p){

        $p->add('Button')->js('click')->univ()->closeDialog();

        $p->js(true)->univ()->getFrameOpener()->text('CLICKED');
    });

See also :php:class:`VirtualPage`

closeDialog
-----------

If called on any element inside a dialog, it will find parent dialog and
close it.


CloseDialog will select frameOpener, because dialog and it's elements
will cease to exist. Here is example::

    $b1 = $this->add('Button')->set('Button1');
    $b2 = $this->add('Button')->set('Button2');

    $vp = $this->add('VirtualPage')->set(function($p){
        $p->add('Button')->js('click')->univ()
            ->closeDialog()->getJQuery()->text('CLICKED');
    });


See also :php:class:`VirtualPage`

loadingInProgress
-----------------


This is called when user perform action without waiting for his previosu action
to be completed. By default this displays alert "Loading in progress. Please wait".

You can redefine this method to do something else.

dialogPrepare
-------------

Configures a default dialog. You can override this method to create your own
dialog.
