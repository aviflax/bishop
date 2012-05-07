Bishop is a [Webmachine](http://wiki.basho.com/Webmachine.html)-like
library for Clojure. Bishop provides tools that make it easy and
straightforward for your web-service to treat
[HTTP](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) as a
first-class application protocol. The library handles things like
content negotiation and predictable caching behavior, leaving you to
concentrate on a building a clean and consistent API be it
[REST-ful](http://en.wikipedia.org/wiki/REST) or even
[HATEOAS](http://en.wikipedia.org/wiki/HATEOAS) compliant.

When you create a “resource” with Bishop, you receive a function that
expects a map request values and will return a map of response
values. This library was designed to be used with
[Ring](https://github.com/mmcgrana/ring) and should work with any Ring
middle-ware. Bishop provides its own routing mechanism but you can use
another if you like (for instance
[Clout](https://github.com/weavejester/clout)).

This is our first release of this library and there may be bugs that
need squashing, please
[register an issue](https://github.com/tnr-global/bishop-sample/issues)
if you notice any or send us a pull request if you fix them. We do
have
[a sample application](https://github.com/tnr-global/bishop-sample)
that we put together, I’m also working on implementing an application
for production use that leverages this library.

## Aren't There Other Projects that Do This?

Yes, there are several other projects that are looking to do this
very same thing. The ones that I am aware of are...

*  [Clothesline](https://github.com/banjiewen/Clothesline)

*  [Plugboard](https://github.com/malcolmsparks/plugboard)

This project has slightly different goals from those mentioned
above. For one, this project isn't particularly interested in
exposing a nice interface to Java code. Our primary concern is to
make things easier for the Clojure developer.

Plugboard is constructed on top of the excellent
[Compojure](https://github.com/weavejester/compojure) library which in
turn builds on Ring, this project instead builds on top of Ring
directly. The web APIs that I have constructed so far have been coded
on Ring and I don't want to pull Compojure into the mix.

## Installation

To use Bishop, add the following to your project’s “:dependencies”:

```
[tnrglobal/bishop "1.0.1"]
```

## How Does it Work?

Anyway, let's say you have a function that will say "Hello" to
people. Add the `com.tnrglobal.bishop.core` namespace to your project.

```
(ns hello.core
  (:require [com.tnrglobal.bishop.core :as bishop]
            [ring.adapter.jetty]
            [ring.middleware.params]))
```
We also define the function that does our work.

```
(defn hello
  [name]
  (str "Hello " name "!"))
```

We can then define a resource that says "Hello" in HTML or JSON.

```
(def hello-resource
  (bishop/resource
    {"text/html" (fn [request]
      (hiccup/html
        [:p (hello (:name (:path-info request)))]))}

    {"text/json" (fn [request]
      (clj-json/generate-string
        {:message (hello (:name (:path-info request)))}))}))
```

This resource can return either HTML or JSON content, depending on
the headers of the request. It expects to have a value in the
"path-info" map under the ":name" key. This comes from the routing.

```
(def routes
  {["hello" :name] hello-resource
  ["*"] (bishop/halt-resource 404)})
```

We route incoming request for "/hello/something" to our
"hello-resource" functions, anything else will result in sending a
"404" code to the client. Bishop will parse the route and the
request's URI to populate the "path-info" map for your application,
the goal is to do it in the same way that
[Webmachine handles dispatch](http://wiki.basho.com/Webmachine-Dispatching.html).

Lastly, you can add this as your Ring handler function.

```
(def app
  (-> (bishop/handler routes)
  (wrap-params)))
```

Aside from parsing the URI and matching it to the route, Bishop is
doing a lot of other work as well. It will cover all of the behavior
in this
[HTTP 1.1 flow chart](http://wiki.basho.com/Webmachine-Diagram.html). In
this instance, Bishop is...

* Parsing the client URI and route, then populating the "path-info"
map

* Verifying that the client is trying to either GET or HEAD the
resource.

* Making sure that the length of the URI isn't totally nutty.

* Verifying that our resource can accept either a GET or a HEAD
request.

* Selecting the appropriate content type to provide to the client and
sending the appropriate error if the client asks for a resource that
we do not provide.

And so on.

## Sample Application

We have put a small, [simple application](https://github.com/tnr-global/bishop-sample) together to make
sure that the library builds and mostly functions. If you’re
interested, you are welcome to take a look at it.

[https://github.com/tnr-global/bishop-sample](https://github.com/tnr-global/bishop-sample)