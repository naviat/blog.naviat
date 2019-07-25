---
layout: post
tags: golang middleware
title: Writing delightful HTTP middleware in Go
---
Writing delightful HTTP middleware in Go

While writing complex services in go, one typical topic that you will encounter is middleware. This topic has been discussed again, and again, and again, on internet. Essentially a middleware should allow us to:

Intercept a ServeHTTP call, and execute any arbitrary code.
Make changes to request/response flow along continuation chain.
Break the middleware chain, or continue onto next middleware interceptor eventually leading to real request handler.
All of this would sound very similar to what express.js middleware do. We explored  various libraries and found existing solutions that closely matched what we wanted, but they either had unnecessary extras, or were not delightful for our taste buds. It was pretty obvious that we can write express.js inspired middleware, with cleaner installation API under 20 lines of code ourselves.

* Intercept a ServeHTTP call, and execute any arbitrary code.
* Make changes to request/response flow along continuation chain.
* Break the middleware chain, or continue onto next middleware interceptor eventually leading to real request handler.

All of this would sound very similar to what [express.js middleware](https://expressjs.com/en/guide/using-middleware.html) do. We explored  [various libraries](!https://lmgtfy.com/?q=golang+middleware+library) and [found existing solutions](https://github.com/urfave/negroni#handlers) that closely matched what we wanted, but they either had [unnecessary extras](https://github.com/urfave/negroni#bundled-middleware), or were [not delightful](https://github.com/go-midway/midway#basic-design) for our taste buds. It was pretty obvious that we can write [express.js](https://expressjs.com/en/guide/using-middleware.html#middleware.router) inspired middleware, with cleaner installation API under 20 lines of code ourselves.

## The abstraction

While designing the abstraction, we first imagined how we want to write our middleware functions (referred as interceptors from this point onwards), and the answer was pretty obvious:

```go
func NewElapsedTimeInterceptor() MiddlewareInterceptor {
    return func(w http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
        startTime := time.Now()
        defer func() {
            endTime := time.Now()
            elapsed := endTime.Sub(startTime)
            // Log the consumed time
        }()

        next(w, r)
    }
}

func NewRequestIdInterceptor() MiddlewareInterceptor {
    return func(w http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
        if r.Headers.Get("X-Request-Id") == "" {
            r.Headers.Set("X-Request-Id", generateRequestId())
        }

        next(w, r)
    }
}
```

They look just like <span style="color:red"> http.HandlerFunc </span>, but having an extra parameter <span style="color:red"> next </span> that continues the handler chain. This would allow anyone to write interceptor as simple function similar to <span style="color:red"> http.HandlerFunc </span> that can intercept the call, do what they want, and pass on the control if they want to.

Next we imagined how to hook these interceptors to our <span style="color:red"> http.Handler </span> or <span style="color:red"> http.HandlerFunc </span>. In order to do so, first thing do is define <span style="color:red"> MiddlewareHandlerFunc </span>  which is simply a type of <span style="color:red"> MiddlewareHandlerFunc </span> (i.e. <span style="color:red"> type MiddlewareHandlerFunc http.HandlerFunc </span> ). This will allow us to build a nicer API on top of stock <span style="color:red"> MiddlewareHandlerFunc </span>. Now given a <span style="color:red"> http.HandlerFunc </span> we want our chain-able API to look somewhat like this:

```go
func HomeRouter(w http.ResponseWriter, r *http.Request) {
		// Handle your request
}

// ...
// Some where when installing Hanlder
chain := MiddlewareHandlerFunc(HomeRouter).
  Intercept(NewElapsedTimeInterceptor()).
  Intercept(NewRequestIdInterceptor())

// Install it like regular HttpHandler
mux.Path("/home").HandlerFunc(http.HandlerFunc(chain))
```

Casting <span style="color:red"> http.HandlerFunc </span> to <span style="color:red"> MiddlewareHandlerFunc </span>, and then calling <span style="color:red"> Intercept </span> method to install our interceptor. The return type of Intercept is again a <span style="color:red"> MiddlewareHandlerFunc </span>, which allows us to call <span style="color:red"> Intercept </span> again.

One important thing to note with <span style="color:red"> Intercept </span> scheme is the order of execution. Since calling <span style="color:red"> chain(responseWriter, request) </span> is indirectly invoking last interceptor, the execution of interceptors is reversed i.e. it goes from interceptor at tail all the way back to handler at head. Which makes perfect sense because you are intercepting the call; so you should get a chance to execute before your parent.

## The simplification

While this reverse chaining system makes the abstraction more fluent, turns out most the time we have a precompiled array of interceptors that will be reused with different handlers. Also when we are defining a chain of middleware as an array, we would naturally prefer to declare them in the order of their execution (not reversed order). Let’s call this array interceptors <span style="color:red"> MiddlewareHandlerFunc </span>. We want our middleware chains to look somewhat like:

```go
// Chain or middleware to be invoked in order of their index
middlewareChain := MiddlewareChain{
  NewRequestIdInterceptor(),
  NewElapsedTimeInterceptor(),
}

// Invoke all middlewares ending up with HomeRouter
mux.Path("/home").Handler(middlewareChain.Handler(HomeRouter))
```

Note these middleware will be invoked in same order as the appear in the chain i.e. <span style="color:red"> RequestIdInterceptor </span> and <span style="color:red"> ElapsedTimeInterceptor </span>. This adds both reusability, and readability to our code.

## The implementation

Once we designed the abstraction the implementation was pretty straight forward:

```go
/*
Copyright (c) 2019 Hai Dam
Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
*/
package middleware

import "net/http"

// MiddlewareInterceptor intercepts an HTTP handler invocation, it is passed both response writer and request
// which after interception can be passed onto the handler function.
type MiddlewareInterceptor func(http.ResponseWriter, *http.Request, http.HandlerFunc)

// MiddlewareHandlerFunc builds on top of http.HandlerFunc, and exposes API to intercept with MiddlewareInterceptor.
// This allows building complex long chains without complicated struct manipulation
type MiddlewareHandlerFunc http.HandlerFunc


// Intercept returns back a continuation that will call install middleware to intercept
// the continuation call.
func (cont MiddlewareHandlerFunc) Intercept(mw MiddlewareInterceptor) MiddlewareHandlerFunc {
	return func(writer http.ResponseWriter, request *http.Request) {
		mw(writer, request, http.HandlerFunc(cont))
	}
}

// MiddlewareChain is a collection of interceptors that will be invoked in there index order
type MiddlewareChain []MiddlewareInterceptor

// Handler allows hooking multiple middleware in single call.
func (chain MiddlewareChain) Handler(handler http.HandlerFunc) http.Handler {
	curr := MiddlewareHandlerFunc(handler)
	for i := len(chain) - 1; i >= 0; i-- {
		mw := chain[i]
		curr = curr.Intercept(mw)
	}

	return http.HandlerFunc(curr)
}
```

So under 20 lines of code (excluding comments), we were able to build a nice middleware library. It’s almost barebones, but the coherent abstraction of these few lines is just amazing. It enables us to write some slick middleware chains, without any fuss. Hopefully these few lines will inspire your middleware experience to be delightful as well.



