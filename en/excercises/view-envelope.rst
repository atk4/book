
Rendering Behaviours - Envelope, Sender, Recipient
==================================================

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
