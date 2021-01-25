# restify-lambda

A (somewhat) in-place Restify implementation for AWS Lambda.

Forked from [lambda-restify](https://github.com/kksharma1618/lambda-restify) as it appears that project is no longer maintained.

## Fork Differences

Current differences between lambda-restify:
- CORS is pre-implemented, meaning if a valid endpoint is hit with an OPTIONS method, a 200 will be responded. 
- Some missing properties and methods have been added to the request.ts class to match the HTTP Request class
  (props: `route`; fns: `on`, `once`, `resume`) - however these are likely not implemented exactly as they should be as
  I've not spent too much time looking into their purpose.
- Allows a JSON body in a request with a content type of 'url-encoded-form' to allow for API
  Gateway usage.
- `request.query` is a getter rather than just a function, meaning the query object can be accessed the same way as on the
  standard Restify request object.
- Implemented routing via Route Name to the next function in the call chain (for middleware/endpoints) to match Restify
  functionality.

### Installation
```
npm install --save restify-lambda
```

## Original Documentation

### What is it about
If you are writing [aws lambda function](https://aws.amazon.com/lambda/) to develop rest apis using [aws api gateway](https://aws.amazon.com/api-gateway/), this package will help you with request/response/routing/middlewares/versioned apis type features generally found in packages like restify or express.

Instead of using http module for opening a server and listening for incoming requests, this package relies on lambda event and callback.

When you make an http request against aws apigateway it triggers aws lambda with an event containing all the information about the incoming request (like method, url, querystring, headers, and body). lambda-restify relies on that information to create request object.

When your route handler sends response back (including headers, content), lambda-restify triggers lambda callback.

### Supported features
- Full support for restify request/response api
- Pre routing hooks 
- Middlewares
- Routing
- Versioned apis

### Dependency
It requires node >= 6.10.0. Make sure you choose "6.10.2" or above while creating lambda function. At the time of writing lambda supports v4.3.2 and 6.10.2.

### Getting started

#### Create server
See list of supported options [here](https://github.com/kksharma1618/lambda-restify/blob/master/src/lib/server_options.ts).

``` javascript
const Server = require('lambda-restify').default;
const server = new Server(options);
```

Or, if you are using imports

``` javascript
import Server from 'lambda-restify';
const server = new Server(options);
```

#### Attach your routes and middlewares
See [restify documentation](http://restify.com/docs/home/) for documentation on server.pre, server.use, server.get (and other http verbs). Since lambda-restify uses restify like interface all that docs apply here as well.

``` javascript
server.pre(function(req, res, next) {
    // this handler is run for all routes, even 404
    
    // do something here
    next()
})

server.use(function(req, res, next) {
    // this handler is run for after routing is done 
    // and successful match is found (not on 404)

    // do something here
    next()
})

server.post('/user/:id', function(req, res) {
    
    // headers available 
    const apiKey = req.header('apikey');
    
    // route param
    const userId = req.params.id;
    
    // query string
    const queryValue = req.query('queryKey')

    // body
    const name = req.body.name

    // send response with res.json or res.send
    res.json({
        status: 1
    })
})

// define other route handlers

```

#### Attach lambda handler
``` javascript
exports.yourlambdaHandler = function(event, context, callback) {
    server.handleLambdaEvent(event, context, callback)
}
```

### Documentation

**Note**<br />
Most likely, you will need to set: 
``` javascript 
context.callbackWaitsForEmptyEventLoop = false 
```
before calling server.handleLamdaEvent in your lamda handler. See [http://docs.aws.amazon.com/lambda/latest/dg/nodejs-prog-model-context.html](http://docs.aws.amazon.com/lambda/latest/dg/nodejs-prog-model-context.html) for details. 

See [restify documentation](http://restify.com/docs/home/). Following items work just as they did in restify:
- Request:
    - headers
    - url
    - httpVersion
    - method
    - params
    - body
    - rawBody
    - header(name: string, defaultValue: string)
    - accepts(type: string | string[])
    - acceptsEncoding(type: string | string[])
    - getContentLength()
    - contentLength() [alias of getContentLength]
    - getContentType()
    - contentType() [alias of getContentType]
    - time()
    - date()
    - getQuery()
    - query() [alias of getQuery()]
    - getUrl()
    - href()
    - id(reqId?: string)
    - getId()
    - getPath()
    - path() [alias of getPath]
    - is(type: string)
    - isSecure()
    - isChunked()
    - toString()
    - userAgent()
    - version()
    - matchedVersion()
    - trailer(name: string, value?: string) [no trailers support. it just pass back the default value]
    - isKeepAlive()
    - isUpload()
- Response
    - finished
    - headersSent
    - sendDate
    - statusCode
    - statusMessage
    - serverName
    - cache(type?: any, options?: any)
    - noCache()
    - header(name: string, value?: any)
    - setHeader(name: string, value: any)
    - getHeaders()
    - headers() [alias of getHeaders]
    - send(code?: number, body?: string | json, headers?: json)
    - sendRaw(code?: number, body?: string | json, headers?: json)
    - removeHeader(name: string)
    - writeHead(code?, message?, headers?)
    - write(chunk: string | Buffer, encoding?: string, callback?: any)
    - end(data?: string | Buffer, encoding?: string, callback?)
    - get(name: string)
    - json(code?, body?, headers?)
    - link(l, rel)
    - charSet(type: string)
    - redirect(...)
    - status(code: number)
    - set(name: string | object, val?: string)
    - getHeaderNames()
    - hasHeader(name: string)
- Server
    - pre(handlers)
    - use(handlers)
    - get(path?, options?, handlers)
    - del(path?, options?, handlers)
    - head(path?, options?, handlers)
    - opts(path?, options?, handlers)
    - post(path?, options?, handlers)
    - put(path?, options?, handlers)
    - patch(path?, options?, handlers)
    - param(name, fn)
    - versionedUse(versions: string | string[], fn)

**Server.handleLambdaEvent(lambdaEvent, context, lambdaCallback)**<br />
Plug this into lambda handler to route all incoming lambda events.
