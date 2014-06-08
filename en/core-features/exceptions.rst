Exceptions
~~~~~~~~~~

Agile Toolkit takes full advantage of exceptions. You can read more
about exceptions in ...

To raise exception please use the exception() method of an object:

::

    $obj = $this->add('MyObject');
    throw $obj->exception('Something is Wrong');

    // alternatively, add more info to exception
    throw $obj->exception('Something is Wrong')
        ->addMoreInfo('foo',123);

My default objects through exception of a class "BaseException". You may
use your own class:

::

    class Exception_MyTest extends BaseException {
        function init(){
            parent::init();
            $this->addMoreInfo('thrown by',$this->owner->name);
        }
    }
    throw $obj->exception('Something is Wrong','MyTest');

Default exception type can also be specified on per-object basis through
a property:

::

    class Model_Test extends AbstractModel {
        public $default_exception='Exception_MyTest';
    }
    // or
    $this->add('MyObject',array('default_exception'=>
        'Exception_MyTest'));

Third argument to exception() can specify a default exception code which
can further be accessed through "code" property.

