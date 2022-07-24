# Lexical Structure

The lexer of the *juice* programming language splits up a source file into sequences of characters, called **tokens**, that form the lexical structure of the language. It always tries to maximize the length of each token.  
This document describes the categories of valid tokens.

## Whitespace and Comments

*juice* mostly ignores whitespace and comment tokens, but they serve some purpose: whitespace separates other tokens in the source file; also, it is necessary in order to correctly parse operators.  
Whitespace consists of the following characters: space, newline, carriage return, horizontal and vertical tab, form feed, and the null character.

Comment tokens are ignored by the compiler as well. There are single line comments, starting with `//` and terminated by a newline or a carriage return character, and multiline comments which start with `/*` and end with `*/`. The comment text of multiline comments can also contain other multiline comments, in this case the opening `/*`s and the closing `*/`s have to be balanced.

#### Grammar of Whitespace and a Comment

```ebnf
whitespace = whitespace_item, [ whitespace ];
whitespace_item = space
                | line_break
                | tab
                | vertical_tab
                | form_feed
                | null_character;

comment = single_line_comment
        | multiline_comment;
single_line_comment = "//", comment_text, line_break;
multiline_comment = "/*", multiline_comment_text, "*/";

comment_text = comment_text_item, [ comment_text ];
comment_text_item = ? any Unicode character except U+000A and U+000D ?;

multiline_comment_text = multiline_comment_text_item, [ multiline_comment_text ];
multiline_comment_text_item = multiline_comment
                            | ? any Unicode character except "/*" and "*/" ?;


line_break = ? U+000A ?
           | ? U+000D ?
           | ? U+000D ?, ? U+000A ?;

space = ? U+0020 ?;
tab = ? U+0009 ?;
vertical_tab = ? U+000B ?;
form_feed = ? U+000C ?;
null_character = ? U+0000 ?;
```

## Identifiers

Identifiers are used for the names of types, functions, and variables. The tokens begin with an `_` or with a letter from `A` through `Z` or `a` through `z`. In addition to these characters, the next identifier characters are allowed to contain the digits from `0` through `9`.

