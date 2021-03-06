Troubleshooting
===============

CSP WARN: Directive "..." violated by https://browserid.org/include.js
----------------------------------------------------------------------

This warning appears in the Error Console when your site uses
`Content Security Policy`_ without making an exception for the browserid.org
external JavaScript include.

To fix this, include https://browserid.org in your script-src directive. If
you're using the `django-csp`_ library, the following settings will work::

    CSP_SCRIPT_SRC = ("'self'", 'https://browserid.org','https://login.persona.org')
    CSP_FRAME_SRC = ("'self'", 'https://browserid.org','https://login.persona.org')

.. note:: The example above also includes the frame-src directive. There is an
   iframe used during BrowserID login, but some people report that login will
   work without the directive. In general, you should probably include it.

.. _Content Security Policy: https://developer.mozilla.org/en/Security/CSP
.. _django-csp: https://github.com/mozilla/django-csp


Site keeps redirecting to the same page repeatedly
--------------------------------------------------

Sometimes, after attempting to login, you might notice that the page keeps
reloading itself over and over. This usually means that something has gone wrong
in your login process, and you should check the log output as well as the
solutions below to see if they can point you in the right direction.

The reason for the repeating redirects has to do with Persona, the default
BrowserID server that ``django-browserid`` uses. If you have attempted to log in
to a site via Persona, and the site fails to accept your login, Persona will
continue to attempt to log you in if the JavaScript shim that it provides is
included on the page.

The easiest way to get around this is to simply not include the login form on
any pages when the user is logged in. ``django-browserid`` attempts to avoid
these infinite loops in certain cases, but they may still come up if, for
example, ``SESSION_COOKIE_SECURE`` is True on a development instance without
SSL.


Login fails silently due to SESSION_COOKIE_SECURE
-------------------------------------------------

If you try to login on a local instance of a site and login fails without any
error (typically redirecting you back to the login page), check to see if you've
set `SESSION_COOKIE_SECURE` to True in your settings.

`SESSION_COOKIE_SECURE` controls if the `secure` flag is set on the session
cookie. If set to True on a local instance of a site that does not use HTTPS,
the session cookie won't be sent by your browser because you're using an HTTP
connection.

The solution is to set `SESSION_COOKIE_SECURE` to False on your local instance,
typically by adding it to `settings/local.py`::

    SESSION_COOKIE_SECURE = False


Login fails silently due to cache issues
----------------------------------------

Another possible cause of silently failing logins is an issue with having no
cache configured locally. Several projects (especially projects based on
playdoh_, which uses `django-session-csrf`_) store session info in the cache
rather than the database, and if your local instance has no cache configured,
the session information will not be stored and login will fail silently.

To solve this issue, you should configure your local instance to use an
in-memory cache with the following in your local settings file::

    CACHES = {
        'default': {
            'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
            'LOCATION': 'unique-snowflake'
        }
    }

.. _playdoh: https://github.com/mozilla/playdoh
.. _django-session-csrf: https://github.com/mozilla/django-session-csrf
