Validation
----------

Agile Toolkit provides a multitude of ways to perform validation.

## Server-side Validation
-------------------------

Client-side validation is not reliable. User can turn off javascript or
modify the page and still bypass validation rules. Therefore if
validation is done in the browser, it must also be performed on the
server too. Problem here is that browser executes JavaScript and server
executes PHP.

To produce two sets of validation codes some framework define a number of
validators which are capable of generating both — server and client
side code and perform identical validation. Agile Toolkit follows
different approach.

Agile Toolkit always performs validation on the server. However submit
form data is sent through AJAX without reloading page. If any field
contins error, then it will be highlighted without reloading page and
potentially loosing some of the data. This approach combines benefits of
both server side validation and JavaScript field error highlighting.

Using callback with $field->validateField() method
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you do not mind passing PHP code to be evaluated, than this is the
easiest way to define form validators. You can pass a callback method 
to the validateField() method in order to validate the field value and 
return a string if validation fail. The returned string will
be display as an error in your form display.

Example using an anonymous function:

::
	$form = $this->add('Form');
	$form->addField('line', 'name')
		  ->validateField( function( $field )
			{ 
				if( !$field->get() )
					return 'Name is mandatory'; 
			});
		   		
Same can be achieve using a class method.  

:: 
	$form->addField('line', 'name')->validateField( array( $this, 'validateName' );
					
					
Defining the class method:

:: 
	function validateName( $field )
	{
		if( !$field->get() )
					return 'Name is mandatory'; 
	}     


Using the $field->validateNotNull() method
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As the name state, this method will check if a field value as been set. 
It also accept one string parameter for error display.

::
	$form = $this->add('Form');
		
	//Check if field has a value.
	$form->addField('line','name')->validateNotNull( 'Name is mandatory' );


Using Controller_Validator with $field->validate( $rule ) method
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Validation done this way take advantage of the Controller_Validator class which is described in more detail here
	.. _Controller Validator: http://book.agiletoolkit.org/controller/validator.html

Using this controller, it is possible to pass a set of rule to the validate() method that will be use for validating the field value.

::
	$form = $this->add('Form');
	
	//Check if field value is greater than 1000 and display an error msg if not.
	$form->addField('line','large_number')->validate('>1000?is not large enough');
	
	//Check if field has a value.
	$form->addField('line','name')->validate('required');


Using Controller_Validator with $form->validate( $rule ) method
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This also uses Controller_Validator, but calls it through $form->validate()
This method allows you to define multiple rules with a single call and is directly passed to the validator is() method. 
Also when calling through $form->validate(), this also binds validation to the form 'validate' hook.

::
	$form = $this->add('Form');
	$form->addField('line','large_number');
	$form->addField('line', 'name');
	
	//Validating using multiple field|rule at once.
	$form->validate([
		'large_number|>1000?is not large enough',
		'name|required'
	]);
	

Using Controller_Validator with $field->validate( $callback )
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Controller_Validator also supports your own callbacks method via anonymous or class method.
The anonymous or class method will receive the validator object and the field value as method parameters.

::
	$form = $this->add('Form');

	$form->addField('line','large_number')
		->validate( function( $validator, $value ){
			if( $value < 1000 )
				$validator->fail( 'is not large enough' ); 
		});


Using Form 'post-validate' hook
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Finally, you may also use the form 'post-validate' hook via an anonymus or class method callback.
The callback method will receive the form object as a parameter. This hook is fire after form submission
and fields are loaded with data value.

::
	$form = $this->add('Form');
	$form->addField('line','large_number');
	$form->addField('line', 'name');
	
	$form->addHook( 'post-validate', function( $form ) {
		if( !$form['name'] )
				$f->error( 'name', 'Name is mandatory' );
		if( !$form['large_number'] > 1000 )
				$f->error('name','Number is not large enough');
	});


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
