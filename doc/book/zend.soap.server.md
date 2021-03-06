# Zend\\Soap\\Server

`Zend\Soap\Server` class is intended to simplify Web Services server part development for *PHP*
programmers.

It may be used in WSDL or non-WSDL mode, and using classes or functions to define Web Service *API*.

When `Zend\Soap\Server` component works in the WSDL mode, it uses already prepared WSDL document to
define server object behavior and transport layer options.

WSDL document may be auto-generated with functionality provided by Zend\\\\Soap\\\\AutoDiscovery
component
&lt;zend.soap.autodiscovery.introduction&gt; or should be constructed manually using
Zend\\\\Soap\\\\Wsdl class
&lt;zend.soap.wsdl&gt; or any other *XML* generating tool.

If the non-WSDL mode is used, then all protocol options have to be set using options mechanism.

## Zend\\Soap\\Server constructor

`Zend\Soap\Server` constructor should be used a bit differently for WSDL and non-WSDL modes.

### Zend\\Soap\\Server constructor for the WSDL mode

`Zend\Soap\Server` constructor takes two optional parameters when it works in WSDL mode:

- `$wsdl`, which is an *URI* of a WSDL file [1].
- `$options`- options to create *SOAP* server object [2].

The following options are recognized in the WSDL mode:

- 'soap\_version' ('soapVersion') - soap version to use (SOAP\_1\_1 or *SOAP*\_1\_2).
- 'actor' - the actor *URI* for the server.
- 'classmap' ('classMap') which can be used to map some WSDL types to *PHP* classes. The option must
be an array with WSDL types as keys and names of *PHP* classes as values.
- 'encoding' - internal character encoding (UTF-8 is always used as an external encoding).
- 'wsdl' which is equivalent to `setWsdl($wsdlValue)` call.

### Zend\\Soap\\Server constructor for the non-WSDL mode

The first constructor parameter **must** be set to `NULL` if you plan to use `Zend\Soap\Server`
functionality in non-WSDL mode.

You also have to set 'uri' option in this case (see below).

The second constructor parameter (`$options`) is an array with options to create *SOAP* server
object [3].

The following options are recognized in the non-WSDL mode:

- 'soap\_version' ('soapVersion') - soap version to use (SOAP\_1\_1 or SOAP\_1\_2).
- 'actor' - the actor *URI* for the server.
- 'classmap' ('classMap') which can be used to map some WSDL types to *PHP* classes. The option must
be an array with WSDL types as keys and names of *PHP* classes as values.
- 'encoding' - internal character encoding (UTF-8 is always used as an external encoding).
- 'uri' (required) - *URI* namespace for *SOAP* server.

## Methods to define Web Service API

There are two ways to define Web Service *API* when your want to give access to your *PHP* code
through *SOAP*.

The first one is to attach some class to the `Zend\Soap\Server` object which has to completely
describe Web Service *API*:

```php
...
class MyClass {
    /**
     * This method takes ...
     *
     * @param integer $inputParam
     * @return string
     */
    public function method1($inputParam) {
        ...
    }

    /**
     * This method takes ...
     *
     * @param integer $inputParam1
     * @param string  $inputParam2
     * @return float
     */
    public function method2($inputParam1, $inputParam2) {
        ...
    }

    ...
}
...
$server = new Zend\Soap\Server(null, $options);
// Bind Class to Soap Server
$server->setClass('MyClass');
// Bind already initialized object to Soap Server
$server->setObject(new MyClass());
...
$server->handle();
```

> ## Important
You should completely describe each method using method docblock if you plan to use autodiscover
functionality to prepare corresponding Web Service WSDL.

The second method of defining Web Service *API* is using set of functions and `addFunction()` or
`loadFunctions()` methods:

```php
...
/**
 * This function ...
 *
 * @param integer $inputParam
 * @return string
 */
function function1($inputParam) {
    ...
}

/**
 * This function ...
 *
 * @param integer $inputParam1
 * @param string  $inputParam2
 * @return float
 */
function function2($inputParam1, $inputParam2) {
    ...
}
...
$server = new Zend\Soap\Server(null, $options);
$server->addFunction('function1');
$server->addFunction('function2');
...
$server->handle();
```

## Request and response objects handling

> ## Note
This section describes **advanced** request/response processing options and may be skipped.

`Zend\Soap\Server` component performs request/response processing automatically, but allows to catch
it and do some pre- and post-processing.

### Request processing

`Zend\Soap\Server::handle()` method takes request from the standard input stream ('php://input'). It
may be overridden either by supplying optional parameter to the `handle()` method or by setting
request using `setRequest()` method:

```php
...
$server = new Zend\Soap\Server(...);
...
// Set request using optional $request parameter
$server->handle($request);
...
// Set request using setRequest() method
$server->setRequest();
$server->handle();
```

Request object may be represented using any of the following:

- DOMDocument (casted to *XML*)
- DOMNode (owner document is grabbed and casted to *XML*)
- SimpleXMLElement (casted to *XML*)
- stdClass (\_\_toString() is called and verified to be valid *XML*)
- string (verified to be valid *XML*)

Last processed request may be retrieved using `getLastRequest()` method as an *XML* string:

```php
...
$server = new Zend\Soap\Server(...);
...
$server->handle();
$request = $server->getLastRequest();
```

### Response pre-processing

`Zend\Soap\Server::handle()` method automatically emits generated response to the output stream. It
may be blocked using `setReturnResponse()` with `TRUE` or `FALSE` as a parameter [4]. Generated
response is returned by `handle()` method in this case. Returned response can be a string or a
SoapFault exception object.

> ## Caution
Always check the returned response type to avoid returning SoapFault object as a string, which will
be returned to the customer as a string with the exception stacktrace.

```php
...
$server = new Zend\Soap\Server(...);
...
// Get a response as a return value of handle() method
// instead of emitting it to the standard output
$server->setReturnResponse(true);
...
$response = $server->handle();
if ($response instanceof \SoapFault) {
    ...
} else {
    ...
}
...
```

Last response may be also retrieved by `getLastResponse()` method for some post-processing:

```php
...
$server = new Zend\Soap\Server(...);
...
$server->handle();
$response = $server->getLastResponse();
if ($response instanceof \SoapFault) {
    ...
} else {
    ...
}
...
```

## Document/Literal WSDL Handling

Using the document/literal binding-style/encoding pattern is used to make SOAP messages as
human-readable as possible and allow abstraction between very incompatible languages. The Dot NET
framework uses this pattern for SOAP service generation by default. The central concept of this
approach to SOAP is the introduction of a Request and an Response object for every function/method
of the SOAP service. The parameters of the function are properties on request object and the
response object contains a single parameter that is built in the style "methodName"Result

Zend SOAP supports this pattern in both AutoDiscovery and in the Server component. You can write
your service object without knowledge about using this pattern. Use docblock comments to hint the
parameter and return types as usual. The `Zend\Soap\Server\DocumentLiteralWrapper` wraps around your
service object and converts request and response into normal method calls on your service.

See the class doc block of the `DocumentLiteralWrapper` for a detailed example and discussion.

[1] May be set later using `setWsdl($wsdl)` method.

[2] Options may be set later using `setOptions($options)` method.

[3] Options may be set later using `setOptions($options)` method.

[4] Current state of the Return Response flag may be requested with `setReturnResponse()` method.
