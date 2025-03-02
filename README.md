# RegExp Escaping Proposal

This ECMAScript proposal seeks to investigate the problem area of escaping a string for use inside a Regular Expression.

<!--
[Formal specification](http://tc39-transfer.github.io/proposal-regex-escaping)
-->

## Status

This proposal is a [stage 2 proposal](https://github.com/tc39/proposals) and is awaiting implementation and more input. Please see [the issues](https://github.com/tc39-transfer/proposal-regex-escaping/issues) to get involved.


## Motivation

It is often the case when we want to build a regular expression out of a string without treating special characters from the string as special regular expression tokens. For example, if we want to replace all occurrences of the the string `let text = "Hello."` which we got from the user, we might be tempted to do `ourLongText.replace(new RegExp(text, "g"))`. However, this would match `.` against any character rather than matching it against a dot.

This is commonly-desired functionality, as can be seen from [this years-old es-discuss thread](https://esdiscuss.org/topic/regexp-escape). Standardizing it would be very useful to developers, and avoid subpar implementations they might create that could miss edge cases.


## Possible solutions:

### `RegExp.escape` function

This would be a `RegExp.escape` function, such that strings can be escaped in order to be used inside regular expressions:

```js
const str = prompt("Please enter a string");
const escaped = RegExp.escape(str);
const re = new RegExp(escaped, 'g'); // handles reg exp special tokens with the replacement.
console.log(ourLongText.replace(re));
```

```js
RegExp.escape("The Quick Brown Fox"); // "The Quick Brown Fox"
RegExp.escape("Buy it. use it. break it. fix it.") // "Buy it\. use it\. break it\. fix it\."
RegExp.escape("(*.*)"); // "\(\*\.\*\)"
RegExp.escape("｡^･ｪ･^｡") // "｡\^･ｪ･\^｡"
RegExp.escape("😊 *_* +_+ ... 👍"); // "😊 \*_\* \+_\+ \.\.\. 👍"
RegExp.escape("\d \D (?:)"); // "\\d \\D \(\?\:\)"
```

### Template tag function

This would be, for example, a template tag function `RegExp.tag`, used to produce a regular expression:

```js
const str = prompt("Please enter a string");
const re = RegExp.tag`/${str}/g`;
console.log(ourLongText.replace(re));
```

## Cross-cutting concerns

The list of escaped identifiers should be kept in sync with what the regular expression grammar considers to be syntax characters that need escaping. For this reason, instead of hard-coding the list of escaped characters, we escape characters that are recognized as `SyntaxCharacter`s by the engine. For example, if regexp comments are ever added to the specification (presumably under a flag), this ensures that they are properly escaped. Additionally, named capture groups must be accounted for.


## In other languages

 - Perl: [quotemeta(str)](http://perldoc.perl.org/functions/quotemeta.html)
 - PHP: [preg_quote(str)](http://php.net/manual/en/function.preg-quote.php)
 - Python: [re.escape(str)](https://docs.python.org/3/library/re.html#re.escape)
 - Ruby: [Regexp.escape(str)](http://ruby-doc.org/core-2.2.0/Regexp.html#method-c-escape)
 - Java: [Pattern.quote(str)](http://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html#quote(java.lang.String))
 - .NET: [Regex.Escape(str)](https://msdn.microsoft.com/en-us/library/system.text.regularexpressions.regex.escape.aspx)

Note that the languages differ in what they do (e.g. Perl does something different from C#), but they all have the same goal.

We've had [a meeting about this subject](https://github.com/benjamingr/RegExp.escape/blob/master/data/other_languages/discussions.md), whose notes include a more detailed writeup of what other languages do, and the pros and cons thereof.


## FAQ

*   **Why not escape every character?**

    Other languages that have done this regretted this choice because of the readability impact and string size. More information on why other languages have moved from this in the data folder under /other_languages.

*   **Why is each escaped character escaped?**

    See [the EscapedChars.md](https://github.com/benjamingr/RegExp.escape/blob/master/EscapedChars.md) file for a detailed per-character description.

*   **What about the `/` character?**

    Empirical data has been collected (see the /data folder) from about a hundred thousand code bases (most popular sites, most popular packages, most depended on packages and Q&A sites) and it was found out that its use case (for `eval`) was not common enough to justify addition.

*   **What about the `,` character?**

The one obscure case where this could suggest a cause for escaping, avoiding a range for user-supplied numbers in `new RegExp('a{'+ RegExp.escape('3,5') + '}')`, does not lead to any clearly safer results with escaping, as doing so will cause the sequence `{3\,5}` to be treated as a literal (rather than say throwing with bad input that an application could recover from).

*   **How is Unicode handled?**

    This proposal deals with code points and not code units, so further extensions and dealing with Unicode is done.

*   **What about `RegExp.unescape`?**

    While some other languages provide an unescape method we choose to defer discussion about it to a later point, mainly because no evidence of people asking for it has been found (while `RegExp.escape` is commonly asked for).

*   **How does this relate to EscapeRegExpPattern?**

    EscapeRegExpPattern (as the name implies) takes a pattern and escapes it so that it can be represented as a string. What `RegExp.escape` does is take a string and escapes it so it can be literally represented as a pattern. The two do not need to share an escaped set and we can't use one for the other. We're discussing renaming EscapeRegExpPattern in the spec in the future to avoid confusion for readers.

<!--
*  **Why not `RegExp.tag` or another tagged template based proposal?**

    During the first time this proposal was presented - an edge case was brought up where tagged templates were suggested as an alternative. We believe a simple function is a much better and simpler alternative to tagged templates here:
      - Users have consistently been asking for `RegExp.escape` over the past 5 years - both in this repo and elsewhere. Packages providing this functionality are very popular (see [escape-string-regexp](https://www.npmjs.com/package/escape-string-regexp) and [escape-regexp](https://www.npmjs.com/package/escape-regexp)). For comparison there are no downloads and [zero issues or interest](https://github.com/benjamingr/RegExp.tag) when I initiated work on a tag proposal.
      - When interviewing users regarding `RegExp.tag` when trying to get motivating use cases for the API - users spoken with were very confused because of the tagged templates. The feedback was negative enough and they found the API confusing and awkward enough for me to stop pursuing it.
      - Virtually every other programming language offers `.escape` (see "in other languages") and made the trade-off to ship `.escape` even though most of these could have shipped a tagged template API (equivalent, per language).
      - This proposal does not block effort on a tag proposal, the two proposals are not mutually exclusive and both APIs can eventually land.
    See [this issue](https://github.com/benjamingr/RegExp.escape/issues/45) for discussion.

-->
*   **Why don't you do X?**

    If you believe there is a concern that was not addressed yet, please [open an issue](https://github.com/benjamingr/RexExp.escape/issues).
