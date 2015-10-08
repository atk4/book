Validation
----------

Agile Toolkit provides a unified way to perform validation. Validation
can be performed in 3 ways:

-  validateField() evail
-  validate hook
-  onSubmit() validation

Additionally you can use jQuery widget for client-side validation or
filtering.

## Server-side Validation
-------------------------

Client-side validation is not reliable. User can turn off javascript or
modify the page and still bypass validation rules. Therefore if
validation is done in the browser, it must also be performed on the
server too. Problem here is that browser executes JavaScript and server
executes PHP.

To produce two set of validaiton codes some framework define a number of
validatiors which are capable of generating both — server and client
side code and perform identical validation. Agile Toolkit follows
different approach.

Agile Toolkit always performs validation on the server. However submit
form data is sent through AJAX without reloading page. If any field
contins error, then it will be highlighted without reloading page and
potentially loosing some of the data. This approach combines benefits of
both server side validation and JavaScript field error highlighting.

Using validateField()
~~~~~~~~~~~~~~~~~~~~~

If you do not mind passing PHP code to be evaluated, than this is the
easiest way to define form validators. First argument contains teh code
which will be eval()'ed. You can reference field through ``$this`` and
access current value through ``$this->get();``

Checking Inside isSubmitted() Condition
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Method ``isSubmitted()`` loads data from ``$_POST`` into form ``$f``.
Another common place to perform validation is inside the ``if()`` block
which checks for form submission.

Using validate Hook
~~~~~~~~~~~~~~~~~~~

Hooks are internal mechanism of Agile Toolkit to inject code into
object's method. Each form have it's own "validate" hook. You can use
this approach to inject a code to be performed at validation stage.

*This approach semantics might change.*

## Client-side validation
-------------------------

Agile Toolkit leaves it up to developer to build client-side filters if
necessary. A good example is the ``univ().numerifcField()`` field.The
following function exists in univ.js, calling it on the field will
introduce bindings for field validation.

::

    numericField: function(){
        this.jquery.bind('keyup change',function () {
        var t= this.value.replace(/[^0-9\.-]/g,'');
            if(t != this.value)this.value=t;
        });
    }

Form Validation Examples
~~~~~~~~~~~~~~~~~~~~~~~~

example 1

::

    $f=$page->add('Form');

    $f->addField('line','email')
        ->validateNotNull()
        ->validateField(
            'filter_var($this->get(), FILTER_VALIDATE_EMAIL)');

    $f->addSubmit();

example 2

::

    $f=$page->add('Form');

    $f_email=$f->addField('line','email')
        ->validateNotNull()
        ->set('test@example.com');

    $f->addSubmit();
    if($f->isSubmitted()){
        // manually displaying error message
        if($f->get('email')=='test@example.com'){
            return $f->getElement('email')->displayFieldError('Choose other email');
        }
    }

example 3

::

    $f=$page->add('Form');

    $f_email=$f->addField('line','email')
        ->validateNotNull()
        ->set('test@example.com');

    // Adding validation hook through closure
    $f_email->addHook('validate',function() use ($f_email){
        if($f_email->get()=='test@example.com')
            $f_email->displayFieldError('Choose other email');
    });

    $f->addSubmit();

example 4

::

    $f=$page->add('Form');

    // JavaScript-based validation
    $f->addField('line','age')->js(true)
        ->univ()->numericField();

