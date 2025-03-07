---
title: Usage instructions
product: vobject
layout: default
---

Getting started
---------------

Make sure you followed all the steps in the [installation instructions][1],
and you've included the autoloader in your code.


A basic parsing example
-----------------------

Assuming there is a vCard in your local directory, named `cowboyhenk.vcf`,
the following complete example will parse it and display the `FN` property:

    use Sabre\VObject;

    include 'vendor/autoload.php';

    $vcard = VObject\Reader::read(
        fopen('cowboyhenk.vcf','r')
    );
    echo $vcard->FN;

That is all.

For all the following examples, we are assuming that you have already included
the autoloader, and also that you've called

    use Sabre\VObject;

at the top of your script. This is just to avoid repetition.


The actual manual
-----------------

### Creating vCards

To create a vCard, you can simply instantiate the `VCard` component, and pass
the properties you need:

    $vcard = new VObject\Component\VCard([
        'FN'  => 'Cowboy Henk',
        'TEL' => '+1 555 34567 455',
        'N'   => ['Henk', 'Cowboy', '', 'Dr.', 'MD'],
    ]);

    echo $vcard->serialize();

This will output:

    BEGIN:VCARD
    VERSION:3.0
    PRODID:-//Sabre//Sabre VObject 4.0.0-beta1//EN
    FN:Cowboy Henk
    TEL:+1 555 34567 455
    N:Henk;Cowboy;;Dr.;MD
    END:VCARD


### Manipulating properties

The vCard also allows object-access to manipulate properties:

    // Overwrites or sets a property:
    $vcard->FN = 'Doctor McNinja';

    // Removes a property
    unset($vcard->FN);

    // Checks for existence of a property:

    isset($vcard->FN);

Certain properties, such as `TEL`, `ADR` or `EMAIL` may appear more than once.
To add any additional properties, use the `add()` method on the vCard.

    $vcard->add('TEL', '+1 555 34567 456', ['type' => 'fax']);

The third argument of the `add()` method allows you to specify a list of
parameters.


### Working with parameters

To get access to a parameter, you can simply use array-access:

    $type = $vcard->TEL['TYPE'];
    echo (string)$type;

Parameters can also appear multiple times. To get to their values, just loop
through them:

    if ($param = $vcard->TEL['TYPE']) {
        foreach($param as $value) {
          echo $value, "\n";
        }
    }

To change parameters for properties, you can use array-access syntax:

    $vcard->TEL['TYPE'] = ['WORK','FAX'];

Or when you're working with singular parameters:

    $vcard->TEL['PREF'] = 1;

It is also possible add a list of parameters while creating the property.

    $vcard->add(
        'EMAIL',
        'foo@example.org',
        [
            'type' => ['home', 'work'],
            'pref' => 1,
        ]
    );

### Parsing vCard

To parse a vCard or iCalendar object, simply call:

    // $data must be either a string, or a stream.
    $vcard = VObject\Reader::read($data);

When you're working with data generated by broken software (such as the
latest Microsoft Outlook), you can pass a 'forgiving' option that will do an
attempt to mend the broken data.

    $vcard = VObject\Reader::read($data, VObject\Reader::OPTION_FORGIVING);

Many vCards these days are encoded as UTF-8. If however you are running into a
vCard with a different encoding, you can specify this as the third option:

    $vcard = VObject\Reader::read($data, 0, 'ISO-8859-1');

Currently ISO-8859-1 and Windows-1252 are supported. This feature appeared in
vObject 4.

### Reading property values

For properties that are stored as a string, you can simply call:

    echo (string)$vcard->FN;

For properties that contain more than 1 part, such as `ADR`, `N` or `ORG` you
can call `getParts()`.

    print_r(
        $vcard->ORG->getParts()
    );

### Looping through properties.

Properties such as `ADR`, `EMAIL` and `TEL` may appear more than once in a
vCard. To loop through them, you can simply throw them in a `foreach()`
statement:

    foreach($vcard->TEL as $tel) {
        echo 'Phone number: ', $tel, "\n";
    }

    foreach($vcard->ADR as $adr) {
        print_r($adr->getParts());
    }


### vCard property grouping

It's allowed in vCards to group multiple properties together with an arbitrary
string.

