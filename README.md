# PlutoServe

PlutoServe is a simple routing based httpserver-wrapper for pluto

## Request

### Type

Specifies the request-type, e.g. `GET`, `POST`, `PUT`, etc.

```lua
print(request.type)
print(request:getType())
```

### Path & Query

Specifies the complete requested path, e.g. `/test?argument1=6&debug=true`

```lua
print(request.path) -- /test?argument1=6&debug=true
print(request.getPath()) -- /test?argument1=6&debug=true
```

`getCleanPath(): string` allows you to only get the path, without any queries

```lua
print(request:getCleanPath()) -- /test
```

`getQueryString(): string` allows you to only get the query-string, without the path

```lua
print(request:getQueryString()) -- argument1=6&debug=true
```

`getQueryValue(key: string): ?string` allows you to get the value of a query-string argument

```lua
print(request:getQueryValue("arg")) -- nil
print(request:getQueryValue("argument1")) -- 6
print(request:getQueryValue("debug")) -- true
```

### Version

Specifies the http version, e.g. `HTTP/1.0`, `HTTP/1.1`, etc.

```lua
print(request.version)
print(request:getVersion())
```

### Host

Specifies the host the client requested, e.g. `google.com`, `localhost:80`, etc.

```lua
print(request:getHost())
```

### Useragent

Returns the clients useragent, e.g. `Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:124.0) Gecko/20100101 Firefox/124.0`

```lua
print(request:getUseragent())
```

### Response

Every Request gets a `response` of type `Response` assigned automatically. You can access it via the `getResponse(): userdata` function

```lua
print(request.response)
print(request:getResponse())
request:getResponse():setString("Hello World"):sendResponse()
```

### Headers

You can access the headers of a request like this

```lua
print(request.headers) -- returns table of headers
print(request.headers["Host"]) -- localhost:80
```

## Response

### Content-Type

You can set the content-type of a response via `setContentType(contentType: string): userdata`

```lua
response:setContentType("text/plain")
response:setContentType("text/html; charset=utf-8")
```

### Response-String

You can set the response-string of a response via `setString(response: string): userdata`

```lua
response:setString("Hello World :D")
```

### Status

You can set the status-code of the response via `setStatus(status: string): userdata`

```lua
response:setStatus("418 I'm a teapot")
```

### Header

You can set a specific header of the response via `setHeader(header: string, value: string): userdata`

```lua
response:setHeader("Cache-Control", "max-age=600")
```

### Send Response

To finally send your response, you can use the `sendResponse()` function

```lua
response:sendResponse()
```

**Note**: all functions of the `Response` class return `self` so you can chain functions like such

```lua
response:setContentType("text/html; charset=utf-8")
        :setString("<h1>404 Not Found</h1>")
        :setStatus("404 Not Found")
        :sendResponse()
```

## Router

### Create Router

When creating a router you can give it the port it should listen on. The default port is 80

```lua
local router = new Router(80)
```

### Routes

PlutoServe is based on routes, meaning you have to specify which routes to listen to. There are currently 2 types of routes, `strict routes` and `slack routes`

- Strict routes will only match if the path **AND** the query string match
- Slack routes will match if the path matches

You can add routes via the `addRoute(route: string, fn: function, slackRoute: bool = false)` function. `slackRoute` specifies whether or not the route will be a `slack route` or a `strict route`

- Adding a strict route

```lua
-- will match "/hi" but not "/hi?arg1=true"
router:addRoute("/hi", function(req)
    req.response:setString("Hello :D"):setContentType("text/plain"):sendResponse()
end)
```

- Adding a slack route

```lua
-- will match "/hi" AND "/hi?arg1=true" but due to priority "/hi" will be handled by the first route instead
router:addRoute("/hi", function(req)
    req.response:setString($"Hi, Your Query Is: {req:getQueryString()}"):setContentType("text/plain"):sendResponse()
end, true)
```

### Middleware

A routers middleware is a table of functions that will be called on every request. You can use them to e.g. log requests or handle requests differently, not just based on their path

```lua
-- gets called for every request
router:addMiddleware(function(req)
    print($"{os.date("%c")}: {req:getPath()} - {req:getUseragent()}") -- log path and useragent of every request
end)
```

### Fallback

Whenever a router cannot find a matching route for a request, the routers fallback function gets called on the request. This function also decides what to respond to the request (e.g. a 404 error page)

```lua
-- function to be called when no matching route was found
router:setFallback(function(req)
    print($"{os.date("%c")}: /!\\ \"{req:getPath()}\" was requested but not found! /!\\") -- additional logging for requests that didnt match any routes

    req.response:setContentType("text/plain")
        :setString($"No Route Found For \"{req:getPath()}\" :["):setStatus("404 NOT_FOUND")
        :sendResponse()
end)
```

### Start Router

To start a router, simply call `startListening()` on the router

```lua
router:startListening()
```