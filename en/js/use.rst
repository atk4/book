**********
Using on()
**********

So far we only looked into :php:meth:`AbstractView::js`. There is another
method, which might be easier to use::

    $view->on('click')->action();
    $view->on('click', $view2->js()->action());

    $view->on('click', 'selector')->action();
    $view->on('click', 'selector', $view->js()->action());

Do not use chain returned by on() in other chains.

Using on() with selector
------------------------

Example::

    $form = $this->add('Form');
    $form->addField('john');
    $form->addField('smith');

    $form->on('focus', 'input')->css([ 'background' => 'red']);
    $form->on('blur',  'input')->css([ 'background' => '']);

Using with PHP callback
-----------------------

On also accepts callable argument, so you can actually pass a PHP call-back::

    $form->on('focus', 'input', function($js) {
        $js->val(rand(1,100));
    });

Above example will fill fields with random numbers when you click in them. The
callback method receives 2 arguments. First ($js) is the chain which selects
affected element, in our case - input field.

Receiving JS data in PHP callback
---------------------------------

The other argument is data() of that element. jQuery implements $.data()
for associtaing JavaScript properties with the field.

Data is automatically passed and you can set it back using data method.
Here is a new implementation, which increases value of the field and stores
it in data()::

    $form->on('focus', 'input', function($js, $data) {
        $c=$data['cnt']+1;
        $js->val($c)->data('cnt', $c);
    });

The functionality of on() PHP callback is implemented using a VirtualPage.

**************
View Reloading
**************

Each page of Agile Toolkit can use :php:method:`App_Web::url` to determine
it's URL location. JavaScript does not have knowledge of a current page.

A method :php:meth:`jQuery_Chain::reload` is a PHP-based method which produces
a JavaScript code for reloading a view::

    $grid->addButton('Reload')->js('click', $grid->js()->reload());

Although this seems like a regular JS Chain, reload() will actually pass an
URL of current page (including :ref:`Sticky GET`) and cut options (See :ref:`cutting`)
for object reloading.

Two-page reload.
================

Sometimes the original object is not available to cretae a reload(). Assuming
you have two pages::

    // Page1
    $grid = $this->add('Grid');

    $this->add('Button')->js('click')->univ()->dialogURL($this->app->url('page2'));


    // Page2
    $b2 = $this->add('Button')->set('Reload Grid');

Although two pages are displayed on the same physical page (second page is in dialog),
the $b2 object cannot reference $grid. This can be fixed like this:

1. Add a custom event for grid reloading.
2. Add some class on a grid for easier selecting
3. On page 2 use selector and trigger your custom event.

Resulting code::

    // Page1
    $grid = $this->add('Grid');
    $grid->addClass('do-reload');
    $grid->js('reload')->reload();

    $this->add('Button')->js('click')->univ()->dialogURL($this->app->url('page2'));


    // Page2
    $b2 = $this->add('Button')->set('Reload Grid');
    $b2->js('click', $this->js()->_selector('.do-reload')->trigger('reload'));


