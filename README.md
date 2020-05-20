# NAME

Rewire - Dependency Injection

# ABSTRACT

Dependency Injection Container for Perl 5

# SYNOPSIS

    use Rewire;

    my $services = {
      filetemp => {
        package => 'File/Temp'
      },
      tempfile => {
        package => 'Mojo/File',
        argument => { '$service' => 'filetemp' }
      }
    };

    my $rewire = Rewire->new(services => $services);

    $rewire->resolve('tempfile');

# DESCRIPTION

This package provides methods for using dependency injection, and building
objects and values.

# INTEGRATES

This package integrates behaviors from:

[Data::Object::Role::Buildable](https://metacpan.org/pod/Data%3A%3AObject%3A%3ARole%3A%3ABuildable)

[Data::Object::Role::Proxyable](https://metacpan.org/pod/Data%3A%3AObject%3A%3ARole%3A%3AProxyable)

# LIBRARIES

This package uses type constraints from:

[Types::Standard](https://metacpan.org/pod/Types%3A%3AStandard)

# SCENARIOS

This package supports the following scenarios:

## $envvar

    use Rewire;

    my $services = {
      file => {
        package => 'Mojo/File',
        argument => { '$envvar' => 'home' }
      }
    };

    my $rewire = Rewire->new(
      services => $services
    );

This package supports inlining environment variables as arguments to services.
The `$envvar` directive is used to specify the name of an environment
variable, and can also be used in metadata for reusability.

## $function

    use Rewire;

    my $services = {
      temp => {
        package => 'File/Temp'
      },
      file => {
        package => 'Mojo/File',
        argument => { '$function' => 'temp#tempfile' }
      }
    };

    my $rewire = Rewire->new(
      services => $services
    );

This package supports inlining the result of a service resolution and function
call as arguments to services. The `#` delimited `$function` directive is
used to specify the name of an existing service on the right-hand side, and an
arbitrary function to be call on the result on the left-hand side.

## $metadata

    use Rewire;

    my $metadata = {
      home => '/home/ubuntu'
    };

    my $services = {
      file => {
        package => 'Mojo/File',
        argument => { '$metadata' => 'home' }
      }
    };

    my $rewire = Rewire->new(
      metadata => $metadata,
      services => $services
    );

This package supports inlining configuration data as arguments to services.
The `$metadata` directive is used to specify the name of a stashed
configuration value or data structure.

## $method

    use Rewire;

    my $services = {
      temp => {
        package => 'File/Temp'
      },
      file => {
        package => 'Mojo/File',
        argument => { '$method' => 'temp#filename' }
      }
    };

    my $rewire = Rewire->new(
      services => $services
    );

This package supports inlining the result of a service resolution and method
call as arguments to services. The `#` delimited `$method` directive is used
to specify the name of an existing service on the right-hand side, and an
arbitrary method to be call on the result on the left-hand side.

## $routine

    use Rewire;

    my $services = {
      temp => {
        package => 'File/Temp'
      },
      file => {
        package => 'Mojo/File',
        argument => { '$routine' => 'temp#tempfile' }
      }
    };

    my $rewire = Rewire->new(
      services => $services
    );

This package supports inlining the result of a service resolution and routine
call as arguments to services. The `#` delimited `$routine` directive is
used to specify the name of an existing service on the right-hand side, and an
arbitrary routine to be call on the result on the left-hand side.

## $service

    use Rewire;

    my $services = {
      io => {
        package => 'IO/Handle'
      },
      log => {
        package => 'Mojo/Log',
        argument => {
          handle => { '$service' => 'io' }
        }
      },
    };

    my $rewire = Rewire->new(
      services => $services
    );

This package supports inlining resolved services as arguments to other
services. The `$service` directive is used to specify the name of a service
to be resolved and passed as an argument.

## arguments

    use Rewire;

    my $metadata = {
      applog => '/var/log/rewire.log'
    };

    my $services = {
      mojo_log => {
        package => 'Mojo/Log',
        argument => {
          path => { '$metadata' => 'applog' },
          level => 'warn'
        },
        argument_as => 'list'
      }
    };

    my $rewire = Rewire->new(
      services => $services,
      metadata => $metadata
    );

This package supports providing static and/or dynamic arguments during object
construction from `metadata` or other `services`.

## builder

    use Rewire;

    my $services = {
      mojo_date => {
        package => 'Mojo/Date',
        builder => [
          {
            method => 'new',
            return => 'self'
          },
          {
            method => 'to_datetime',
            return => 'result'
          }
        ]
      }
    };

    my $rewire = Rewire->new(
      services => $services,
    );

This package supports specifying multiple build steps as `function`,
`method`, and `routine` calls and chaining them together.

## config

    use Rewire;

    my $metadata = {
      home => '/home/ubuntu'
    };

    my $services = {
      tempfile => {
        package => 'Mojo/File',
        argument => { '$metadata' => 'home' }
      }
    };

    my $rewire = Rewire->new(
      services => $services,
      metadata => $metadata
    );

This package supports configuring services and metadata in the service of
building objects and values.

## constructor

    use Rewire;

    my $services = {
      mojo_date => {
        package => 'Mojo/Date',
        constructor => 'new'
      }
    };

    my $rewire = Rewire->new(
      services => $services
    );

This package supports specifying constructors other than the traditional `new`
routine. A constructor is always called with the package name as the invocant.

## extends

    use Rewire;

    my $services = {
      io => {
        package => 'IO/Handle'
      },
      log => {
        package => 'Mojo/Log',
        argument => {
          handle => { '$service' => 'io' }
        }
      },
      development_log => {
        package => 'Mojo/Log',
        extends => 'log',
        builder => [
          {
            method => 'path',
            argument => '/tmp/development.log',
            return => 'none'
          },
          {
            method => 'level',
            argument => 'debug',
            return => 'none'
          }
        ]
      },
      production_log => {
        package => 'Mojo/Log',
        extends => 'log',
        builder => [
          {
            method => 'path',
            argument => '/tmp/production.log',
            return => 'none'
          },
          {
            method => 'level',
            argument => 'warn',
            return => 'none'
          }
        ]
      },
    };

    my $rewire = Rewire->new(
      services => $services
    );

This package supports extending services in the definition of other services,
effectively using the extended service as the invocant in the creation of the
requested service.

## function

    use Rewire;

    my $services = {
      foo_sum => {
        package => 'Mojo/Util',
        function => 'md5_sum',
        argument => 'foo',
      }
    };

    my $rewire = Rewire->new(
      services => $services,
    );

This package supports specifying construction as a function call, which when
called does not provide an invocant.

## lifecycle

    use Rewire;

    my $metadata = {
      home => '/home/ubuntu'
    };

    my $services = {
      tempfile => {
        package => 'Mojo/File',
        argument => { '$metadata' => 'home' },
        lifecycle => 'singleton'
      }
    };

    my $rewire = Rewire->new(
      services => $services,
      metadata => $metadata
    );

This package supports different lifecycle options which determine when services
are built and whether they're persisted.

## metadata

    use Rewire;

    my $metadata = {
      homedir => '/home',
      tempdir => '/tmp'
    };

    my $services = {
      home => {
        package => 'Mojo/Path',
        argument => { '$metadata' => 'homedir' },
      },
      temp => {
        package => 'Mojo/Path',
        argument => { '$metadata' => 'tempdir' },
      }
    };

    my $rewire = Rewire->new(
      services => $services,
      metadata => $metadata
    );

This package supports specifying data and structures which can be used in the
construction of multiple services.

## method

    use Rewire;

    my $services = {
      mojo_url => {
        package => 'Mojo/URL',
        argument => 'https://perl.org',
        method => 'new'
      }
    };

    my $rewire = Rewire->new(
      services => $services,
    );

This package supports specifying construction as a method call, which when
called provides the package or object instance as the invocant.

## proxyable

    use Rewire;

    my $services = {
      home => {
        package => 'Mojo/Path',
        argument => '/home',
      },
      temp => {
        package => 'Mojo/Path',
        argument => '/tmp',
      }
    };

    my $rewire = Rewire->new(
      services => $services
    );

    # resolve services via method calls
    [
      $rewire->home, # i.e. $rewire->process('home')
      $rewire->temp  # i.e. $rewire->process('temp')
    ]

This package supports the resolution of services using a single method call.
This is enabled by intercepting method calls and proxying them to the
["process"](#process) method.

## routine

    use Rewire;

    my $services = {
      mojo_url => {
        package => 'Mojo/URL',
        argument => 'https://perl.org',
        routine => 'new'
      }
    };

    my $rewire = Rewire->new(
      services => $services,
    );

This package supports specifying construction as a function call, which when
called provides the package as the invocant.

## service

    my $metadata = {
      home => '/home/ubuntu'
    };

    my $services = {
      tempfile => {
        package => 'Mojo/File',
        argument => { '$metadata' => 'home' },
        lifecycle => 'eager'
      }
    };

    my $rewire = Rewire->new(
      services => $services,
      metadata => $metadata
    );

This package supports defining services to be constructed on-demand or
automatically on instantiation.

# ATTRIBUTES

This package has the following attributes:

## context

    context(CodeRef)

This attribute is read-only, accepts `(CodeRef)` values, and is optional.

## engine

    engine(InstanceOf["Data::Object::Space"])

This attribute is read-only, accepts `(InstanceOf["Data::Object::Space"])` values, and is optional.

## metadata

    metadata(HashRef)

This attribute is read-only, accepts `(HashRef)` values, and is optional.

## services

    services(HashRef)

This attribute is read-only, accepts `(HashRef)` values, and is optional.

# METHODS

This package implements the following methods:

## config

    config() : HashRef

The config method returns the configuration based on the `services` and
`metadata` attributes.

- config example #1

        # given: synopsis

        $rewire->config;

## process

    process(Str $name, Any $argument, Maybe[Str] $argument_as) : Any

The process method processes and returns an object or value based on the
service named but where the arguments are provided ad-hoc. **Note:** This method
is meant to be used to construct services ad-hoc and as such bypasses caching
and lifecycle effects.

- process example #1

        # given: synopsis

        $rewire->process('tempfile', 'rewire.tmp');

- process example #2

        use Rewire;

        my $metadata = {
          logfile => '/var/log/rewire.log',
        };

        my $services = {
          mojo_log => {
            package => 'Mojo/Log',
            argument => { '$metadata' => 'logfile' },
          }
        };

        my $rewire = Rewire->new(
          services => $services,
          metadata => $metadata
        );

        $rewire->process('mojo_log', {
          level => 'fatal',
          path => { '$metadata' => 'logfile' }
        });

- process example #3

        use Rewire;

        my $metadata = {
          logfile => '/var/log/rewire.log',
        };

        my $services = {
          mojo_log => {
            package => 'Mojo/Log',
            builder => [
              {
                method => 'new',
                return => 'self'
              }
            ]
          }
        };

        my $rewire = Rewire->new(
          services => $services,
          metadata => $metadata
        );

        $rewire->process('mojo_log', {
          level => 'fatal',
          path => { '$metadata' => 'logfile' }
        });

## resolve

    resolve(Str $name) : Any

The resolve method resolves and returns an object or value based on the service
named. **Note:** This method is recommended to be used to construct services as
defined by the configuration and as such doesn't not allow passing additional
arguments.

- resolve example #1

        # given: synopsis

        $rewire->resolve('tempfile');

- resolve example #2

        use Rewire;

        my $services = {
          mojo_log => {
            package => 'Mojo/Log',
            argument => {
              level => 'fatal',
              path => '/var/log/rewire.log'
            },
          }
        };

        my $rewire = Rewire->new(
          services => $services,
        );

        $rewire->resolve('mojo_log');

- resolve example #3

        package Dynamic;

        sub import;

        sub AUTOLOAD {
          bless {};
        }

        sub DESTROY {
          ; # noop
        }

        package main;

        use Rewire;

        my $services = {
          dynamic => {
            package => 'Dynamic',
            builder => [
              {
                method => 'new',
                return => 'self'
              },
              {
                method => 'missing_method',
                return => 'result'
              }
            ],
          }
        };

        my $rewire = Rewire->new(
          services => $services,
        );

        $rewire->resolve('dynamic');

## validate

    validate() : Object

The validate method validates the configuration and throws an exception if
invalid, otherwise returns itself.

- validate example #1

        # given: synopsis

        $rewire->validate;

# AUTHOR

Al Newkirk, `awncorp@cpan.org`

# LICENSE

Copyright (C) 2011-2019, Al Newkirk, et al.

This is free software; you can redistribute it and/or modify it under the terms
of the The Apache License, Version 2.0, as elucidated in the ["license
file"](https://github.com/iamalnewkirk/rewire/blob/master/LICENSE).

# PROJECT

[Wiki](https://github.com/iamalnewkirk/rewire/wiki)

[Project](https://github.com/iamalnewkirk/rewire)

[Initiatives](https://github.com/iamalnewkirk/rewire/projects)

[Milestones](https://github.com/iamalnewkirk/rewire/milestones)

[Contributing](https://github.com/iamalnewkirk/rewire/blob/master/CONTRIBUTE.md)

[Issues](https://github.com/iamalnewkirk/rewire/issues)
