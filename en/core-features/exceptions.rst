Exceptions Details
==================

Exception basics and introduction, see :php:meth:`AbstractObject::exception`.

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


Bundled Exception Classes
-------------------------

.. php:class:: BaseException

    A standard agile-toolkit friendly exception

.. php:class:: Exception_Logic

    Exception showing some problem in business logic

.. php:class:: Exception_ForUser

    Excetpion which must be shown to user nicely.

.. php:class:: Exception_Obsolete

    Signaling that some feature is obsolete.

.. php:class:: PathFinder_Exception

    rename into Exception_PathFinder

.. php:class:: SQLException

    is this still used?

.. php:class:: ObsoleteException

    isn't this obsolete class obsolete?



Exception Handling
------------------

Uncaught exceptions bubble up to the application class, are caught and
passed on to :php:class:`App_CLI::caughtException`.

In most cases exceptions will end up in :php:class:`Logger` or a similar
error-reporting controller.





