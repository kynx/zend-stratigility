# Executing and composing middleware

The easiest way to execute middleware is to write closures and attach them to a
`Zend\Stratigility\MiddlewarePipe` instance. You can nest `MiddlewarePipe`
instances to create groups of related middleware, and attach them using a base
path so they only execute if that path is matched.

```php
$api = new MiddlewarePipe();  // API middleware collection
$api->pipe(/* ... */);        // repeat as necessary

$app = new MiddlewarePipe();  // Middleware representing the application
$app->pipe('/api', $api);     // API middleware attached to the path "/api"
```

## Handling errors

While the above will give you a basic application, it has no error handling
whatsoever. We recommend adding an initial middleware layer using the
`Zend\Stratigility\Middleware\ErrorHandler` class:

```php
use Zend\Diactoros\Response;
use Zend\Stratigility\Middleware\ErrorHandler;

$app->pipe(new ErrorHandler(new Response());
// Add more middleware...
```

You can learn how to customize the error handler to your needs in the
[chapter on error handlers](error-handlers.md).

## Extending the MiddlewarePipe

Another approach is to extend the `Zend\Stratigility\MiddlewarePipe` class
itself, particularly if you want to allow attaching other middleware to your
own middleware. In such a case, you will generally override the `process()`
method to perform any additional logic you have, and then call on the parent in
order to iterate through your stack of middleware:

```php
use Interop\Http\Middleware\DelegateInterface;
use Interop\Http\Middleware\ServerMiddlewareInterface;
use Psr\Http\Message\ServerRequestInterface as Request;

class CustomMiddleware extends MiddlewarePipe
{
    public function process(Request $request, DelegateInterface $delegate)
    {
        // perform some work...

        // delegate to parent
        parent::process($request, $delegate);

        // maybe do more work?
    }
}
```

Another approach using this method would be to override the constructor to add
in specific middleware, perhaps using configuration provided. In this case,
make sure to also call `parent::__construct()` to ensure the middleware queue
is initialized; we recommend doing this as the first action of the method.

```php
use Zend\Stratigility\MiddlewarePipe;

class CustomMiddleware extends MiddlewarePipe
{
    public function __construct($configuration)
    {
        parent::__construct();

        // do something with configuration ...

        // attach some middleware ...

        $this->pipe(/* some middleware */);
    }
}
```

These approaches are particularly suited for cases where you may want to
implement a specific workflow for an application segment using existing
middleware, but do not necessarily want that middleware applied to all requests
in the application.