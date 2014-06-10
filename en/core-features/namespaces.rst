Using Namespaces in your project
================================

Agile Toolkit support namespaces primarily for Add-on management. Here is how you define a class inside a add-on and call it::

    // inside shared/addons/myaddon/Test.php
    namespace myaddon;
    class Test extends \View {    // backslash is not typo. It refers to a top-level namespace
        function init(){
            parent::init();
            $this->set('Hello from Add-on');
        }
    }

    // Using class from add-on
    $this->add('myaddon/Test');

Agile Toolkit also supports Composer namespace declarations, so if you would rather
use composer.json to define your class locations you can safely do that.

By default Agile Toolkit will also attempt to load your class::

    foo\bar\My_Class

from the file "foo/bar/My/Class.php", which would be located inside one of the
``addon`` locations. :php:class:`PathFinder` already defines one location for
you, ``shared/addons`` but you can add more if you want inside your application class::

    $this->pathfinder->base_location->defineContents(array(
        'addons'=>array('3rdparty-addon-pack','atk4-addons'),
    ));

Converting namespace into full-featured add-on
----------------------------------------------

You might feel that some classes in your namespace might be re-usable or even
good enough to sell. We believe that it's your decision to either go open-source
route or offer a premium quality at affordable price.

You would need to submit your addon to Agile Toolkit marketplace, to
allow Agile Toolkit users to purchase and use your add-on easily. You should
get in touch with us if you think your addon could appear as "featured" during
the standard installation proces of Agile Toolkit.

You can find furtherinformation inside :doc:`/addons`.


For open-source add-ons, they must be released under MIT license, so that all
users of Agile Toolkit can use them in both commercial and open-source projects.


