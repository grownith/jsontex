# jsonTEx
JSON Transform Expression

Domain specific language for querying, processing, and transforming JSON data to JSON data

# core concept

- Superset of plain old json. jsonTEx parser can parse normal json even without input
- Input json for json output
- Mathematic and boolean operation
- string manipulation
- Query value from any json structure

## JSON
- primitive
  - `null`
  - boolean
    - `false`
    - `true`
  - `number`
  - `string`
- collection
  - `array`
  - `object`

## Specific Symbol
- identifier
  - Any string match <code>[_A-Za-z][_A-Za-z0-9]*</code> is count as identifier
    - example of valid identifier
      - `_Z`
      - `Name`
      - `long_name_snake_case`
      - `arg21`
    - example of ivalid identifier
      - `~a`
      - `0x23`
  - Identifier will be used as property access and variable declaration
- $
  - JSON value that was the input of current scope
    - `null` when no input
- $_`identifier`_
  - Get value from environment variable or binded variable in the scope
- @
  - JSON value that was the current evaluated value when iterate collection
- @index
  - numeric value for the index when iterate collection
- @key
  - string value for the index when iterate object
    - `null` when iterate array
- @last
  - JSON value that was the last returned value when iterate collection
    - `null` for the first item
- @count
  - numeric value for the length of current iterated collection

## operator
- plus : `+`
  - `number` + `number`
    - Math add operation
  - `string` + `primitive`
    - String concatenate
  - `collection` + `collection`
    - collection concatenate by value. always return array
- minus : `-`
  - -`number`
    - shorthand for `number` * `-1`
  - `number` - `number`
    - Math remove operation
  - `object` - `array`
    - remove key from object. only remove with string typed item in the array
  - `object` - `_identifier_`
    - shorthand for `object - ["_identifier_"]`
- multiply : `*`
  - `number` * `number`
    - Math multiplication operation
  - `collection` * `collection`
    - return array of array of permutation by value
- divide : `/`
  - `number` / `number`
    - Math division operation
  - `collection` / `collection`
    - return array of array of zip by value
- modulo : `%`
  - `number` % `number`
- AND : `&&`
  - `left expression` && `right expression`
    - Return right expression if left expression pass boolean test, else return left expression
- OR : `||`
  - `left expression` || `right expression`
    - Return left expression if pass boolean test, else return right expression
- logical AND : `?&`
  - `left expression` ?& `right expression`
    - Return true or false if both expression evaluate as boolean. Return null if any expression is not a boolean
- logical OR : `?|`
  - `left expression` ?| `right expression`
    - Return true or false if both expression evaluate as boolean. Return null if any expression is not a boolean
- nullable : `??`
  - `expression` ?? `expression`
    - Test for nullability. Evaluate left expression and return if not null, else evaulate right expression
- negate : `!`
  - !`boolean`
    - Negate boolean
- boolean : `!!`
  - !!`expression`
    - Force convert any value to bolean. These below return false value
      - `null`
      - `""`
      - `0``
      - empty collection
- equality : `==` / `!=`
  - `expression` (`==` | `!=`) `expression`
    - Test equality for the same json type
    - Array will be tested for deep equality with ordered member comparison
    - Object will be tested for deep equality with unordered member comparison
- inequality : `>` / `>=` / `<` / `<=`
  - `expression` (`==` | `!=`) `expression`
    - Test inequality for the same json type
      - `null` < `boolean` < `number` < `string` < `array` < `object`
      - Array will be compare size, then deep compare each item
      - Object will be deep compare only on matched key, then compare size if equal
- ternary : `?` `:`
  - `condition expression` ? `true expression` : `false expression`
    - Test for boolean condition. Then evaluate matched expression. null if condition is not evaluated as boolean
- map : `.`
  - `expression.expression`
    - Return value by evaluation
- filter : `[?]`
  - `expression[?condition]`
    - Test for boolean condition. Filtering item out of array
- wildcard : `*`
  - `expression.*`
- power : `^`
  - `number` ^ `number`
  - ^`expression`
- sequence : `(;)`
  - `($_identifier_ = expresion;$_identifier_ = expresion;...expression)`
    - bind value to variable in this scope, then evaluate next expression until last expression to return
- pipe : `|>`
  - Declare that the left side will be evaluate as json to be used as an input of the right side

Precedence
- parentheses
- binary `*` `/`
- binary `+` `-`
- pipe `|>`

| sample |input|expression|result|
|--------|-----|----------|------|
| math operation | | 1 + 2 * 3 | 7 |
| parentheses | | (1 + 2) / 6 | 0.5 |
| array indexing | ["a","b","c"] | $[1] | "b" |
| negative indexing | ["a","b","c"] | $[-1] | "c" |
| slicing | ["a","b","c","d"] | $[1:2] | ["b","c"] |
| string slicing | "abcd" | $[1:2] | "bc" |
| map identifier | { "foo" : "bar" } | $.foo | "bar" |
| object indexing | { "foo bar" : 1 } | $["foo bar"] + 1 | 2 |
| object remove key | { "foo" : 0,"bar" : 1 } | $ - ["bar"] | { "foo" : 0 } |
| chain pipe | 1 | $ + 1 \|> $ * 2 \|> $ * 3 | 12 |
| indexed pair | { "foo" : ["a","b","c"] } | $.foo.[@index,@] | [[0,"a"],[1,"b"],[2,"c"]] |
| permutation | { "foo" : [0,1,2],"bar" : ["a","b","c"] } | $.foo * $.bar | [<br>[0,"a"],[0,"b"],[0,"c"],<br>[1,"a"],[1,"b"],[1,"c"],<br>[2,"a"],[2,"b"],[2,"c"]<br>] |
| zip | { "foo" : [0,1,2],"bar" : ["a","b","c"] } | $.foo / $.bar | [<br>[0,"a"],[1,"b"],[2,"c"]<br>] |
| scan | { "foo" : [1,2,3,4] } | $.foo.((@last ?? 0) + @) | [1,3,6,10] |
| sum | { "foo" : [1,2,3,4] } | $.foo.((@last ?? 0) + @)[-1] | 10 |
| all |  | { "foo" : [1 < 2,2 + 2 == 4,3 / 3 != 3] }<br>\|> $.foo.((@last ?? 0) ?& @)[-1] | true |
