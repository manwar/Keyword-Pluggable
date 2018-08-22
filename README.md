# NAME

Keyword::Pluggable - define new keywords in pure Perl

# SYNOPSIS

    package Some::Module;
    
    use Keyword::Pluggable;
    
    sub import {
        # create keyword 'provided', expand it to 'if' at parse time
        Keyword::Pluggable::define 'provided', sub {
            my ($ref) = @_;
            substr($$ref, 0, 0) = 'if';  # inject 'if' at beginning of parse buffer
        };
    }
    
    sub unimport {
        # lexically disable keyword again
        Keyword::Pluggable::undefine 'provided';
    }

    'ok'

# DESCRIPTION

Warning: This module is still new and experimental. The API may change in
future versions. The code may be buggy. Also, this module is a fork from
`Keyword::Simple`, that somehow got stalled. If its author accepts pull requests,
then it will probably be best to use it instead.

This module lets you implement new keywords in pure Perl. To do this, you need
to write a module and call
[`Keyword::Pluggable::define`](#keyword-pluggable-define) in your `import`
method. Any keywords defined this way will be available in the lexical scope
that's currently being compiled.

## Functions

- `Keyword::Pluggable::define`

    Takes three arguments, the name of a keyword, a coderef, and a boolean flag if
    the result of the keyword handler is an expression. Injects the keyword
    in the lexical scope currently being compiled. For every occurrence of the
    keyword, your coderef will be called with one argument: A reference to a scalar
    holding the rest of the source code (following the keyword).

    You can modify this scalar in any way you like and after your coderef returns,
    perl will continue parsing from that scalar as if its contents had been the
    real source code in the first place.

- `Keyword::Pluggable::undefine`

    Takes one argument, the name of a keyword. Disables that keyword in the lexical
    scope that's currently being compiled. You can call this from your `unimport`
    method to make the `no Foo;` syntax work.

# BUGS AND LIMITATIONS

This module depends on the [pluggable keyword](https://metacpan.org/pod/perlapi.html#PL_keyword_plugin)
API introduced in perl 5.12. `parse_` functions were introduced in 5.14.
Older versions of perl are not supported.

Every new keyword is actually a complete statement or an expression by itself. The parsing magic
only happens afterwards. This means that e.g. the code in the ["SYNOPSIS"](#synopsis)
actually does this:

    provided ($foo > 2) {
      ...
    }

    # expands to

    ; if
    ($foo > 2) {
      ...
    }

The `;` represents a no-op statement, the `if` was injected by the Perl code,
and the rest of the file is unchanged.

This also means your new keywords can only occur at the beginning of a
statement, not embedded in an expression.

Keywords in the replacement part of a `s//.../e` substitution aren't handled
correctly and break parsing.

There are barely any tests.

# AUTHOR

Lukas Mai, `<l.mai at web.de>`
Dmitry Karasik , `<dmitry at karasik.eu.org`

# COPYRIGHT & LICENSE

Copyright (C) 2012, 2013 Lukas Mai.
Copyright (C) 2018 Dmitry Karasik

This program is free software; you can redistribute it and/or modify it
under the terms of either: the GNU General Public License as published
by the Free Software Foundation; or the Artistic License.

See http://dev.perl.org/licenses/ for more information.