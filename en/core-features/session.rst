Using Session
=============

Session management
~~~~~~~~~~~~~~~~~~

All objects in Agile Toolkit come with four methods to access session
data: memorize(), learn(), forget() and recall(). You can specify the
“key”, which will be used in scope of a current object:

::

    $obj1 = $this->add('MyObject');
    $obj2 = $this->add('MyObject');

    $obj1->memorize('test','foo');

    $obj2->recall('test'); //returns null
    $obj1->recall('test'); // returns foo

You can learn more about Agile Toolkit management under the Application
section.

+------------+--------------------------------+-------------------------------------------------------------------------------+
| Method     | Arguments                      | Description                                                                   |
+============+================================+===============================================================================+
| memorize   | name, value                    | Store value in session under name                                             |
+------------+--------------------------------+-------------------------------------------------------------------------------+
| recall     | name, default                  | If value for name was stored previously return it, otherwise return default   |
+------------+--------------------------------+-------------------------------------------------------------------------------+
| forget     | name                           | Remove previously memorized value from session                                |
+------------+--------------------------------+-------------------------------------------------------------------------------+
| learn      | name, value1, value2, value3   | Memorize first non-null argument.                                             |
+------------+--------------------------------+-------------------------------------------------------------------------------+


