# NAME

Plack::App::EventSource - EventSource/SSE for Plack

# SYNOPSIS

    use Plack::App::EventSource;
    use Plack::Builder;

    builder {
        mount '/events' => Plack::App::EventSource->new(
            handler_cb => sub {
                my ($conn, $env) = @_;

                $conn->push('foo');
                # or
                # $conn->push('foo', 'bar', 'baz');
                # or
                # $conn->push({id => 1, data => 'foo'});
                $conn->close;
            }
        )->to_app;

        mount '/' => $app;
    };

# DESCRIPTION

Plack::App::EventSource is an EventSource or Server Sent Events applications.
EventSource is an alternative to WebSockets when there is no need for duplex
communication. EventSource uses HTTP and is much simpler in implementation.
Ideal for website notifications or read only update streams.

It is recommended to use EventSource with polyfills to enable them in browsers
that don't support SSE. This does not need any server changes, which is very
handy. Take a look at
[EventSource.js](https://github.com/remy/polyfills/blob/master/EventSource.js)
and [jquery.eventsource](https://github.com/rwaldron/jquery.eventsource).

This library stays event loop agnostic, which means that you can use it with
[AnyEvent](https://metacpan.org/pod/AnyEvent) or [POE](https://metacpan.org/pod/POE) or even just with a plain forking server.

## Options

- `handler_cb`

    The main application entry point. It is called with
    [Plack::App::EventSource::Connection](https://metacpan.org/pod/Plack::App::EventSource::Connection) and `$env` parameters.

        handler_cb => sub {
            my ($conn, $env) = @_;

            $conn->push('hi');
            $conn->close;
        }

- `headers`

    Additional response headers. This is useful when you want to add Access Control
    headers:

        headers => [
            'Access-Control-Allow-Origin' : 'http://localhost:5000',
            'Access-Control-Allow-Credentials' : 'true'
        ]

# HOWTOs

## Sending cookies to another domain

Set CORS headers:

    headers => [
        'Access-Control-Allow-Origin' : 'http://original-domain',
        'Access-Control-Allow-Credentials' : 'true'
    ]

If you try to set `Origin` to `*` some browsers will complain. So make sure to
set the correct original domain.

When connecting to EventSource on the client side pass `withCredentials`
option:

    var es = new EventSource("http://another-domain", {
        withCredentials: true
    });

## Nginx proxy

    location /events {
            proxy_pass http://backend;
            proxy_buffering off;
            proxy_cache off;
            proxy_set_header Host $host;
            proxy_set_header Connection '';
            proxy_http_version 1.1;
            chunked_transfer_encoding off;
    }

# ISA

[Plack::Component](https://metacpan.org/pod/Plack::Component)

# METHODS

## `call($env)`

# INHERITED METHODS

## `new`

## `mk_accessors`

## `prepare_app`

## `response_cb($res, $cb)`

## `to_app`

## `to_app_auto`

# AUTHOR

Viacheslav Tykhanovskyi, <viacheslav.t@gmail.com>

# COPYRIGHT AND LICENSE

Copyright (C) 2015, Viacheslav Tykhanovskyi

This program is free software, you can redistribute it and/or modify it under
the terms of the Artistic License version 2.0.

This program is distributed in the hope that it will be useful, but without any
warranty; without even the implied warranty of merchantability or fitness for
a particular purpose.
