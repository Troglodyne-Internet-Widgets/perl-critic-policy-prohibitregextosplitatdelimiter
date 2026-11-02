# NAME

Perl::Critic::Policy::RegularExpressions::ProhibitRegexToSplitAtDelimiter - Take the text before a literal separator with index and substr, not a regex capture.

# VERSION

version 0.001

# Perl::Critic::Policy::RegularExpressions::ProhibitRegexToSplitAtDelimiter

A regex that captures everything from the start of a string up to the first
occurrence of a fixed separator is finding that separator and taking what is in
front of it.  `index` finds the separator, and `substr` takes the part before
it.  They say what the code does, and they do it without compiling a pattern
and running it:

```perl
while ( $buffer =~ s/\A([^\n]*)\n// ) { take($1) }            # reported

while ( ( my $at = index $buffer, "\n" ) >= 0 ) {
    take( substr $buffer, 0, $at );
    substr $buffer, 0, $at + 1, q{};
}

my ($key) = $line =~ m/\A(.*?)=/s;                             # reported

my $at  = index $line, '=';
my $key = $at >= 0 ? substr( $line, 0, $at ) : undef;
```

The substitution takes the part before the separator off the string, together
with the separator.  Four-argument `substr` with an empty replacement does
that.

## PROHIBITED

```
s/\A([^\n]*)\n//          # the negated class of the separator
m/\A([^=]+)=/             # one or more, see CAVEATS
m/^([^:]*):/              # ^ without /m is the start of the string
m/\A([^:]*):/m            # /m changes ^ and $, not \A
m/\A(.*?)--/s             # a lazy dot under /s, before a string
m/\A(.+?), /s             # one or more, see CAVEATS
m/\A(.*?)\n/              # without /s, . is anything but a newline
m/\A(.*)\n/               # so before a newline it can be greedy too
m/\A(\N*)\n/              # and so can \N
s/\A([^\n]*)\n//r         # /r: a copy
m/\A(?<key>[^:]*):/       # a named capture
s/\A([^\n]*)\x0a//        # any spelling of the character
s/\A([^\n]*)[\n]//        # a class of one character is that character
m/\A([^.]*)\./            # an escaped metacharacter is a character
m/\A(.*?)\Q-.\E/s         # and so is anything between \Q and \E
m/\A ( [^:]* ) : /x       # whitespace under /x is not text
```

## ALLOWED

```perl
m/\A(.*?):/               # without /s, . refuses a newline before the :
m/\A(.*):/s               # greedy: the last separator, which is rindex
m/^([^:]*):/m             # /m: ^ is the start of any line
m/([^:]*):/               # unanchored
m/\G([^:]*):/gc           # \G is where the last match stopped
m/\A([^:]*):/g            # /g
s/\A([^\n]*)\n//g         # /g
m/\A([^x]*)x/i            # /i: index compares case
m/\A([^:;]*)[:;]/         # a class of several characters
m/\A([^a]*)ab/            # the first a, which must be followed by b
m/\A(.*?)\s/s             # \s, \R and the like are not literal text
m/\A(.*?)(?:,|;)/s        # alternation
m/\A(.*?)$sep/s           # interpolation
s/\A([^\n]*)\n+//         # a quantified separator
m/\A([^:]{2,}):/          # a counted quantifier
m/\A(\w*):/               # a capture that checks what it captures
m/\A([^:]*):(.*)\z/s      # two captures
s/\A([^:]*):\s*//         # more pattern after the separator
m/\A([^\n]*)\n\z/         # an anchor at the end as well
s/\A[^\n]*\n//            # no capture
s/\A([^\n]*)\n/$1 /       # a replacement
qr/\A([^\n]*)\n/          # a compiled regex, which may be used elsewhere
```

## CAVEATS

`[^X]+` and `.+?` refuse a string that starts with the separator, and
`index` finds it at position 0.  When that check is the point, keep the regex
and say `## no critic (ProhibitRegexToSplitAtDelimiter)` and why, or test
`$at > 0` instead of `$at >= 0`.

`index` returns -1 when the separator is not there.  Test the position before
you take `substr` of it: `substr( $line, 0, -1 )` is the whole string less its
last character, not an error.

Modifiers turned on by `use re` in the lexical scope of the regex count as if
they were written on it.  So under `use re '/s'`, `m/\A(.*?):/` is reported,
and under `use re '/m'`, `m/^([^:]*):/` is not.

## METHODS

### supported\_parameters

### default\_severity

### default\_themes

### applies\_to

### violates

# AUTHORS

Current Maintainers:

- George S. Baugh <george@troglodyne.net>

# COPYRIGHT AND LICENSE

Copyright (c) 2026 Troglodyne LLC

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
