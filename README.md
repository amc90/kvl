# kvl

kvl is a restrictive and extremely simple line-oriented means of data representation.

kvl has the following properties:
 * Data consists of keys mapping to values.
 * Keys are organised into a strict tree hierarchy.
 * Each branch of a key uses a sigil to denote type.
 * Values are simply unformatted UTF8 data
 * Comments are a value with a special type

There are multiple ways to express a kvl tree (kvl0, kvl1, etc). Each higher level is a subset of the level below it, but it is possible to freely convert backwards and forwards between levels without losing data.

## kvl0

### Example
     This comment applies to the root of the tree
    .animals.cat This is a cat
    .animals.cat.colours/00000000'black
    .animals.cat.colours/00000001'white
    .animals.cat.colours/00000002'brown
    .animals.cat.legs'4
    .animals.cat.says'meow/nmeow
    /00000000'This is an array value just to demonstrate them

### Syntax

The file consists of a series of lines, where each line terminates with LF. Extra whitespace is disallowed.

The line begins with a key, which is composed of a series of branches. When a value character is reached, the remainder of the line is considered value data.

 * **kvl0** → **line** `LF` **kvl0** | `empty string`
 * **line** → **key** **value**
 * **key** → **branch** **key** | **branch**
 * **branch** → **numeric-branch** | **associative-branch**
 * **associative-branch** → `.` **branch-identifier**
 * **numeric-branch** → `/` ``Exactly 8 utf8 numbers`` 
 * **branch-identifier** → **branch-identifier** **branch-identifier** | ``utf8 character between 0 and ~``
 * **value** → ` ` **utf8-value** | `'` **utf8-value** 
 * **utf8-value** → `empty string` | ``any utf8 value except / or LF`` | `\n` | '\\' | **utf8-value** **utf8-value**

Numeric branches are basically arrays, and the integers are zero based, and don't support gaps. The first one given must read /00000000

### Properties

 * There is exactly one possible way to express a given tree using kvl0. If the file checksum differs, then the data has changed.
 * Strictly line-oriented (one line per key-value pair).
 * Valid kvl0 is unchanged by `LC_ALL=C sort -n`

## kvl1

### Example

:.animals.cat
 This is a cat
::.colours
/'black
/'white
/'brown
:<
.legs'4
:<.says
meow/nmeow
/0'This is an array value just to demonstrate them

### Syntax

As per kvl0 (sorting isn't relaxed), except:

 * **line** → **key** **value** | **prefix-line**
 * **prefix-line** → `:` | `:` **key** | `::` **key** | `:` `<`+ **key**
 * **numeric-branch** → `/` `Exactly 8 utf8 numbers` | `/`

If a prefix-line is encountered:
 * Given `:` alone, PREFIX is unset (the default state).
 * Given `:` **key**, PREFIX will be set to **key** (which means that **key** will be prefixed to all subsequent lines)
 * Given `:` `<`+ **key**, the current PREFIX will have the rightmost branch removed for each repetition of `<`. Then the **key** will be appended.
 * Given `::` **key** the given **key** is appended to the current PREFIX.

Numeric branches may have the integer ommitted, in which case the next highest number is inserted.

### Properties
 * kvl1 is a superset of kvl0. Any kvl0 tree is valid kvl1
 * kvl1 is intended to make it possible to describe trees more concisely.

## kvl2

This will be where we relax things for human use. Whitespace and so on.
