Middlewares
===========

Silex allows you to run code at different stages during the handling of a
request through middlewares to let you change the default Silex behavior:

* *Application middlewares* are triggered independently of the current handled
  request;

* *Route middlewares* are triggered when their associated route is matched.

Application Middlewares
-----------------------

The application middlewares are only run for the "master" Request.

Before Middleware
~~~~~~~~~~~~~~~~~

A *before* middleware allows you to tweak the Request before the controller is
executed::

    $app->before(function (Request $request) {
        // ...
    });

By default, the middleware is run after the routing and the security.

If you want your middleware to be run even if an exception is thrown early on
(on a 404 or 403 error for instance), then, you need to register it as an
early event::

    $app->before(function (Request $request) {
        // ...
    }, Application::EARLY_EVENT);

Of course, in this case, the routing and the security won't have been
executed, and so you won't have access to the locale, the current route, or
the security user.

.. note::

    The before middleware is an event registered on the Symfony *request*
    event.

After Middleware
~~~~~~~~~~~~~~~~

An *after* middleware allows you to tweak the Response before it is sent to
the client::

    $app->after(function (Request $request, Response $response) {
        // ...
    });

.. note::

    The after middleware is an event registered on the Symfony *response*
    event.

Finish Middleware
~~~~~~~~~~~~~~~~~

A *finish* middleware allows you to execute tasks after the Response has been
sent to the client (like sending emails or logging)::

    $app->finish(function (Request $request, Response $response) {
        // ...
        // Warning: modifications to the Request or Response will be ignored
    });

.. note::

    The finish middleware is an event registered on the Symfony *terminate*
    event.

Route Middlewares
-----------------

Before Middleware
~~~~~~~~~~~~~~~~~

* ``before`` middlewares are fired just before the route callback, but after
  the application ``before`` middlewares;

After Middleware
~~~~~~~~~~~~~~~~

* ``after`` middlewares are fired just after the route callback, but before
  the application ``after`` middlewares.

This can be used for a lot of use cases; for instance, here is a simple
"anonymous/logged user" check::

    $mustBeAnonymous = function (Request $request) use ($app) {
        if ($app['session']->has('userId')) {
            return $app->redirect('/user/logout');
        }
    };

    $mustBeLogged = function (Request $request) use ($app) {
        if (!$app['session']->has('userId')) {
            return $app->redirect('/user/login');
        }
    };

    $app->get('/user/subscribe', function () {
        ...
    })
    ->before($mustBeAnonymous);

    $app->get('/user/login', function () {
        ...
    })
    ->before($mustBeAnonymous);

    $app->get('/user/my-profile', function () {
        ...
    })
    ->before($mustBeLogged);

For convenience, the ``before`` middlewares are called with the current
``Request`` instance as an argument and the ``after`` middlewares are called
with the current ``Request`` and ``Response`` instance as arguments.

Middlewares Priority
--------------------

You can add as many middlewares as you want, in which case they are triggered
in the same order as you added them.

You can explicitly control the priority of your middleware by passing an
additional argument to the registration methods::

    $app->before(function (Request $request) {
        // ...
    }, 32);

As a convenience, two constants allow you to register an event as early as
possible or as late as possible::

    $app->before(function (Request $request) {
        // ...
    }, Application::EARLY_EVENT);

    $app->before(function (Request $request) {
        // ...
    }, Application::LATE_EVENT);

Short-circuiting the Controller
-------------------------------

If a before middleware returns a Response object, the Request handling is
short-circuited (the next middlewares won't be run, neither the route
callback), and the Response is passed to the after middlewares right away::

    $app->before(function (Request $request) {
        // redirect the user to the login screen if access to the Resource is protected
        if (...) {
            return new RedirectResponse('/login');
        }
    });

.. note::

    If a before middleware does not return a Response or ``null``, a
    ``RuntimeException`` is thrown.
