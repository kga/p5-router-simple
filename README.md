# NAME

Router::Simple - simple HTTP router

# SYNOPSIS

    use Router::Simple;

    my $router = Router::Simple->new();
    $router->connect('/', {controller => 'Root', action => 'show'});
    $router->connect('/blog/{year}/{month}', {controller => 'Blog', action => 'monthly'});

    my $app = sub {
        my $env = shift;
        if (my $p = $router->match($env)) {
            # $p = { controller => 'Blog', action => 'monthly', ... }
        } else {
            [404, [], ['not found']];
        }
    };

# DESCRIPTION

Router::Simple is a simple router class.

Its main purpose is to serve as a dispatcher for web applications.

Router::Simple can match against PSGI `$env` directly, which means
it's easy to use with PSGI supporting web frameworks.

# HOW TO WRITE A ROUTING RULE

## plain string

    $router->connect( '/foo', { controller => 'Root', action => 'foo' } );

## :name notation

    $router->connect( '/wiki/:page', { controller => 'WikiPage', action => 'show' } );
    ...
    $router->match('/wiki/john');
    # => {controller => 'WikiPage', action => 'show', page => 'john' }

':name' notation matches `qr{([^/]+)}`.

## '\*' notation

    $router->connect( '/download/*.*', { controller => 'Download', action => 'file' } );
    ...
    $router->match('/download/path/to/file.xml');
    # => {controller => 'Download', action => 'file', splat => ['path/to/file', 'xml'] }

'\*' notation matches `qr{(.+)}`. You will get the captured argument as
an array ref for the special key `splat`.

## '{year}' notation

    $router->connect( '/blog/{year}', { controller => 'Blog', action => 'yearly' } );
    ...
    $router->match('/blog/2010');
    # => {controller => 'Blog', action => 'yearly', year => 2010 }

'{year}' notation matches `qr{([^/]+)}`, and it will be captured.

## '{year:\[0-9\]+}' notation

    $router->connect( '/blog/{year:[0-9]+}/{month:[0-9]{2}}', { controller => 'Blog', action => 'monthly' } );
    ...
    $router->match('/blog/2010/04');
    # => {controller => 'Blog', action => 'monthly', year => 2010, month => '04' }

You can specify regular expressions in named captures.

## regexp

    $router->connect( qr{/blog/(\d+)/([0-9]{2})', { controller => 'Blog', action => 'monthly' } );
    ...
    $router->match('/blog/2010/04');
    # => {controller => 'Blog', action => 'monthly', splat => [2010, '04'] }

You can use Perl5's powerful regexp directly, and the captured values
are stored in the special key `splat`.

# METHODS

- my $router = Router::Simple->new();

    Creates a new instance of Router::Simple.

- $router->method\_not\_allowed() : Boolean

    This method returns last `$router->match()` call is rejected by HTTP method or not.

- $router->connect(\[$name, \] $pattern, \\%destination\[, \\%options\])

    Adds a new rule to $router.

        $router->connect( '/', { controller => 'Root', action => 'index' } );
        $router->connect( 'show_entry', '/blog/:id',
            { controller => 'Blog', action => 'show' } );
        $router->connect( '/blog/:id', { controller => 'Blog', action => 'show' } );
        $router->connect( '/comment', { controller => 'Comment', action => 'new_comment' }, {method => 'POST'} );

    `\%destination` will be used by _match_ method.

    You can specify some optional things to `\%options`. The current
    version supports 'method', 'host', and 'on\_match'.

    - method

        'method' is an ArrayRef\[String\] or String that matches **REQUEST\_METHOD** in $req.

    - host

        'host' is a String or Regexp that matches **HTTP\_HOST** in $req.

    - on\_match

            $r->connect(
                '/{controller}/{action}/{id}',
                {},
                {
                    on_match => sub {
                        my($env, $match) = @_;
                        $match->{referer} = $env->{HTTP_REFERER};
                        return 1;
                    }
                }
            );

        A function that evaluates the request. Its signature must be `($environ, $match) => bool`. It should return true if the match is
        successful or false otherwise. The first argument is `$env` which is
        either a PSGI environment or a request path, depending on what you
        pass to `match` method; the second is the routing variables that
        would be returned if the match succeeds.

        The function can modify `$env` (in case it's a reference) and
        `$match` in place to affect which variables are returned. This allows
        a wide range of transformations.

- `$router->submapper($path, [\%dest, [\%opt]])`

        $router->submapper('/entry/', {controller => 'Entry'})

    This method is shorthand for creating new instance of [Router::Simple::Submapper](https://metacpan.org/pod/Router::Simple::Submapper).

    The arguments will be passed to `Router::Simple::SubMapper->new(%args)`.

- `$match = $router->match($env|$path)`

    Matches a URL against one of the contained routes.

    The parameter is either a [PSGI](https://metacpan.org/pod/PSGI) $env or a plain string that
    represents a path.

    This method returns a plain hashref that would look like:

        {
            controller => 'Blog',
            action     => 'daily',
            year => 2010, month => '03', day => '04',
        }

    It returns undef if no valid match is found.

- `my ($match, $route) = $router->routematch($env|$path);`

    Match a URL against one of the routes contained.

    Will return undef if no valid match is found, otherwise a
    result hashref and a [Router::Simple::Route](https://metacpan.org/pod/Router::Simple::Route) object is returned.

- `$router->as_string()`

    Dumps $router as string.

    Example output:

        home         GET  /
        blog_monthly GET  /blog/{year}/{month}
                     GET  /blog/{year:\d{1,4}}/{month:\d{2}}/{day:\d\d}
                     POST /comment
                     GET  /

# AUTHOR

Tokuhiro Matsuno <tokuhirom AAJKLFJEF@ GMAIL COM>

# THANKS TO

Tatsuhiko Miyagawa

Shawn M Moore

[routes.py](http://routes.groovie.org/).

# SEE ALSO

Router::Simple is inspired by [routes.py](http://routes.groovie.org/).

[Path::Dispatcher](https://metacpan.org/pod/Path::Dispatcher) is similar, but so complex.

[Path::Router](https://metacpan.org/pod/Path::Router) is heavy. It depends on [Moose](https://metacpan.org/pod/Moose).

[HTTP::Router](https://metacpan.org/pod/HTTP::Router) has many dependencies. It is not well documented.

[HTTPx::Dispatcher](https://metacpan.org/pod/HTTPx::Dispatcher) is my old one. It does not provide an OO-ish interface.

# THANKS TO

DeNA

# LICENSE

Copyright (C) Tokuhiro Matsuno

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.
