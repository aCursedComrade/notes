# Strings all day

> **Note:** Depending on the program or environment you are in, special characters (`() {} +` and so on) may need to be escaped in order to use them as a special operator rather than a part of the pattern. This specially applies when in a terminal/command prompt.

- A case sensitive tool, provide a basic string to match the same string anywhere within a given scope
- **`.*`** represents a wildcard
- **`^`** to match a string at the **start of a line** : `^public` selects the string "public" at the start of any line
- **`$`** to match a string at the **end of a line** : `box$` select the string "box" at the end of any line
- All options can be chained:
	- `^private.*` selects all lines which starts with "private"
	- `.*the box$` selects all lines which ends with "the box"

## Character Classes
- A way to specify a set/range of data to match, defined using `[]`
- All given character are matched per character sequentially
- `[ABCD]` would match all `A B C D` in the given scope individually
- Ranges:
	- `[A-Z]` match all uppercase and `[a-z]` match all lowercase
	- `[0-9]` match all digits
- Invert the class match by adding caret as `[^0-9]` (match anything that is not a digit)
- Special classes:
	- `\s` match all white-spaces and `\S` match all non white-spaces (inverse of the latter)
	- `\w` match all words and digits and `\W` does the inverse of the latter
	- `\d` match all digits and `\D` does inverse of the latter

## Quantifiers
- To determine how many times a given patter should occur
- Defined in multiple ways:
	- `[a-z]{6,}` matches any lowercase pattern that are 6 characters or more in length
	- `e{2,4}` matches occurrences of `e` from 2 to 4 characters in length (`ee`, `eee` and `eeee`)
	- `A-[0-9]*` matches occurrences of `A-` with **zero or more** digits (when supplied as characters, operator only counts the immediate character before it)
	- `B-[0-9]+` matches occurrences of `B-` with **one or more** digits (when supplied as characters, operator only counts the immediate character before it)

## Alternation and Optional
- Alternation acts as an `OR` operator, it matches the preceding and the proceeding matches
	- `|` "pipe" is used to define an alternation
	- `Be|be` matches either `Be` instances or `be` instances
- Optional matches patters with zero or more instances of the "optional" characters
	- `?` is used to define an optional, counts the immediate character, class or group as an optional
	- `be?` matches all instance of `b` and `be` if available
- Used with capture groups

## Capture Groups
- Capture groups `()` is used to "group" certain patters in the find operation
- These groups can be used fine-tune certain such as when using alternations and can be used to easily refer when using a replace function (unique to programs)
- Example:
	- `\t([A-Za-z]+)` matches tab spaces followed by one or more characters, the capture group `([A-Za-z]+)` can be referred by `$1` or `\1` (depends on the program). This is useful when replacing.
	- Find: `\t([A-Za-z]+)` and Replace: `>> $1` (VS Code style) using the replace functionality on the above matches by replacing tab spaces `\t` with `>> ` followed by the first capture group

