**************************************
Using Forms with AJAXEC for Validation
**************************************

In this tutorial I will explain how you can create a immidiate
on-change validation in your from along with some clever visual
feedback. The resulting form example will look like this:

.. image:: /figures/excercise-form-ajaxec/final-look.png


Set up a basic form
===================

I am going to start with a simple form with two fields - email and name. I
will be doing AJAX-based validation for the email field. Normally you could
use this for verifying if specified email was already used by some user,
when registering, but in my case, I'll just use a very basic validation. Place
the following code in your page class::

    function init() {
        parent::init();

        $form=$this->add('Form');
        $f_email = $form->addField('email')->validateNotNull();

        $f_name = $form->addField('name')->validateNotNull();

        $form->addSubmit('Submit');
    }

Make sure the form works. Next I'd like to place a button to the right from
the email field. The button will have no label but will only use icon
to indicate the verification status::

    $f_email = $form->addField('email')->validateNotNull();
    $em_verify = $f_email
        ->afterField()
        ->add('Button')
        ->set(['', 'icon'=>'ellipsis'])
        ->addClass('do-check');

Our UI is now complete, let's get started with the action. The verification
will happen through a JavaScript chain and it will occur when I will
change the email field or if I manually click on the button.

To produce efficient code I'm going to create a :ref:`javascript chain <JavaScript
Chain>` and bind it
to both events::

    $vp = $this->add('VirtualPage');

    $js_check=$this->js()->univ()->ajaxec($vp->getURL(), ['val'=>$f_email->js()->val()]);

    $f_email->js('change', $js_check);
    $em_verify->js('click', $js_check);

I had also to create a :php:class:`VirtualPage` which I will use as a end-point
for AJAX callback execution.

For the chain I'm using :ref:`ajaxec` method to execute a call to the URL of
a virtual page and pass the value of email field.

Afterwards I am binding chain to two events. Next I am going to define the
PHP code to be executed during callback by using :php:meth:`VirtualPage::set`::

    $vp->set(function($p)use($form){
        if(strlen($_POST['val'])<5){
            $js=[
                // invalid
                $form->js()->find('.do-check span')->attr('class','icon-cancel'),
                $form->js()->find('.do-check')->children()->removeClass('atk-effect-success')->addClass('atk-effect-danger')
            ];
        }else{
            $js=[
                // valid
                $form->js()->find('.do-check span')->attr('class','icon-check'),
                $form->js()->find('.do-check')->children()->removeClass('atk-effect-danger')->addClass('atk-effect-success')
            ];
        }
        $form->js(null,$js)->execute();
    });

The ajaxec() call will pass supplied arguments as a POST data, so I'll be
able to get the value of the email field through ``$_POST['val']``. If the
value is shorter that 5 characters I'll consider it to be invalid. I'm taking
advantage of the fact that I've set the ``do-check`` class on the button
to find it and apply necessary changes.

As a further example, plase try the following:

- Make callback access the button withotu use of do-check class. This will
  make it possible to have multiple fields verify their values.
- Wrap the functionality into a controller and abstract everything away. Make
  sure the usage syntax is simple, like this::

        $form->addField('email')->add('Controller_FieldValidator')
            ->check(function($v){ return strlen($v)>=5; })
