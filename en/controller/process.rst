********************
Process I/O Handling
********************

Agile Toolkit provides a basic object-oriented access to executing command-line
processes asyncronously.

.. note:: This class is designed to work on UNIX operating systems and may
    not work on Windows.

.. php:class:: System_ProcessIO

    This class provides a much more flexible and efficient interface
    to process creation and execution. It's similar to how popen
    works but allows handling multiple streams and supports
    exceptions

Basics
======

.. php:method:: exec

.. php:method:: close

.. php:method:: terminate

ProcessIO basic usage example::

    $p=$this->add('System_ProcessIO')
        ->exec('/usr/bin/perl', ['-e', 'Hello|world\n'] );

    $output = $p->close();

In this example, a new process will be executed and associated with project
$p. Latter the project execution is closed and all the output of the process
is passed into a variable $output.

The process will actually be running between it's initialization and until
you close it.

Writing input to process
========================

.. php:method:: write

.. php:method:: writeAll

Example::

    $output = $this->add('System_ProcessIO')
        ->exec('/usr/bin/sed','s/l/r/g')
        ->write_all('Hello world')
        ->close();

    // output contains: Herro worrd

Reading output
==============

.. php:method:: read

.. php:method:: readAll

Example for line-by-line reading and writing::

    $p=$this->add('System_ProcessIO')
        ->exec('/usr/bin/sed','s/l/r/g');

    $p->write('coca cola');
    $out=$p->read_line();

    // coca cora

    $p->write('little love');
    $out=$p->read_line();

    // rittre rove

    $this->terminate();

Error Handling
==============

.. php:class:: Exception_ProcessIO

Example error::

    try {
        $p=$this->add('System_ProcessIO')
            ->exec('/usr/non/existant/path','hello world')
            ->write_all('Hello world');
        $out=$p->close();
    }catch(System_ProcessIO_Exception $e){
        $error=$e->getMessage();

        // will contain error about non-existang executable
    }

