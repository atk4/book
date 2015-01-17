*************
App_Installer
*************

TODO:

- Link this document with class
- Write more documentation on how to "distribute" your application through .agiletoolkit.org


This application class helps you build web installers. It builds on top of
App_Web and introduces multi-step concept that will help guide your user
through necessary actions in installing your full web application.

As you extend App_Installer and define multiple functions started with ``step_``
you will automatically create path for your installer. Here is a very simple
installer example::

    class MyInstaller extends App_Installer {

        function step_welcome($p)
            $p->add('P')->set('Welcome to this installation. Please press Continue');
            $p->add('Button')->link($this->stepURL('next'))->setLabel('Continue');
        }

        function step_welcome($p)
            $p->add('H3')->set('Installaton is complete.');
            $p->add('Button')->link($this->stepURL('first'))->setLabel('Restart');
        }

    }

Navigating between Steps
========================

The only thing here you need to remember is that links between different steps
needs to be created by calling $app->stepURL() instead of $app->url(). As
an argument you can pass one of the following arguments: 'first', 'last',
'current', 'next' and 'prev';

If you need to link to specific step, use::

    return $this->url(null,array('step'=>$s));

Intro
=====

The property $app->show_intro set to "false" by default will make installer
redirect user to the first step automatically. If it's not set, then it will
call showIntro($p) instead that will create some basic welcome screen for
your user.
