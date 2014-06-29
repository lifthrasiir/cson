CSON
====

**CSON**(Cursive Script Object Notation) is
a strict superset of [JavaScript Object Notation][json](JSON)
that can be written by hand (hence the name)
and translated to a canonical JSON.

[json]: http://json.org/

Among other machine-readable semi-structured data formats,
CSON has many benefits:

* Every CSON data can be translated to JSON back and forth,
  so you can continue using the existing library
  that only understands JSON.
* CSON is a strict superset of JSON,
  so you don't have to convert existing JSON data to CSON.
* Valid JSON fragments can be used anywhere in the CSON data,
  unlike several configuration file formats.
* CSON is not whitespace-sensitive
  but still encourages writers to put the proper indentation.
* [You can use it for evil!][crockford-on-evil]

[crockford-on-evil]: https://en.wikipedia.org/wiki/Douglas_Crockford#Criticism

CSON is designed by [Kang Seonghoon][kang-seonghoon].
While the core principle of CSON is set in stone,
please note that this is not the final specification
and details may change without a notice.

[kang-seonghoon]: http://mearie.org/


Brief Introduction
------------------

Every JSON data is a valid CSON.

~~~~
{"hello": "world",
 "the": ["answer", "is", 42]}
~~~~

In CSON you can write a line-long comment starting with `#`.
It can go anywhere the whitespace is expected.

~~~~
# CSON data example
{"hello": "world", # ...and goodbye
 "the": ["answer", "is", 42]}
~~~~

Unlike JSON, you can use a single quoted (`'`) string as well.

~~~~
# CSON data example
{'hello': 'world', # ...and goodbye
 'the': ['answer', 'is', 42]}
~~~~

Commas right before the closing bracket (`]`) or the closing brace (`}`)
are ignored for the ease of copy and paste.

~~~~
# CSON data example
{
'hello': 'world', # ...and goodbye
'the': ['answer', 'is', 42],
}
~~~~

You can omit the comma (`,`) when followed by newline.

~~~~
# CSON data example
{
'hello': 'world' # ...and goodbye
'the': ['answer', 'is'
        42]
}
~~~~

Likewise, the colon (`:`) can be replaced with the equal sign (`=`).

~~~~
# CSON data example
{
'hello' = 'world' # ...and goodbye
'the' = ['answer', 'is'
         42]
}
~~~~

Escape sequences in the string work same as JSON.
CSON provides an alternative string syntax called a **verbatim** string
which starts with `|` and ends with a newline.
No escape sequence is processed within the verbatim string,
so `\n` in the following example is parsed as is.

~~~~
# CSON data example
{
'hello' = |world\n  ...and goodbye
'the' = ['answer', 'is'
         42]
}
~~~~

Multiple verbatim strings in a row are connected to a single string
with a newline (`\n`) among them.
You are not required to align the starts of verbatim strings,
but it would be a good habit to do so.

~~~~
# CSON data example
{
'hello' =
  |world
  |  ...and goodbye
'the' = ['answer', 'is'
         42]
}
~~~~

Connecting multiple verbatim strings
take precedence over the comma in the array.
If you want an array with multiple verbatim strings
not connected to each other,
you have to explicitly insert a comma (a bit ugly):

~~~~
# CSON data example
{
'hello' =
  |world
  |  ...and goodbye
'the' = [
  |answer
 ,|is
 ,42]
}
~~~~

Or you may put an additional newline
to separate verbatim strings (a bit better):

~~~~
# CSON data example
{
'hello' =
  |world
  |  ...and goodbye
'the' = [
  |answer

  |is

  42]
}
~~~~

You can use a bare string with quotes as the key in the object
as long as it does not contain certain chatacters
including whitespaces and CSON-special punctuations:

~~~~
# CSON data example
{
hello =
  |world
  |  ...and goodbye
the = ['answer', 'is'
       42]
}
~~~~

Finally, if the top-level data consists of the object,
the enclosing braces can be omitted:

~~~~
# CSON data example
hello =
  |world
  |  ...and goodbye
