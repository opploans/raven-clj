# raven-clj [![raven cljdoc badge](https://cljdoc.xyz/badge/raven-clj)](https://cljdoc.xyz/d/raven-clj/raven-clj/CURRENT)

A Clojure interface to Sentry.

# _DEPRECATED_ -- do not use!!!

[Usage](#usage) | [Ring Middleware](#ring-middleware) | [Alternatives](#alternatives) | [Changes](#changes) | [Contributing](#contributing)

## Usage

[](dependency)
```clojure
[raven-clj "1.6.0-alpha3"] ;; latest release
```
[](/dependency)

`raven-clj.core/capture` is a general use function that can be
placed throughout your Clojure code to log information to your Sentry
server.

```clojure
(require '[raven-clj.core :refer [capture]])

(def dsn "https://b70a31b3510c4cf793964a185cfe1fd0:b7d80b520139450f903720eb7991bf3d@example.com/1")

(capture dsn {:message "Test Exception Message"
              :tags {:version "1.0"}
              :logger "main-logger"
              :extra {:my-key 1
                      :some-other-value "foo bar"}})
```

Various interfaces are supported. 

#### HTTP

```clojure
(require '[raven-clj.interfaces :as interfaces])

(capture dsn
        (-> {:message "Test HTTP Exception"
             :tags {:testing "1.0"}}
            (interfaces/http ring-style-request-map)))
```

#### Exceptions

```clojure
;; All namespaces in the following map will be regarded
;; as application code and highlighted in Sentry's stacktraces
;; > Note that this also matches "myapp.ns.core" etc.
(def your-namespace ["myapp.ns"])

(capture dsn
        (-> {:message "Test Stacktrace Exception"}
            (interfaces/stacktrace (Exception.) your-ns)))
```

#### Note about event-info map

In the `capture` function I use merge to merge together the final
packet to send to Sentry.  The only fields that can't be overwritten
when sending information to `capture` is `event-id` and `timestamp`.
Everything else can be overwritten by passing along a new value for
the key:

```clojure
(capture dsn
        (-> {:message "Test Stacktrace Exception"
             :logger "application-logger"}
            (interfaces/stacktrace (Exception.))))
```

Please refer to [Building the JSON Packet](https://docs.getsentry.com/hosted/clientdev/#building-the-json-packet) for more information on what
attributes are allowed within the packet sent to Sentry.

#### Uncaught exception handler

Since `1.6.0-alpha` a function that will install an [uncaught exception
handler](https://stuartsierra.com/2015/05/27/clojure-uncaught-exceptions)
is included, see `raven-clj.core/install-uncaught-exception-handler!` for details.

## Ring middleware

raven-clj also includes a Ring middleware that sends the Http and Stacktrace interfaces for Sentry packets.  Usage (for Compojure):

```clojure
(use 'raven-clj.ring)

(def dsn "https://b70a31b3510c4cf793964a185cfe1fd0:b7d80b520139450f903720eb7991bf3d@example.com/1")

;; If you want to fully utilize the Http interface you should make sure
;; you use the wrap-params and wrap-keyword-params middlewares to ensure
;; the request data is stored correctly.
(-> routes
    (wrap-sentry dsn)
    (handler/site))

;; You could also include some of the optional attributes, pass
;; your app's namespace prefixes so your app's stack frames are
;; highlighted in sentry or specify a function to alter the data
;; that is stored from the HTTP request.
(-> routes
    (wrap-sentry dsn {:extra {:tags {:version "1.0"}}
                      :namespaces ["myapp" "com.mylib"]
                      :http-alter-fn (fn [r] (dissoc r :data))})
    (handler/site))
```

## Alternatives

There are a variety of Clojure libraries for Sentry, a quick, not necessarily up-to-date overview/comparison:

- [`io.sentry/sentry-clj`](https://github.com/getsentry/sentry-clj), the official one. Wraps Sentry's own Java client. Light on dependencies.
- [`exoscale/raven`](https://github.com/exoscale/raven), a newer one. Depends on Netty. Well documented.
- [`raven-clj`](https://github.com/sethtrain/raven-clj/), the one you're looking at. Similar dependencies as Sentry's own client. Perhaps slightly more ergonomic.

## Changes

- **1.6.0-alpha3**
    - Support query params in DSNs ([#27](https://github.com/sethtrain/raven-clj/pull/27))
- **1.6.0-alpha2**
    - Switch from `clj-http-lite` to `org.martinklepsch/clj-http-lite` for Java 9+ compatibility
- **1.6.0-alpha**
    - Switch from `clj-http` to `clj-http-lite`
    - **New!** `raven-clj.core/install-uncaught-exception-handler!` can be used to install an exception handler for uncaught exceptions.
- **1.5.2**
    - Support new secret-less DSNs
- **1.5.1**
    - Prevent indexOutOfBounds exception when determining context line ([#25](https://github.com/sethtrain/raven-clj/pull/25))
- **1.5.0**
    - fix how request method is extracted from ring requests ([#22](https://github.com/sethtrain/raven-clj/pull/22))
    - send stringified ex-data ([#21](https://github.com/sethtrain/raven-clj/pull/22))
- **1.4.3**
    - fix docstring argvec order in `capture` function ([#16](https://github.com/sethtrain/raven-clj/pull/16))
    - fix NPE when a frame's file path is `nil` ([#14](https://github.com/sethtrain/raven-clj/pull/14))
- **1.4.2**
    - add missing requires
- **1.4.1 (defunct)**
    - actually depend on prone [prone](https://github.com/magnars/prone)
- **1.4.0 (defunct)**
    - use [prone](https://github.com/magnars/prone) to retrieve better stackframes ([#10](https://github.com/sethtrain/raven-clj/pull/10))
    - fix warnings in Sentry UI caused by unknown keys in request body ([#13](https://github.com/sethtrain/raven-clj/pull/13))
- **1.3.2**
    - update `clj-http` to `3.0.1`

## Contributing

- Build and install the jar with `boot build-jar`
- Start a repl with `boot repl`
- Run tests with `boot test!`
- Deploy a release with `boot build-jar push-release`

PRs and issues welcome :tada:

**Note:** Because this library uses it's own version in various
places the version is stored on the classpath. In order to modify
the version you need to modify `resources/raven_clj/version.txt`.

## License

Copyright © 2013-2015 Seth Buntin

Distributed under the Eclipse Public License, the same as Clojure.
