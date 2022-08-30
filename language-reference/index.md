---
title: Language Reference
layout: page
nav: docs
---

# The *juice* Programming Language Reference

This reference describes the lexical structure and syntax of the *juice* programming language. It defines the language as clearly as possible, while still providing some useful additional information, such that the language can be relatively easily understood and learned.

The reference often refers to types from *juice*'s standard library, which is explained more in-depth in the [Language Guide](../language-guide), as well as the [Standard Library Reference](../stdlib-reference).

## About the Grammar Notation

The notation used for describing *juice*'s grammar is derived from [extended Backus–Naur form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form).

It consists of so called non-terminal production rules that define all building blocks of the language's syntax.

For example, the grammar of an operator looks like this:

```ebnf
operator = operator_characters
         | ".", dot_operator_characters;

operator_characters = operator_character, [ operator_characters ];
dot_operator_characters = dot_operator_character, [ dot_operator_characters ];

dot_operator_character = "." | operator_character;

operator_character = "+" | "-" | "*" | "/" | "%" | "<" | ">"
                   | "=" | "&" | "|" | "^" | "!" | "?" | "~";
```

Literal characters (also called **terminal symbols**) are written by enclosing them in double quotation marks. A vertical line character (`|`) means that there is an alternative and the `,` concatenates terminal symbols and other non-terminal rules. E.g., an `operator` can either consist of `operator_characters`, or of a dot (`.`) followed by `dot_operator_characters`.

If some part of a production rule is enclosed in square brackets, that part is optional.

Sometimes it is not possible to write a terminal symbol literally in text, maybe because it's a special character, or because there is an alternative between too many characters. In this case, the Unicode value or a description is provided, surrounded by question marks (`?`), like this:

```ebnf
space = ? U+0020 ?;

comment_text_item = ? any Unicode character except U+000A and U+000D ?;
```

If a part of a production rule should be repeated a certain number of times, it can be written like multiplication:

```ebnf
unicode_scalar_digits = hexadecimal_digit, 7 * [ hexadecimal_digit ];

hexadecimal_digit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
                  | "a" | "b" | "c" | "d" | "e" | "f"
                  | "A" | "B" | "C" | "D" | "E" | "F";
```

This means that `unicode_scalar_digits` can consist of between one and eight `hexadecimal_digit`s.