the = ['answer', 'is'
       42]
~~~~

You can now see why CSON is so good for configuration files.


Formal Grammar
--------------

CSON is defined as grammar additions
to the ABNF grammar specified by [RFC 4627],
which formally defines JSON.
Other constraints of JSON, like an unique key requirement,
equally apply to CSON.
Changes follow:

[RFC 4627]: http://tools.ietf.org/html/rfc4627

~~~~
  JSON-text = object
            / array
+           / ws object-items

  begin-array     = ws %x5B ws    ; [ left square bracket
  begin-object    = ws %x7B ws    ; { left curly bracket
  end-array       = ws %x5D ws    ; ] right square bracket
  end-object      = ws %x7D ws    ; } right curly bracket
  name-separator  = ws %x3A ws    ; : colon
+                 / ws %x3D ws    ; = equal sign
  value-separator = ws %x2C ws    ; , comma
+                 / newline ws

  ws = *(
            %x20 /                ; Space
            %x09 /                ; Horizontal tab
-           %x0A /                ; Line feed or New line
-           %x0D                  ; Carriage return
+           newline-char /
+           comment
        )
+ newline = *(%x20 / %x09) newline-char
+ newline-char = %x0A             ; Line feed or New line
+              / %x0D             ; Carriage return
+ comment = sharp *comment-char
+ sharp = %x23                    ; # sharp
+ comment-char = %x00-09 / %x0B-0C / %x0E-10FFFF

  value = false / null / true / object / array / number / string

  false = %x66.61.6c.73.65        ; false
  null  = %x6e.75.6c.6c           ; null
  true  = %x74.72.75.65           ; true

- object = begin-object [ member *( value-separator member ) ] end-object
+ object = begin-object [ object-items ] end-object
+ object-items = member *( value-separator member ) [ value-separator ]
- member = string name-separator value
+ member = name name-separator value
+ name = string / bare-string

- array = begin-array [ value *( value-separator value ) ] end-array
+ array = begin-array [ array-items ] end-array
+ array-items = value *( value-separator value ) [ value-separator ]

  number = [ minus ] int [ frac ] [ exp ]
  decimal-point = %x2E            ; .
  digit1-9 = %x31-39              ; 1-9
  e = %x65 / %x45                 ; e E
  exp = e [ minus / plus ] 1*DIGIT
  frac = decimal-point 1*DIGIT
  int = zero / ( digit1-9 *DIGIT )
  minus = %x2D                    ; -
  plus = %x2B                     ; +
  zero = %x30                     ; 0

