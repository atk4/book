*********
Agile CSS
*********

Agile CSS is a flexible and extensible CSS framework for simple and consistent
Web UI development.

Design philosophy and features
==============================

When a major design company creates a high-end product, design or interface,
they invest immense amount of theory and knowledge to make a harmonic and
consistent design.

The rest of us are working on a limited budgets or maybe even lack knowledge
of concepts such as "grid system", "golden ratio", "baseline", "color palettes".

We decided to create AgileToolkit CSS to make it super easy for designers and
developers to create a great looking websites with minimum effort and knowledge.


Components
==========

AgileToolkit CSS uses a concept of Widgets and Components. Below is a widget
example. Each class starting with "atk-" is what we call a component (atk-box,
atk-jackscrew, etc)::

    <div class="atk-cells atk-cells-gutter-large atk-box atk-swatch-yellow">
        <div class="atk-cell">
            <img src="playground/temp/tomatoes.jpg" alt="Tomatoes">
        </div>
        <div class="atk-cell atk-valign-middle atk-jackscrew">
            <h4>Tomatoes</h4>
            <p>Donec dolor sapien, sollicitudin in erat ac.</p>
            <hr class="small">
            <a href="#"><span class="icon-down-circled-1"></span>&nbsp;Download</a>
        </div>
    </div>

.. note::
     
     Class-names could be simplified a lot using the syntax from semantic-ui.com
     example above would look like this without loosing any features, only the
     css file would need to adopt to the new syntax: 
     
     <div class="ui cells gutter large box swatch yellow">
         <div class="ui cell valign middle jackscrew">
       
     and in css file you limit the selectors with 
     .ui.cells and make it unique, so .cells can be used independently
     read documentation of semantic-ui.com to understand how it works. 

Using Extra CSS
===============

Most Agile Toolkit projects can be implemented without need to create a custom
CSS. That's the beauty of CSS framework - the majority of User Interface
can be developed without any changes in CSS - as long as you fully understand
Agile CSS framework.


In some cases you do want to add a custom CSS. One option would be to change
the :ref:`boilerplate_html` (template/html.jade and template/html.html),
however if you simply looking to add an extra CSS file to Agile Toolkit
add this into your application class::

    $this->addStylesheet('my');

Or you can specify a full URL::

    $this->addStylesheet('http://cdn.example.com/my');

Settings
========

LESS setting description.



.. toctree::
    :maxdepth: 1

    css/grid
    css/responsive
    css/typography
    css/helpers
    css/components
    css/menu
    css/actions
    css/layouts
    css/forms
    css/widgets
    css/tables
    css/navigation
    css/popovers
    css/banners
    css/jquery


