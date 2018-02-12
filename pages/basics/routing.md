---
title: Routing and Piping
summary: "kiwi provides the mechanisms 'piping' and 'routing'. You can <i>pipe</i> a PSR-15 middleware to your application or <i>route</i> an incoming request (on defined criteria) to a given action."
sidebar: doc_sidebar
permalink: basics_routing.html
folder: basics
---

## General
Handling HTTP Request is build on top of [zend-expressive](https://github.com/zendframework/zend-expressive). kiwi uses 
PSR-7 and PSR-15 for handling server requests and middleware components.

Below a diagram how kiwi processes a http request:

{% include image.html file="psr15_layers.png" url="images/psr15_layers.png" caption="PSR-15 Layers" %}

Middleware can be injected anywhere in the pipe, specify by a priority integer to dictate the order in which middleware 
should be attached and executed; returning a response from middleware prevents any middleware attached later from executing.

_Routing_ within kiwi consists of matching the request to a route. Matching works by checking the requested URI path along 
with allowed HTTP methods. _Dispatching_ is simply the act of calling the action mapped by routing.

## PipeConfigurator

The pipeConfigurator is a little helper which supports you to register pipes and routes. The 
pipeConfigurator is injected to `bootstrap/pipe.php`.

A small example:

```php
$pipeConfigurator->pipe(BaseMiddleware1::class);
$pipeConfigurator->segment('/api', function (PipeConfigurator $pipeConfigurator) {
    $pipeConfigurator->pipe(BaseMiddleware2::class);
    
    $pipeConfigurator->group(function (GroupPipeConfigurator $groupPipeConfigurator) {
        $groupPipeConfigurator->before(AuthorizationGuardMiddleware::class);
    
        $groupPipeConfigurator->get('/auth/user', UserAction::class, "api.auth.user");
        $groupPipeConfigurator->post('/auth/logout', LogoutAction::class, "api.auth.logout");
    
    });
});

$pipeConfigurator->get('[/]', IndexAction::class, "index");
```

### Routing Methods
The following six methods are supported to register routes:
```php
$pipeConfigurator->get('/api/user', UserIndex::class, 'user.index');
$pipeConfigurator->post('/api/user', UserAdd::class, 'user.add');
$pipeConfigurator->patch('/api/user', UserChange::class, 'user.change');
$pipeConfigurator->put('/api/user', UserEdit::class, 'user.edit');
$pipeConfigurator->delete('/api/user', UserDelete::class, 'user.delete');
$pipeConfigurator->any('/do-something', DoSomething::class, 'do.something');
```

In some cases you might need to add a middleware which is executed before or after the actual action.
All the above routing methods have a 4th optional parameter, which gives you the possibility to modify 
the routing configuration a little further.

```php
$pipeConfigurator->any('/do-something', DoSomething::class, 'do.something', function(RouteConfigurator $routeConfigurator) {
    $routeConfigurator->before(DoSomethingBeforeAction::class);
    
    $routeConfigurator->disablePatch();
});
```

kiwi uses [FastRoute](https://github.com/nikic/FastRoute) for matching the incoming request to a given criteria. Some routing examples:
```php

// Matches /do-something/hello, but not /do-something/hello/world
$pipeConfigurator->any('/do-something/{title}', DoSomething::class, 'do.something');

// Matches /do-something/hello and /do-something/hello/world
$pipeConfigurator->any('/do-something/{title:.+}', DoSomething::class, 'do.something');

// Matches /do-something/42, but not /do-something/abc
$pipeConfigurator->any('/do-something/{id:\d+}', DoSomething::class, 'do.something');

// Matches /do-something/de, but not /do-something/fr
$pipeConfigurator->any('/do-something/{lang:en|de}', DoSomething::class, 'do.something');

// Matches /do-something/42 and /do-something/42/abc
$pipeConfigurator->any('/do-something/{id:\d+}[/{name}]', DoSomething::class, 'do.something');
```


{% include note.html content="For digging deeper into the router consider further reading on [FastRoute](https://github.com/nikic/FastRoute)." %}


### Grouping of Routing Methods

Sometimes you want to add the same middlewares to a set of routes (for example: a middleware which
checks authentication/authorization for all routes available when logged in).
You can achieve this by using the `group` method:

```php
$pipeConfigurator->group(function (GroupPipeConfigurator $groupPipeConfigurator) {
    $groupPipeConfigurator->before(AuthenticationGuardMiddleware::class);
    $groupPipeConfigurator->before(AuthorizationGuardMiddleware::class);
    $groupPipeConfigurator->before(SecurityGuardMiddlware::class);
    $groupPipeConfigurator->after(LoggingWhenActionDoesntGenerateResponse::class);
    
    $groupPipeConfigurator->get('/auth/user', UserAction::class, "auth.user");
    $groupPipeConfigurator->post('/auth/logout', LogoutAction::class, "auth.logout");
    
});
```

This is equivalent to the following:

```php
$pipeConfigurator->get('/auth/user', UserAction::class, "auth.user" function(RouteConfigurator $routeConfigurator){
    $routeConfigurator->before(AuthenticationGuardMiddleware::class);
    $routeConfigurator->before(AuthorizationGuardMiddleware::class);
    $routeConfigurator->before(SecurityGuardMiddlware::class);
    $routeConfigurator->after(LoggingWhenActionDoesntGenerateResponse::class);
});

$pipeConfigurator->post('/auth/logout', UserAction::class, "auth.logout" function(RouteConfigurator $routeConfigurator){
    $routeConfigurator->before(AuthenticationGuardMiddleware::class);
    $routeConfigurator->before(AuthorizationGuardMiddleware::class);
    $routeConfigurator->before(SecurityGuardMiddlware::class);
    $routeConfigurator->after(LoggingWhenActionDoesntGenerateResponse::class);
});
```

### Segmenting Middleware
You might need to _segment_ your middleware pipes into several pipes (for example you want to segregate between
api and frontend calls) by segregating your middleware pipeline and routes.
You can achieve this by using the `segment` method:
```php
$pipeConfigurator->segment('/api', function (PipeConfigurator $pipeConfigurator) {
    //passed $pipeConfigurator is a new instance of PipeConfigurator just for everything under /api
    $pipeConfigurator->get('/books', BookListAction::class, "api.book.list");
    $pipeConfigurator->get('/book/{id}', BookDetailAction::class, "api.book.detail");
});

$pipeConfigurator->segment('/admin', function (PipeConfigurator $pipeConfigurator) {
    //passed $pipeConfigurator is a new instance of PipeConfigurator just for everything under /admin
    $pipeConfigurator->get('/auth', AuthAction::class, "admin.auth");
});

```

{% include note.html content="Segments can be nested. Even the root is just a segment with the segment path ''" %}


### Middleware priority
Middlewares will be added to a queue and will be dequeued until a middleware returns a PSR-7 response.
The second parameter of the `pipe` and the `segment` method defines the priority of the given middleware.
Middleware with a higher priority will be executed earlier.
Priority above `500000` are considered as global middleware. They will run on every request of the segment. Middleware between
`500000` and `1000` are post-routing middlewares. The router tried to match the request to a given route and sets a `RouteResult`.
Middleware below `1000` has already dispatched the matched route to the registered action.

{% include warning.html content="Priority `500000` and `1000` are reserved for internal usage and can't be assigned." %}

## Middleware and Actions

From a technical `actions` are the same as a `middleware`. Both implement the PSR-15 MiddlewareInterface, both can return
a PSR-7 Response or delegate to the next middleware in the pipe. Unlike in a classic MVC workflow, routing doesn't _just_ 
delegate to a target (controller and action), it defines complete pipeline and executes it in order. 
Because an action is defined by a route, the action will be by definition called after _dispatching_.

### Accessing routing parameters

In the following example you can see how you can access routing parameters:
```php
$pipeConfigurator->any('/do-something/{title}', DoSomething::class, 'do.something');
```

You can access `title` inside a middleware like this:

```php
public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
{
    $request->getAttribute("title");
}
```

{% include note.html content="Only middlewares executed after the routing can access routing parameters." %}

### Passing data between middleware
```php
public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
{
    return $handler->handle($request->withAttribute('data', ['foo' => 'bar']));
}
```

You can access `data` inside a later middleware:

```php
public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
{
    $request->getAttribute("data");
}
```