- string = quotation-mark *char quotation-mark
+ string = quotation-mark *dquoted-char quotation-mark
+        / apostrophe-mark *squoted-char apostrophe-mark
- char = unescaped /
-        escape (
+ dquoted-char = dquoted-unescaped / escaped
+ squoted-char = squoted-unescaped / escaped
+ escaped = escape (
+            %x27 /               ; '    apostrophe      U+0027
             %x22 /               ; "    quotation mark  U+0022
             %x5C /               ; \    reverse solidus U+005C
             %x2F /               ; /    solidus         U+002F
             %x62 /               ; b    backspace       U+0008
             %x66 /               ; f    form feed       U+000C
             %x6E /               ; n    line feed       U+000A
             %x72 /               ; r    carriage return U+000D
             %x74 /               ; t    tab             U+0009
             %x75 4HEXDIG )       ; uXXXX                U+XXXX
  escape = %x5C                   ; \
  quotation-mark = %x22           ; "
+ apostrophe-mark = %x27          ; '
- unescaped = %x20-21 / %x23-5B / %x5D-10FFFF
+ dquoted-unescaped = %x20-21 / %x23-5B / %x5D-10FFFF
+ squoted-unescaped = %x20-26 / %x28-5B / %x5D-10FFFF

+ verbatim-string = verbatim-fragment *(newline ws verbatim-fragment)
+ verbatim-fragment = pipe *verbatim-char
+ pipe = %x7C                     ; |
+ verbatim-char = %x20-10FFFF

+ bare-string = id-start *id-end

+ id-start = %x24 / %x2D / %x41-5A / %x5F / %x61-7A / %xAA / %xB5
+          / %xBA / %xC0-D6 / %xD8-F6 / %xF8-02FF / %x0370-037D
+          / %x037F-1FFF / %x200C-200D / %x2070-218F / %x2C00-2FEF
+          / %x3001-D7FF / %xF900-FDCF / %xFDF0-FFFD / %x10000-EFFFF
+ id-end = id-start / %x2E / %x30-39 / %xB7 / %x0300-036F / %x203F-2040
~~~~

Please note that this grammar itself is ambiguous about
the sequence of nonterminals
`verbatim-fragment`, `newline`, `ws` and `verbatim-fragment`
in the array context (i.e. `array-items`),
which can be interpreted as a single `verbatim-string`
or a `verbatim-string` followed by `value-separator`
and another `verbatim-string`.
The parser should use the former interpretation in this case.


Design Considerations
---------------------

CSON is designed with the following considerations in mind.

### JSON Equivalence

JSON has a broad language, library and tool support.
It is very important that CSON can be translated to JSON
to leverage this support.
[YAML], on the other hand, is also a strict JSON superset
but YAML falls short on this criterion
as YAML cannot be readily converted to JSON.

[YAML]: http://yaml.org/

It is not a strong requirement for CSON to be a JSON superset,
but since JSON already has important data structures (arrays and objects)
and since many would want to write JSON fragment in the CSON data
it was decided that CSON would be a JSON superset.
This also gave a benefit of simpler grammar
and no requirement for additional types and recursive structures.

### Ease of Writing

CSON solves several major problems with hand-writing JSON by providing:

- The ability to write comments;
- The ability to use both single-quoted and double-quoted strings;
- The ability to write multi-line strings in multiple lines;
- The ability to omit quotes around the string in certain circumstances; and
- The ability to write a redundant comma.

These problems have been frequently wanted features for JSON,
especially since many of them are allowed by JavaScript and ECMAScript,
on which JSON is based.

Also, since CSON is expected to be used for configuration formats,
an equal sign `=` used by INI files can be also used in CSON.
This makes CSON a direct replacement for simple configuration files.

### Ease of Parsing

CSON is as easy to parse as JSON.
It has an obvious LL(1) grammar which is omitted for brevity
and can be implemented with modifications to the existing JSON parser.
In practice, the hand-written recursive descent parser with a combined lexer
would fare better due to its simplicity.

The design of CSON explicitly avoids the context sensitivity
by giving an unique lookahead character for different constructs,
and also avoids the dependence to different Unicode standards
by giving a simplified set of Unicode ranges as needed.
(The latter will be discussed later in depth.)

### Incompatibility to JavaScript

CSON is *not* designed for being a JavaScript subset,
as some features wanted for CSON are absent in JavaScript anyway.
(For example, JavaScript does not have a multi-line string literal.)
Therefore it was decided that
new features to CSON are made an invalid JavaScript if possible.
Specifically:

- The comment syntax (`#`) is different from JavaScript's,
  and CSON's comment will cause an unconditional error in JavaScript.
- Same for the verbatim string syntax (`|`), albeit in the limited extent.
  For example, `42, |foo` is an unconditional error,
  but `42` followed by a newline and `|foo` is not.

Still, unlike JSON
(JSON with a top-level object is always an invalid JavaScript),
CSON does not have a strong guarantee of being an invalid JavaScript.
You should avoid using CSON over HTTP for this reason.

### No Whitespace Significance

Whitespace is very prone to accidental changes,
which is not desirable for the data format.
For example, copying and pasting the whitespace-significant data
or expanding tabs to spaces in the editor
can easily lose the information from time to time.

Whitespace also makes the implementation more complex.
Whitespace-significant grammars need a special treatment for the lexer,
and should implement all possible indent and dedent scenarios.
(That is why YAML grammar is so horrible.)
It is also very hard to handle a mix of tabs and spaces;
in fact, Python traditionally had an arbitrary assumption of
eight-space tabs (!) until Python 3.

While CSON does not have a whitespace-significant structure,
one major feature of CSON does encourage the whitespace significance:
multi-line verbatim strings.
The prefix character for them, `|`, is intentionally chosen
to encourage writers to align them in the same column.

### No Additional Types

Besides from the fact that JSON does not have them,
additional types brings lots of complexity in the implementations.
As an example,
if we had a date and time format like [TOML]
then we and every implementation have to deal with the following things:

- The date without the time;
- The timezone (the UNIX timestamp would be a better option
  if you want to force UTC);
- The canonicalization of date and time
  (which is essentially impossible in certain timezones);
- Leap seconds (so you would want to force TAI instead);
- Sub-second accuracy; and
- Other niceties from [ISO 8601].

There is [Erik Naguum's excellent essay][lugm-time] on this subject.

[TOML]: https://github.com/mojombo/toml/
[ISO 8601]: https://en.wikipedia.org/wiki/ISO_8601
[lugm-time]: http://naggum.no/lugm-time.html

The truth is that,
such complex constraints are not a job of data formats.
The complexity of data formats directly affects
the (much larger) complexity of supporting implementations,
therefore we want to keep the data formats simple.

You can always define your own standard over JSON/CSON
for the interchange of customized data types.
In fact JSON users use a reserved key like `$type` for that,
so CSON respects this convention
by making a key starting with `$` easier to write.

### No Recursive Structure

Again, besides from the fact that JSON does not have it,
recursive structures are considered harmful.

Unlike programming languages
(you basically wants a Turing-completeness)
data formats should be limited in computational power
(and similarly, expressiveness)
in order to be efficiently processed.
For example, many configuration formats with programability
suffer from the inabillity of static inspection.

That said, recursive structures are not necessarily harmful.
LISP has supported recursive structures for decades
and even has a proper serialization and deserialization algorithm.
But this "feature" is not without a complexity;
the tree traversal requires a complex routine,
and it introduces free-form identifiers independent of the actual data
to the data formats.
In some cases, it even requires a temporarily mutable data structure
which is definitely bad for restricting expressiveness.

As with additional types,
the better way is to restrict data formats and
using a supplementary standard like [JSPON] on top of CSON.
That is much better than a rule-'em-all serialization format.

[JSPON]: http://www.jspon.org/

### (A bit of) Internationalization

The bare string syntax of CSON requies some explanation.
It is basically an union of two major Unicode-aware identifier syntax:

1. JavaScript (i.e. ECMAScript 5th edition) identifier syntax
2. XML (i.e. XML 1.0 5th edition) name syntax

...minus a colon (`:`), which is special to CSON.
Notably, this repertoire allows for `$` and `-` in any position,
so special keys like `$type` can be written without quotes.

Note that almost all JavaScript identifier is an XML name:
only exceptions are `$`, U+00AA, U+00B5 and U+00BA.
It is also worthwhile that the range of an XML name is very simplified;
it contains lots of unassigned characters or punctuations
that can be assumed to be letters for casual use.
(For example, U+3002 IDEOGRAPHIC FULL STOP is actually a punctuation
but included in an XML name anyway.)
This makes matching an XML name a lot easier.

Both syntaxes are carefully designed to allow as much characters as possible
without introducing any ambiguity or conflict.
For example, both syntaxes remains valid
after Unicode normalization algorithm C and D,
so a valid CSON data also remains valid after the normalization.
(This characteristics breaks down with canonical normalizations KC/KD though.)
Thanks to these prior arts,
CSON is able to make both users and implementations comfortable enough.


Frequently Asked Questions
--------------------------

### What the hell with the name?

CSON is written by hand and cursive script is also written by hand.
And I wanted to keep -SON suffix.
I apologize for careless naming.
And I am well aware of the existence of CoffeeScript Object Notation.
No, I don't intend to rename CSON.


Implementations
---------------

Working in progress. Known implementations:

* [CSON-js](http://0xabcdef.com/CSON-js/)


License
-------

The specification of CSON is dedicated to the public domain.

