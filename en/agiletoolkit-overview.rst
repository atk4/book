Agile Toolkit Overview
######################

To use Agile Toolkit successfully, you must understand it's concepts.


Everything is an Object
=======================
In our application so far we have defined and created objects, such as:
model, crud, button and error-box. Agile Toolkit only work with objects.

The classes of those objects would inherit from ``AbstractObject``, so
all objects have a common anchestor.

Objects live in a Runtime-Tree
==============================
In Agile Toolkit you don't create instances using a "new" statement, instead
you "add" objects into other objects. It is, in a way, similar, but objects
are by default connected with the rest of your application.

Any object of Agile Toolkit would have a reference to it's owner - the object
which spawned it. It also has a reference to the top-most object - Application
object.

Create a new PHP file called "cmd.php"::

    include'vendor/autoload.php';

    $app = new App_CLI('myapp');

    $model = $app->add('Model', 'mymodel');
    $ctl = $model->add('Controller', 'mycontroller');

    echo $ctl->name."\n";
    // Outputs myapp_mymodel_mycontroller

    echo $model->name."\n";
    // Outputs myapp_mymodel

    echo $ctl->owner->name."\n";
    // Outputs myapp_mymodel

    echo $ctl->app->name."\n";
    // Outputs myapp



.. toctree::
    :maxdepth: 1

    agiletoolkit-overview/design-goals
    agiletoolkit-overview/view
    agiletoolkit-overview/model
    agiletoolkit-overview/controller
    agiletoolkit-overview/application
    agiletoolkit-overview/what-is-cakephp-why-use-it
    agiletoolkit-overview/understanding-model-view-controller
    agiletoolkit-overview/where-to-get-help


.. meta::
    :title lang=en: CakePHP Overview
    :keywords lang=en: web application framework,model view controller,object oriented programming,piece of cake,cookbook,functionality,xml,cakephp
