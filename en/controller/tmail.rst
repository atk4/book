****************************
Find Documentation for TMail
****************************

.. php:class:: Tmail

    Template-based mailer for Agile Toolkit

TMail is using :php:class:`GiTemplate` to prepare mail message body
and can send using several transports such as:

- mail() function
- phpmailer
- Amason SES
- echo (simply outputs mail in browser instead of sending)

General Usage
=============

.. php:method:: loadTemplate
.. php:method:: set
.. php:method:: send

Typical usage pattern::

    $tm = $this->add('TMail');

    $tm->loadTemplate('my-mail');

    $tm->set('name', 'John');
    $tm->set('prize', 'Baloon');
    $tm->set(
        'link',
        $this->app->url('prize', ['type'=>'baloon'])
        ->useAbsoluteURL()
    );

    $tm->send($email);

Normally mail managers will want you to specify subject, from address
and message body, but TMail takes all of this from the template located
in folder ``template/mail/my-mail.mail``::

    {subject}Congrats. You won {$prize}{/}

    <h2>Congratulations, {$name}</h2>

    <p>You have just won an amazing {$prize}. To claim the prize you
    must click on the following link</p>

    <a href="{$link}">{$link}</a>

By now you might have noticed some of the benefits TMail gives you:

- Template is located in a separate file. If we are using templates
  for UI elements, why not also use it for mail.
- Template may contain tags, which are populated by a programmer. In
  our case the prize tag appears several times in the template.
- Template also defines mail subject and it may contain tags too.

TMail is also object-friendly. Let me rewrite the same code using
objects::

    function prizeNotification(Model_User $user, Model_Prize $prize) {

        $tm->loadTemplate('my-mail');

        $tm->set($user);    // fill template tags from user
        $tm->set('prize',$prize['name']);   // fill prize name
        $tm->set(
            'link',
            $this->app->url('prize', ['id'=>$prize->id])
            ->useAbsoluteURL()
        );

        $tm->send($user['email']);
    }

The code above will populate all the fields from the ``$user`` model first,
but will set {$prize} properly. It will also use prize_id for building
the URL and will pick user's email from the model.

If the template will be modified to include user's surname or some other
fields those will be automatically populated from the model without
the need of changing the code.

Configuration Parameters
========================

.. todo:: This must be TESTED and fixed in ATK:

TMail will recognize some settings in your configuration file::

    $config['tmail']['form'] = 'John Smith <john@example.com>';
    $config['tmail']['transport'] = 'SES';

    // Most transports would implement following optional settings:
    $config['tmail']['bcc'] = 'copy@example.com'; // bcc on all email
    $config['tmail']['relpy_to'] = '' // same format as from
    $config['tmail']['agent'] = 'agent';

Transport will look for more configuration options. For SES you may have::

    $config['tmail']['AWSAccessKeyId'] = 'access key';
    $config['tmail']['AWSSecretKey'] = 'secure key';
    $config['tmail']['ses_url'] = 'url';

For SMTP/TSL and auth you would need to use PHPMailer
(you must install it through composer)::

    $config['tmail']['pmpmailer'] = [

        'transport'=>'smtp-tsl',   // only smtp-tsl is supported currently
        'username'=>'smtp username',
        'password'=>'smtp password',
        'host'=>'smtp host',

        // optional: 'port'=>'smtp port',
    ];


