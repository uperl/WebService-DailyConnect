# WebService::DailyConnect ![linux](https://github.com/uperl/WebService-DailyConnect/workflows/linux/badge.svg)

Web client to download events from Daily Connect

# SYNOPSIS

```perl
use WebService::DailyConnect;
use Term::Clui qw( ask ask_password );
use Path::Tiny qw( path );

my $user = ask("email:");
my $pass = ask_password("pass :");

my $dc = WebService::DailyConnect->new;
$dc->login($user, $pass) || die "bad email/pass";

my $user_info = $dc->user_info;

foreach my $kid (@{ $dc->user_info->{myKids} })
{
  my $kid_id = $kid->{Id};
  my $name   = lc $kid->{Name};
  foreach my $photo_id (map { $_->{Photo} || () } @{ $dc->kid_status($kid_id)->{list} })
  {
    my $dest = path("~/Pictures/dc/$name-$photo_id.jpg");
    next if -f $dest;
    print "new photo: $dest\n";
    $dest->parent->mkpath;
    $dc->photo($photo_id, $dest);
  }
}
```

# DESCRIPTION

**NOTE**: I no longer use DailyConnect, and happy to let someone who does need it
maintain it.  This module is otherwise unsupported.

Interface to DailyConnect, which is a service that can provide information about
your kids at daycare.  This is more or less a port of a node API that I found here:

[https://github.com/Flet/dailyconnect](https://github.com/Flet/dailyconnect)

I wrote this module for more or less the same reasons as that author, although I
wanted to be able to use it in perl.

It uses [HTTP::AnyUA](https://metacpan.org/pod/HTTP::AnyUA), so should work with any Perl user agent supported by that
layer.

# ATTRIBUTES

## ua

An instance of [HTTP::AnyUA](https://metacpan.org/pod/HTTP::AnyUA).  If a user agent supported by [HTTP::AnyUA](https://metacpan.org/pod/HTTP::AnyUA)
(such as [HTTP::Tiny](https://metacpan.org/pod/HTTP::Tiny) or [LWP::UserAgent](https://metacpan.org/pod/LWP::UserAgent)) is provided, a new instance of
[HTTP::AnyUA](https://metacpan.org/pod/HTTP::AnyUA) will wrap around that user agent instance.  The only requirement
is that the underlying user agent must support cookies.

If a `ua` attribute is not provided, then an instance of [HTTP::AnyUA](https://metacpan.org/pod/HTTP::AnyUA) will
be created wrapped around a [LWP::UserAgent](https://metacpan.org/pod/LWP::UserAgent) using the default proxy and a
cookie jar.

## base\_url

The base URL for daily connect.  The default should be correct.

## req

The most recent request.  The format of the request object is subject to change, and therefore should only be used for debugging.

## res

The most recent response.  The format of the response object is subject to change, and therefore should only be used for debugging.

## METHODS

Beside login, methods typically return a hash or file content depending on the type of object requested.
On error they return `undef`.  Further details for the failure can be deduced from the response object
stored in `res` above.

## login

```perl
my $bool = $dc->login($email, $pass);
```

Login to daily connect using the given email and password.  The remaining methods only work once you have successfully logged in.

## user\_info

```perl
my $hash = $dc->user_info;
```

Get a hash of the user information.

## kid\_summary

```perl
my $hash = $dc->kid_summary($kid_id);
```

Get today's summary for the given kid.

## kid\_summary\_by\_date

```perl
my $hash = $dc->kid_summary_by_date($kid_id, $date);
```

Get the summary for the given kid on the given day.  `$date` is in the form YYMMDD.

## kid\_status

```perl
my $hash = $dc->kid_status($kid_id);
```

Get today's status for the given kid.

## kid\_status\_by\_date

```perl
my $hash = $dc->kid_status_by_date($kid_id, $date);
```

Get the status for the given kid on the given date.  `$date` is in the form YYMMDD.

## photo

```
$dc->photo($photo_id);
$dc->photo($photo_id, $dest);
```

Get the photo with the given `$photo_id`.  If `$dest` is not provided then the content of the photo in
JPEG format will be returned.  If `$dest` is a scalar reference, then the content will be stored in that
scalar.  If `$dest` is a string, then that string will be assumed to be a file, and the photo will be saved
there.  If a [Path::Tiny](https://metacpan.org/pod/Path::Tiny) or [Path::Class::File](https://metacpan.org/pod/Path::Class::File) object are passed in, then the content will be written
to the files at that location.

# CAVEATS

DailyConnect does not provide a standard RESTful API, so it is entirely possible
they might change the interface of their app, and break this module.

My kiddo is an only child so I haven't been able to test this for more than one
kiddo.  May be a problem if you have twins or septuplets.

# AUTHOR

Graham Ollis <plicease@cpan.org>

# COPYRIGHT AND LICENSE

This software is copyright (c) 2018-2024 by Graham Ollis.

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.
