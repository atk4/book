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



