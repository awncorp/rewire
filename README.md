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

# LIBRARIES

This package uses type constraints from:

[Types::Standard](https://metacpan.org/pod/Types%3A%3AStandard)

# SCENARIOS

This package supports the following scenarios:

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
