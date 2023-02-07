---
title: CalDAV Scheduling
layout: default
type: plugin
plugin_name: calendar-auto-schedule
plugin_since: 2.1.0
---

Since version 2.1.0, sabre/dav ships with support for CalDAV scheduling, as
defined by [rfc6638][rfc6638].

This plugin enables several features:

1. Inviting people on a CalDAV server.
2. Automatically delivering invites to a CalDAV inbox.
3. Functionality to Accept or Decline invites.
4. Reading free-busy information from users on the server.

This specification is very widely supported by clients, but even clients that
do not support this specification will trigger scheduling-related behavior.

For example, if a user creates a new event with an `ORGANIZER` that matches
the current user, and at least one `ATTENDEE`, the server will automatically
detect this and send invites to the relevant people.

When a user deletes the same invite, we will automatically send `CANCEL`
notifications to anyone affected.

When an attendee sets their `PARTSTAT` to `ACCEPTED` or `DECLINED`, the server
will also automatically update the organizer's event with this information and
notify him that this has happened.


Enabling scheduling
-------------------

To enable the base system, just add the plugin to your server:

    $server->addPlugin(
        new Sabre\CalDAV\Schedule\Plugin()
    );


Backend support
---------------

The `PDO` CalDAV backend has built-in support for local delivery. If you
created your own CalDAV backend, you must implement the methods from the
`Sabre\CalDAV\Backend\SchedulingSupport` interface to add support.

This interface adds 4 new methods for managing objects in a user's inbox.


Email delivery
--------------

If it's not possible to do a local delivery, or your backend does not support
local delivery yet, it's possible to deliver messages via email. This is
called iMip and defined in [rfc6047][rfc6047].

To add support for iMip, add the following additional plugin:

    $server->addPlugin(
        new Sabre\CalDAV\Schedule\IMipPlugin('server@example.org')
    );

The iMip plugin generates very plain and arguably ugly emails. It also uses
PHP's `mail()` function, which may not work for everyone due to the use of
an STMP server.

We recommend people who need more advanced email functionality, or want to
customize their email templates to subclass the IMipPlugin. It's a solid
example and hopefully provides enough information to customize your emails
further.


Availability
------------

Since sabre/dav 3.1 [availability][avail] is supported. This feature allows
users to specify for example when their working hours are, such as 9am to
5pm on a daily basis.

After specifying this, the server will then use this information to generate
freebusy reports, making them a lot more accurate and useful, especially
with teams distributed in multiple timezones.

This feature is enabled automatically, but it will only work if the
[PropertyStorage][property-storage] plugin is enabled.


Limitations
-----------

We don't support the full specification yet. Non-exhaustive list of known
issues:

1. We don't do VTODO scheduling yet, and only support VEVENT.
2. We don't have support for `Schedule-Tag` yet.

If you really need either of those features, let us know through
[the mailinglist][mailinglist] or [github issues][issues]. If we don't know,
we don't know if we need to prioritize it.


[rfc6638]: https://tools.ietf.org/html/rfc6638
[rfc6047]: https://tools.ietf.org/html/rfc6047
[avail]: https://tools.ietf.org/html/draft-daboo-calendar-availability-05
[mailinglist]: https://groups.google.com/group/sabredav-discuss
[issues]: https://github.com/sabre-io/dav/issues
[property-storage]: /dav/property-storage/