If you have to use an identifier that already is a reserved [keyword](#keywords), you can escape it using backticks like so: `` `class` ``.

#### Grammar of  an Identifier

```ebnf
identifier = identifier_text
           | "`", identifier_text, "`";

identifier_text = identifier_head, [ identifier_characters ];

identifier_head = "_"
                | letter;

identifier_characters = identifier_character, [ identifier_characters ];
identifier_character = "_"
                     | letter
                     | digit;

letter = uppercase_letter
       | lowercase_letter;

uppercase_letter = "A" | "B" | "C" | "D" | "E" | "F" | "G"
                 | "H" | "I" | "J" | "K" | "L" | "M" | "N"
                 | "O" | "P" | "Q" | "R" | "S" | "T" | "U"
                 | "V" | "W" | "X" | "Y" | "Z";

lowercase_letter = "a" | "b" | "c" | "d" | "e" | "f" | "g"
                 | "h" | "i" | "j" | "k" | "l" | "m" | "n"
                 | "o" | "p" | "q" | "r" | "s" | "t" | "u"
                 | "v" | "w" | "x" | "y" | "z";

digit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9";
```

## Keywords and Punctuation

There are several reserved keywords that can't be used as identifiers in *juice*. For example, they are used to introduce a declaration, or to begin a certain type of statement. Some keywords are only reserved in specific contexts and are free to use anywhere else.
Here is a list containing all of *juice*'s keywords:

- Declaration keywords: `binary`, `enum`, `extension`, `func`, `import`, `init`, `internal`, `let`, `module`, `operator`, `private`, `precedencegroup`, `public`, `static`, `struct`, `subscript`, `throws`, `trait`, `type`, `typeprivate`, and `var`.
- Statement keywords: `break`, `case`, `catch`, `continue`, `default`, `defer`, `do`, `else`, `fallthrough`, `for`, `guard`, `if`, `in`, `loop`, `match`, `return`, `throw`, `where`, and `while`
- Expression keywords: `as`, `case`, `false`, `is`, `nil`, `self`, `true`, `try`
- Type keywords: `any`, `some`, `throws`, `_`
- Pattern keywords: `_`
- Contextual keywords: `above`, `associativity`, `below`, `binary`, `didSet`, `get`, `indirect`, `left`, `none`, `postfix`, `prefix`, `right`, `set`, `Type`, `value`, `willSet`.

*juice* also considers the following tokens as reserved punctuation: `(`, `)`, `[`, `]`, `{`, `}`, `.`, `,`, `:`, `;`, `=`, `#`, `&` (as a prefix operator), `->`, `=>`, `` ` ``, `?`, and `!` (as a postfix operator). Even if some of these tokens consist of valid operator characters, they cannot be used as custom operators.

## Literals

Literals represent the value of a type in the source code. There are several different types of literals in *juice* that all have a default type they infer as, but many of which even can be used for custom types, if the corresponding standard library traits are implemented by the type.  
Here are the most common literals:

| Literal | Example | Default type | Corresponding Trait(s) |
| - | - | - | - |
| Integer | `54` | `Int` | `ExpressibleByIntegerLiteral` |
| Floating-Point | `3.14159` | `Double` | `ExpressibleByFloatingPointLiteral` |
| String | `"Hello"` | `String` | `ExpressibleByStringLiteral` and `ExpressibleByStringInterpolation` |
| Character | `'\n'` | `Char` | `ExpressibleByCharacterLiteral` |
| Boolean | `true` | `Bool` | `ExpressibleByBooleanLiteral` |

#### Grammar of a Literal

```ebnf
literal = numeric_literal
        | string_literal
        | character_literal
        | boolean_literal
        | nil_literal;

numeric_literal = integer_literal | floating_point_literal;
boolean_literal = "true" | "false";
nil_literal = "nil";
```

### Integer Literals

Integer literals represent integers of arbitrary size. Normally, they are written using decimal digits, but by using the prefixes `0b`, `0o` or `0x`, you can also write binary, octal or hexadecimal integer literals accordingly.  
In an integer literal, you can group digits using `_` between them (e.g. as a thousands separator).

The type of an integer literal is generally inferred to be an `Int`, unless you specify another type, implementing `ExpressibleByIntegerLiteral`. For example, all the other integer types in the standard library implement this trait.

#### Grammar of an Integer Literal

```ebnf
integer_literal = binary_literal
                | octal_literal
                | decimal_literal
                | hexadecimal_literal;

binary_literal = "0b", binary_digit, [ binary_literal_characters ];
binary_literal_characters = binary_literal_character, [ binary_literal_characters ];
binary_literal_character = binary_digit | "_";
binary_digit = "0" | "1";

octal_literal = "0o", octal_digit, [ octal_literal_characters ];
octal_literal_characters = octal_literal_character, [ octal_literal_characters ];
octal_literal_character = octal_digit | "_";
octal_digit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7";

decimal_literal = decimal_digit, [ decimal_literal_characters ];
decimal_literal_characters = decimal_literal_character, [ decimal_literal_characters ];
decimal_literal_character = decimal_digit | "_";
decimal_digit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9";

hexadecimal_literal = "0x", hexadecimal_digit, [ hexadecimal_literal_characters ];
hexadecimal_literal_characters = hexadecimal_literal_character, [ hexadecimal_literal_characters ];
hexadecimal_literal_character = hexadecimal_digit | "_";
hexadecimal_digit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
                  | "a" | "b" | "c" | "d" | "e" | "f"
                  | "A" | "B" | "C" | "D" | "E" | "F";
```

### Floating-Point Literals

A floating-point literal represents a floating-point value of unspecified precision.  
It is written using decimal digits followed either by a decimal point (`.`) and decimal digits for the fractional part, or by an exponent, starting with `e` or `E`, followed by an optional sign (`+` or `-`) and decimal digits that indicate which power of ten the value preceding the exponent is multiplied with, or both.  
As with integer literals, you can optionally group the digits of a floating point literal using `_` between digits.

By default, *juice* infers the type `Double`, representing a 64-bit floating point number, for a floating-point literal, but any other type that implements `ExpressibleByFloatingPointLiteral` can be specified as well. The standard library defines another type, implementing this trait: `Float`, which represents a 32-bit floating-point number.

#### Grammar of a Floating-Point Literal

```ebnf
floating_point_literal = decimal_literal, floating_point_fraction, [ floating_point_exponent ];
                       | decimal_literal, floating_point_exponent;

floating_point_fraction = ".", decimal_literal;
floating_point_exponent = floating_point_e, [ sign ], decimal_literal;

floating_point_e = "e" | "E";
sign = "+" | "-";
```

### String Literals

A string literal consists of a sequence of characters that are surrounded by double quotation marks (`"`). There are two types of string literals:

- #### Single-Line String Literals
  Single-line string literals are surrounded by a single pair of double quotes and look like this:

  ```juice
  "Hello, world!"
  ```

  As the name implies, they cannot contain a carriage return or a line feed. Additionally, a single-line string literal must not contain an unescaped double quotation mark (`"`), since it is used to delimit the literal, or an unescaped backslash (`\`), because backslashes are used to escape special characters in the string literal.

- #### Multiline String Literals
  Multiline string literals are delimited by three consecutive double quotes on either side and look like this:

  ```juice
  """
  Hello, world!
  """
  ```

  In contrast to single-line string literals, multiline literals can contain cariage return and line feed characters, as well as unescaped double quotation marks. However, three consecutive unescaped double quotes and unescaped backslashes cannot be part of a multiline string literal for the same reasons.

  All carriage returns or combinations of carriage return and line feed are converted to just a line feed in a multiline literal. If it then immediately starts with a line feed, it will get stripped from the resulting string; the same also happens if it ends with a line feed.  
  All other line feed characters are kept in the resulting string, unless escaped by a backslash (`\`) at the end of a line.

  Multiline string literals can be indented using as many space and horizontal tab characters as you like. The indentation will get stripped from the beginning of each line. You can specify the stripped indentation by indenting the closing delimiter by the desired amount of spaces/tabs.

  These rules ensure that all of the following literals result in the exact same string:

  ```juice
  """
  Hello, world!
  This is a juice string!
  """
  
  """Hello, world!
  This is a juice string!"""
  
  """
      Hello, \
      world!
      This is a juice string!
      """
  ```

#### Escape Sequences

If you want to insert a special character into either a single-line or a multiline string literal, you can use one of these escape sequences:

| Escape sequence | Unicode | Description |
| - | - | - |
| `\0` | U+0000 | Null character |
| `\\` | U+005C | Backslash |
| `\t` | U+0009 | Horizontal tab |
| `\n` | U+000A | Line feed |
| `\r` | U+000D | Carriage return |
| `\"` | U+0022 | Double quotation mark |
| `\'` | U+0027 | Single quotation mark (Apostrophe) |
| `\$` | U+0024 | Dollar sign |
| `\u{n}` | U+n | Unicode scalar, where `n` is a hexadecimal number with one to eight digits |

#### String Interpolation

Other values can be included into a string by using string interpolation. You can surround arbitrary expressions with `${...}` to include them in the resulting string.  
For example, the following string literals will produce the exact same string:

```juice
"I have 4 apples."

"I have ${3 + 1} apples."

let four = 4; "I have ${four} apples."
```

#### Raw String Literals

You can create a raw string literal by surrounding a normal string literal with a balanced amount of number signs (`#`). Raw string literals can contain unescaped versions of special characters like quotation marks or backslashes; they do not, however, support string interpolation. Here are some examples of raw string literals:

```juice
#"This literal contains an unescaped backslash: \"#

##"You can include "# in the literal, because the literal is only terminated when the right amount of number signs is encountered"##

#"""
It works for multiline literals as well
"""#
```

 If you want to use an escape sequence inside of a raw string literal, you have to put the corresponding amount of number signs between the backslash and the rest of the sequence like this:

```juice
#"First line\#nSecond line"#

##"""
Raw string literals support Unicode scalars as well.
This is a rightwards arrow: \##u{2192}.
"""##
```

---

Normally, string literals are inferred to be of type `String`, but as with other literals, custom types can implement the `ExpressibleByStringLiteral` trait, such that they can be constructed using a string literal as well. *juice* even supports defining custom behavior for literals containing string interpolation by implementing the `ExpressibleByStringInterpolation` trait.

#### Grammar of a String Literal

```ebnf
string_literal = single_line_string_literal
               | multiline_string_literal
               | raw_string_literal;

single_line_string_literal = string_literal_delimiter, string_literal_text, string_literal_delimiter;
multiline_string_literal = multiline_string_literal_delimiter, multiline_string_literal_text, multiline_string_literal_delimiter;

string_literal_text = string_literal_text_item, [ string_literal_text ];
string_literal_text_item = string_interpolation
                         | escaped_character
                         | ? any Unicode character except '"', "\", U+000A, and U+000D ?;

multiline_string_literal_text = multiline_string_literal_text_item, [ multiline_string_literal_text ];
multiline_string_literal_text_item = string_interpolation
                                   | escaped_character
                                   | escaped_newline
                                   | ? any Unicode character except '"""' and "\" ?;

string_interpolation = "${", string_interpolation_arguments, "}";
string_interpolation_arguments = string_interpolation_argument, [ ",", string_interpolation_arguments ];
string_interpolation_argument = [ identifier, ":" ], expression;

raw_string_literal = "#", raw_string_literal, "#"
                   | single_line_raw_string_literal
                   | multiline_raw_string_literal;
single_line_raw_string_literal = string_literal_delimiter, raw_string_literal_text, string_literal_delimiter;
multiline_raw_string_literal = multiline_string_literal_delimiter, multiline_raw_string_literal_text, multiline_string_literal_delimiter;

raw_string_literal_text = raw_string_literal_text_item, [ raw_string_literal_text ];
raw_string_literal_text_item = raw_escaped_character
                             | ? any Unicode character except the special combinations of "\", '"', and "#" ?;

multiline_raw_string_literal_text = multiline_raw_string_literal_text_item, [ multiline_raw_string_literal_text ];
multiline_raw_string_literal_text_item = raw_escaped_character
                                       | raw_escaped_newline
                                       | ? any Unicode character except the special combinations of "\", '"', and "#" ?;

escaped_character = escape_sequence_start, escape_sequence;
raw_escaped_character = raw_escape_sequence_start, escape_sequence;

escaped_newline = escape_sequence_start, line_break;
raw_escaped_newline = raw_escape_sequence_start, line_break;

escape_sequence_start = backslash;
raw_escape_sequence_start = "\#", ? additional "#"s to match the amount of the enclosing literal ?;

escape_sequence = "0"
                | backslash
                | "t"
                | "n"
                | "r"
                | '"'
                | "'"
                | "$"
                | "u{", unicode_scalar_digits, "}";

unicode_scalar_digits = hexadecimal_digit, 7 * [ hexadecimal_digit ];

backslash = ? U+005C ?;
```

### Character Literals

A character literal represents either an ASCII character or a Unicode scalar value. It is written by surrounding the literal character or an escape sequence with single quotation marks (`'`) like this:

```juice
'a'

'\n'
```

Character literals support the same [escape sequences as string literals](#escape-sequences).

By default, character literals are inferred to be of type `Char` that represents a 32-bit unicode scalar value. As with the other kinds of literals, custom types can be created from those literals as well, if they implement the `ExpressibleByCharacterLiteral` trait. The standard library, for example, defines another character type: `ASCIIChar`, a type representing the value of an ASCII character.

#### Grammar of a Character Literal

```ebnf
character_literal = "'", character_literal_content, "'";
character_literal_content = escape_sequence
                          | ? any Unicode character except "'" and "\" ?;
```

## Operators

Operators in *juice* consist of a combination of the following operator characters: `+`, `-`, `*`, `/`, `%`, `<`, `>`, `=`, `&`, `|`, `^`, `!`, `?`, `.`, or `~`. However, there are several special cases for some of these characters to remove ambiguities with certain language features:

- You can only use the dot (`.`) in an operator that also begins with one. For example, `.+.` is a valid operator, while `+.+` is treated as the `+` operator followed by the `.+` operator.
- Operators cannot consist of only a single question mark (`?`). Furthermore, postfix operators can begin neither with an exclamation point (`!`) nor with a question mark.
- *juice* reserves the tokens `=`, `->`, `=>`, `//`, `/*`, `*/`, and `.` for special features. Additionally, the prefix operators `<` and `&`, and the postfix operator `>` cannot be used as custom operators.

Since custom operators can be defined in *juice*, certain parsing rules are required to unambiguously determine, if an operator gets parsed as a prefix, binary or postfix operator:

- An operator is parsed as a binary operator, if it is surrounded by whitespace or if it has no whitespace on either side.
- An operator is parsed as a prefix operator, if it only has whitespace on the left side, while an operator that only has whitespace on its right side is parsed as a postfix operator.
- If an operator has no whitespace on both sides but is immediately followed by a dot (`.`), it is not parsed as a binary operator but as a postfix operator instead.
- If one of the operators `!` and `?` has no whitespace on the left, it is automatically treated as a postfix operator.
- The characters `(`, `[`, and `{` before an operator, `)`, `]`, and `}` after an operator, and the characters `,`, `;`, and `:` on either side of an operator are treated as whitespace for the purposes of these rules.

For example, in `a++ - b`, the `++` operator is parsed as a postfix operator, while `-` is considered a binary operator. Also, in `a--.b`, the `--` operator is parsed as a postfix instead of a binary operator.

Depending on the context, operators containing the 'greater than' character (`>`) and no other operator characters may be split into multiple tokens, to support writing generics like `HashMap<String, Vector<Bool>>`, where the closing `>>` could be incorrectly parsed as a bit shift operator.

#### Grammar of an Operator

```ebnf
operator = operator_characters
         | ".", dot_operator_characters;

operator_characters = operator_character, [ operator_characters ];
dot_operator_characters = dot_operator_character, [ dot_operator_characters ];

dot_operator_character = "." | operator_character;

operator_character = "+" | "-" | "*" | "/" | "%" | "<" | ">"
                   | "=" | "&" | "|" | "^" | "!" | "?" | "~";
```