Apple clients use this feature to assign custom labels to things like phone
numbers and email addresses. Below is an example:

    BEGIN:VCARD
    VERSION:3.0
    groupname.TEL:+1 555 12342567
    groupname.X-ABLABEL:UK number
    END:VCARD

In our example, you can see that the TEL properties are prefixed. These are 'groups' and
allow you to group multiple related properties together.

In most situations these group names are ignored, so when you execute the following
example, the `TEL` properties are still traversed.

    foreach($vcard->TEL as $tel) {
        echo $tel, "\n";
    }

But if you would like to target a specific group + property, this is possible too:

    echo (string)$vcard->{'groupname.TEL'};

To expand that example a little bit; if you'd like to traverse through all phone
numbers and display their custom labels, you'd do something like this:


    foreach($vcard->TEL as $tel) {

        echo $vcard->{$tel->group . '.X-ABLABEL'}, ': ';
        echo $tel, "\n";

    }

### Getting a property by its TYPE value

Many vCard properties, such as `TEL`, `ADR` and `EMAIL` support a `TYPE` attribute.

    BEGIN:VCARD
    TEL;TYPE=HOME,FAX:+15551234566
    EMAIL;TYPE=HOME;TYPE=WORK:foobar@example.org
    END:VCARD

If you quickly want to get all the properties that have a specific `TYPE`
value, you can do this with:

    $email = $vcard->getByType('EMAIL','WORK');

This function only returns the first property it finds, and will return `null`
if no property with that `TYPE` could be found.


### Converting between different vCard versions

Since sabre/vobject 3.1, there's also a feature to convert between various
vCard versions. Currently it's possible to convert from vCard 2.1, 3.0 and
4.0 and to 3.0 and 4.0. It's not yet possible to convert to vCard 2.1.

To do this, simply call the `convert()` method on the vCard object.

    $input = <<<VCARD
    BEGIN:VCARD
    VERSION:2.1
    FN;CHARSET=UTF-8:Foo
    TEL;PREF;HOME:+1 555 555 555
    END:VCARD
    VCARD;

    $vcard = VObject\Reader::read($input);
    $vcard = $vcard->convert(VObject\Document::VCARD40);

    echo $vcard->serialize();

    // This will output:
    /*
    BEGIN:VCARD
    VERSION:4.0
    FN:Foo
    TEL;PREF=1;TYPE=HOME:+1 555 555 555
    END:VCARD
    */

It's important to note that not every bit of information can be cleanly
converted between versions. So there's a possibility that you loose some
small bits of information going back and forward. For instance, vCard 2.1
is the only vCard version that supports the `AGENT` property, so it's
dropped when going to vCard 3 or higher.


### Validating vCard

When you parse a vCard, the parser grabs all the values and does
basic syntax checking. It does not however, validate every value.

You can ask the parser to validate the entire document by calling the `validate`
function though:

    $result = $vcard->validate();

The returned value is an array of messages and might look like this:

    [
        [
            'level' => 2,
            'message' => '...',
            'node' => ...A VObject component or property...
        ]
    ]

Each item constitutes a problem with the document. Every item contains a level
containing a number between 1 and 3. 3 Means that the document is invalid, 2
means a warning. A warning means it's valid but it could cause interopability
issues, and 1 means that there was a problem earlier, but the problem was
automatically repaired.

The message is a human-readable string with more information about the problem,
and lastly the node refers to the actual VObject Component or Property object
that had the issue.

You can also pass several options. The options have to be passed as a bitfield:

    $vcalendar->validate(Sabre\VObject\Node::REPAIR);
    $vcalendar->validate(Sabre\VObject\Node::PROFILE_CARDDAV);

The `REPAIR` option automatically repeairs the object, if possible. Without
`REPAIR` the validator does not change the object.

The `PROFILE_CARDDAV` validates the object specifically for within the use of
CardDAV. vCards CardDAV servers have a few additional special
restrictions (must have a `UID` property for instance).

The validator is not perfect, and we improve it when we come across new issues,
so any suggestions are welcome.


Support
-------

Head over to the [SabreDAV mailing list](https://groups.google.com/group/sabredav-discuss) for any questions.

[1]: /vobject/install

