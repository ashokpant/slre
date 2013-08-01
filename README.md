SLRE: Super Light Regular Expression library
============================================

SLRE is an ISO C library that implements a subset of Perl regular
expression syntax. Main features of SLRE are:

   * Written in strict ISO C, conforming to ANSI C'89
   * Small size (compiled x86 code is about 4kB)
   * Uses no dynamic memory allocation and small stack space
   * [Simple API](https://github.com/cesanta/slre/blob/master/slre.h)
   * Implements most useful subset of Perl regex syntax (see below)
   * Easily extensible. For example, if one wants to introduce a new
metacharacter `\i`, meaning "IPv4 address", it is easy to do so with SLRE.

SLRE is perfect for tasks like parsing network requests, configuration
files, user input, etc, when libraries like [PCRE](http://pcre.org) are too
heavyweight for the given task. Developers of embedded systems would benefit
most.

## Supported Syntax

    ^        Match beginning of a buffer
    $        Match end of a buffer
    ()       Grouping and substring capturing
    \s       Match whitespace
    \S       Match non-whitespace
    \d       Match decimal digit
    +        Match one or more times (greedy)
    +?       Match one or more times (non-greedy)
    *        Match zero or more times (greedy)
    *?       Match zero or more times (non-greedy)
    ?        Match zero or once
    x|y      Match x or y (alternation operator)
    \meta    Match one of the meta character: ^$().[]*+?|\

Not supported but in progress:

    [...]    Match any character from set
    [^...]   Match any character but ones from set
    \xDD     Match byte with hex value 0xDD

## API

    int slre_match(const char *regexp, const char *buf, int buf_len,
                   struct slre_cap *caps, const char **error_msg);


`slre_match()` matches string buffer `buf` of length `buf_len` against
regular expression `regexp`, which should conform the syntax outlined
above. If regular expression `regexp` contains brackets, `slre_match()`
will capture the respective substrings. Array of captures, `caps`,
must have at least as many elements as number of bracket pairs in the `regexp`.

`slre_match()` returns 0 if there is no match found. Otherwise, it returns
the number scanned bytes from the beginning of the string. This way,
it is easy to do repetitive matches. Hint: if it is required to know
the exact matched substring, enclose `regexp` in a brackets and specify `caps`,
which should be an array of following structures:

    struct slre_cap {
      const char *ptr;  /* Points to the matched fragment */
      int len;          /* Length of the matched fragment */
    };

## Example: parsing HTTP request

    const char *error_msg, *request = " GET /index.html HTTP/1.0\r\n\r\n";
    struct slre_cap caps[4];

    if (slre_match("^\\s*(\\S+)\\s+(\\S+)\\s+HTTP/(\\d)\\.(\\d)",
                   request, strlen(request), caps, &error_msg)) {
      printf("Method: [%.*s], URI: [%.*s]\n",
             caps[0].len, caps[0].ptr,
             caps[1].len, caps[1].ptr);
    } else {
      printf("Error parsing [%s]: [%s]\n", request, error_msg);
    }

# Licensing

SLRE is dual licensed. It is available either under the terms of [GNU GPL
v.2 license](http://www.gnu.org/licenses/old-licenses/gpl-2.0.html) for
free, or under the terms of standard commercial license provided by [Cesanta
Software](http://cesanta.com). Businesses who whish to use Cesanta's products
must [license commercial version](http://cesanta.com/products.html).
