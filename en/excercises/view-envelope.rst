
Rendering Behaviours - Envelope, Sender, Recipient
==================================================

This excercise demonstrates how the basic Views work with their templates. For
more information see :doc:`/views/core`.

Consider the following template on the parent object located in file
:file:`envelope.html` (I'm omitting HTML markup to make this easy to read)::

    <pre>
    Sender info {Sender}
        Name: {$name}
        Surname: {$surname}
    {/Sender}

    Recipient info {Recipient}
        No Recipient
    {/}
    </pre>

This template defines a region ``Sender`` containing tags for name and surname.
The Recipient is an optional property in this template, it may be present, but
if not specified, it would contain No Recipient. When Recipient is present, the
markup for that field would be same as Sender.

.. figure:: /figures/view-envolpe-figure.png


I'll start by defining my own custom class 'View_Envelope'. It's advised that
you prefix all your View classes with 'View_'::

    class View_Envelope extends AbstractView {
        function defaultTemplate() {
            return ['envelope'];
        }
    }

This is a good start. Let's add the envelope to your page, and see how it would
appear::

    Sender info
        Name:
        Surname:

    Recipient info
        No Recipient

Next, we will add a method setSender($properties) to our view class::

    public $sender = null;
    function setSender($properties) {
        $this->sender = $this->add('View', null, 'Sender', 'Sender');
        $this->sender->template->set($properties);
        return $this;
    }

When setting sender, you will see that it inherits parent's template.
I will also update the code which was placed on the page::

    $this->add('View_Envelope')
        ->setSender( [ 'name'=>'John', 'surname'=>'Smith' ])
        ;

Next I will add a new method which will populate the template for Receiver
region. Sometimes we might not need the recipient at all, so unless
a method `setRecipient` is called the "No Recipient" label will appear
in the output.

Unlike senders object we can't use Region closing for template definition.
After template is cloned, child object will remove contents of the tag and
it remains empty until childn't render method will re-populate it with
resulting output.

You can have multiple Views outputing into the same spot, but just can't
use it for cloning. To work around this, I'm going to specify template file
explicitly for the recipient's object::

    public $receiver = null;
    function setReceiver($properties) {

        // Reuse same template, but clone region.
        $template = $this->defaultTemplate();
        $template[] = 'Sender';

        $this->receiver = $this->add('View', null, 'Receiver', $template);
        $this->receiver->template->set($properties);
        return $this;
    }

This method re-uses the default template of the envelope, however will clone
the Sender's region for the object. The output will be placed inside "Receiver"
spot. The following figure illustrates how the templates are loaded, cloned
and rendered:

.. figure:: /figures/view-envolpe-figure-2.png



