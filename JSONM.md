#JSON with macros (JSONM)

JSONM is a JSON extension that allows using macro definitions inside JSON files.

###Why?
The main goals of this format are to make configuration files more readable and to get rid
of scripts that generate huge configuration files.

###What?
JSONM is superset of JSON. Any JSON object may be treated as JSON with
macros. JSON with macros is also a valid JSON object – the difference lies in enhancing some JSON properties to allow
reuse of similar parts of large JSON objects.

###How?
Using a simple preprocessor, we generate standard JSON objects from
user-friendly JSONM by processing all macros and substituting all constants. . 

###Syntax
JSONM is a JSON object with two special optional keys: `consts` and `macros`:

```
  {
    "macros": {
      "macroName1": macroDefinition,
      "macroName2": macroDefinition,
      …
    },
    "customProperty1": valueWithMacros,
    "customProperty2": valueWithMacros
  }
```

* `value`  
 any JSON value (string, object, number, etc.)
* `valueWithMacros`  
 JSON value that may contain `macroCall`, `paramSubstitution`, `builtInCall`
* `macroCall`  
 `"@macroName(param1,param2,…)"`  
 or  
 ```
  {
    "type": "macroName",
    "paramName1": valueWithMacros,
    "paramName2": valueWithMacros,
    …
  }
 ```
* `paramSubstitution`
 string of form %paramName%. Examples: "%paramName%", "a%paramName%b".
 %paramName% will be replaced with value of the corresponding parameter.
 If the whole string is a parameter substitution (i.e., "%paramName%") parameter
 values may be any valueWithParams. Otherwise, it should be a string.</dd>
* `builtInCall`
 Basically the same as template call, but has no short form.  
 ```
  {
    "type": "if|transform|process",
    … params for this call …
  }
 ```

###Comments
JSONM allows C-style comments, which are removed by
the preprocessor. Example:

```JavaScript
 {
   // some comment here
   "key": /* and one more comment here */ "value/**/"
 }
```

After preprocessing: 

```JavaScript
 {
   "key": "value/**/"
 }
```
###Macro
Macro is a reusable piece of JSON. One can think about it as a function that
takes an arbitrary list of values and returns `valueWithMacros`.

####Macro definition
The "macros" property should be an object with macro definitions. Syntax is following:

```JavaScript
 {
   "macros": {
     "macroName": {
       "type": "macroDef",
       "params": macroDefParamList,
       "result": valueWithMacros
     }
   }
 }
```
"result" is JSONM that may contain `paramSubstitution`.

* `macroDefParamList`
 `[ macroDefParam, macroDefParam, …]`
* `macroDefParam`
 `"paramName" | { "name": "paramName", "default": value }`

Example:

```JavaScript
 {
   "pair": {
     "type": "macroDef",
     "params": [ "key", "value" ],
     "result": {
       "%key%": "%value%"
     }
   },
   "fullName": {
     "type": "macroDef",
     "params": [ "first", "last" ],
     "result": [ "@pair(first,%first%)", "@pair(last,%last%)" ]
   }
 }
```

Parameters may have defaults. Example:

```JavaScript
 {
   "car": {
     "type": "macroDef",
     "params": [
       "model",
       // parameter with default
       {
         "name": "color",
         "default": "green"
       }
     ],
     "result": {
       "model": "%model%",
       "color": "%color%"
     }
   }
 }
```

####Macro call
Given an object with the macros from our previous examples, other
properties may include macro calls:

```JavaScript
 {
   "person": "@fullName(John, Doe)",
   "car": "@car(Mercedes)"
 }
```

Trailing and leading spaces are trimmed from arguments. After preprocessing:

```JavaScript
 {
   "person": {
     "first": "John",
     "last": "Doe"
   },
   "car": {
     "model": "Mercedes",
     "color": "green"
   }
 }
```

###Consts
Consts are `valueWithMacros` that may be substituted everywhere. One may use
only built-in macros and built-in calls in consts.

```JavaScript
 {
   "macros": {
     "author": {
       "type": "constDef",
       "result": "John Doe"
     },
     "copyright": {
       "type": "constDef",
       "result": "%author% owns it"
     }
   },
   "file": "%copyright%. Some content"
 }
```

After preprocessing: 
```JavaScript
 {
   "file": "John Doe owns it. Some content"
 }
```

###Escaping
These characters have special meaning for the preprocessor: ‘@', ‘%', ‘(',
‘)', ‘,'. Add two backslashes (\\) before any character to escape
the character. It will then be added to the string 'as is' and will not
be interpreted as a preprocessor instruction. To escape a backslash, write \\\\. 
Two backslashes are required because JSON uses a single backslash (\) as
an escape character.
Example:

```JavaScript
 {
   "email": "fake\\@fake.fake",
   "valid": "100\\%",
   "backslash": "\\\\"
 }
```

After preprocessing:

```JavaScript
 {
   "email": "fake@fake.fake",
   "valid": "100%",
   "backslash": "\\"
 }
```

_Note: "backslash" is JSON property, so it will be interpreted as only one backslash_.

###Built-in macros
These macros perform different operations on JSON values.

####import
Usage: `@import(path)`  
Allows loading JSONM from external source. Example:

> File: cities.json
```JavaScript
 {
   "cities": [
     "New York",
     "Washington"
   ]
 }
```

> File: city.json
```JavaScript
 {
   "city": {
     "type": "select",
     "key": 0,
     "dictionary": "@import(cities.json) "
   }
 }
```

After preprocessing, `city.json` becomes:

```JavaScript
 {
   "city": "New York"
 }
```

####int, str, bool
Usage: `@int(5)`; `@str(@int(5))`; `@bool(true)`  
`@int` casts its argument to an integer:

```JavaScript
 {
   "key": "@int(100)"
 }
```

After preprocessing: 

```JavaScript
 {
   "key": 100
 }
```
`@str` casts its argument to string; `@bool` to boolean.

####keys, values
Usage: `@keys(object)`; `@values(object)`  
`@keys` returns list of object keys; `@values` returns list of object values:

```JavaScript
 {
   "type": "keys",
   "dictionary": {"a": 1, "b": 2}
 }
```

After preprocessing:

```JavaScript
 ["a", "b"]
```

####merge
Usage:

```JavaScript
 "type": "merge",
 "params": [ list1, list2, list3, ... ]
```

or

```JavaScript
 "type": "merge",
 "params": [ obj1, obj2, obj3, ... ]
```

or

```JavaScript 
 "type": "merge"
 "params": [ str1, str2, str3, ... ]
```

Combines multiple lists or objects into one.
In case `params` is a list of strings, `merge` concatenates them.

```JavaScript
 {
   "type": "merge",
   "params": [
     [1, 2],
     [3, 4]
   ]
 }
```

After preprocessing:

```JavaScript
 [1, 2, 3, 4]
```

If `params` is a list of objects, it will also be merged :

```JavaScript
 {
   "type": "merge",
   "params": [
     {"a": 1, "b": 2},
     {"b": 3, "c": 4}
   ]
 }
```

After preprocessing:

```JavaScript
 {
   "a": 1,
   "b": 3,
   "c": 4
 }
```
 
####select
Usage: `@select(obj,string)` or `@select(list,int)`  
Returns element from list or object.

```JavaScript
 {
   "type": "select",
   "key": "a",
   "dictionary": {
     "a": 1,
     "b": 2
   }
 }
```

After preprocessing: 

```JavaScript
 1
```

####shuffle
Usage: `@shuffle(list)`  
Randomly shuffles a list.

```JavaScript
 {
   "type": "shuffle",
   "dictionary": [1, 2, 3, 4]
 }
```

After preprocessing, (one possible example):

```JavaScript
 [2, 4, 3, 1]
```


####slice
Usage:

```JavaScript
 "type": "slice",
 "dictionary": obj,
 "from": string,
 "to": string
```

or

```JavaScript 
 "type": "slice",
 "dictionary": list/string,
 "from": int,
 "to": int
```

Returns a slice (subrange) of list, object or string.

```JavaScript
 {
   "type": "slice",
   "from": 1,
   "to": 2,
   "dictionary": [1, 2, 3, 4]
 }
```

After preprocessing:

```JavaScript
 [2, 3]
```

####size
Usage:

```JavaScript
  "type": "size",
  "dictionary": list/string/object
```

Returns size of object/array/string.

```JavaScript
 {
   "type": "size",
   "dictionary": [1, 2],
 }
```

After preprocessing:

```JavaScript
 2
```

####sort
Usage:

```JavaScript
 "type": "sort",
 "dictionary": list,
```

Sort a list of strings/numbers.

```JavaScript
 {
   "type": "sort",
   "dictionary": [2, 1],
 }
```

After preprocessing:

```JavaScript
 [1, 2]
```

####range
Usage: `@range(@int(1),@int(2))`  
Returns list of integers [from, from + 1, ..., to]

```JavaScript
 {
   "myRange": "@range(@int(1),@int(2))"
 }
```

After preprocessing:

```JSON
 {
   "myRange": [1, 2]
 }
```

####isArray, isBool, isInt, isObject, isString
Usage: `@isArray(value)`; `isBool(value)`; etc.  
Return true if value is list, bool, int, object or string respectively:
```JavaScript
 "@isString(abc)"
```

After preprocessing:

```JavaScript
 true
```

####add, sub, mul, div, mod
Usage: `@add(A,B)`; `@sub(A,B)`; etc.  
Perform corresponding operation on integers:
```JavaScript
 // 2*3 + 5
 { "value": "@add(@mul(@int(2),@int(3)),@int(5))" }
```

After preprocessing:

```JavaScript
 "value": 11
```

####contains
Usage: `@contains(dictionary,value)`  
Returns true if dictionary contains a key; list contains a value; string contains a substring:
```JSON
 { "condition": "@contains(abacaba,aca)" }
```

After preprocessing:

```JSON
 { "condition": true }
```

####empty
Usage: `@empty(dictionary)`  
Returns true if object, array or string is empty.

####less
Returns true if A is less than B. Can compare any values except objects.

```JSON
 { "condition": "@less(bcd,abcd)" }
```

After preprocessing:

```JSON
 { "condition": false }
```

####equals
Usage: `@equals(A,B)`  
Returns true if `A == B`. Can compare any values.

####and, or
Returns true if `A and B`; `A or B` respectively. Both A and B should be booleans.

###Built-in calls
####if
Usage:

```JavaScript
 "type": "if",
 "condition": bool,
 "is_true": any value
 "is_false": any value
```

Conditional operator: returns `is_true` property if `condition` is true, `is_false` otherwise:

```JavaScript
 {
   "value": {
     "type": "if",
     "condition": "@equals(a,a)"
     "is_false": "Oops",
     "is_true": "Yeah"
   }
 }
```

After preprocessing:

```JSON
  { "value": "Yeah" }
```

####transform
Usage:

```JavaScript 
 "type": "transform",
 "dictionary": obj,
 "itemTransform": macro with extended context
 "keyTranform": macro with extended context (optional)
 "itemName": string (optional, default: item)
 "keyName": string (optional, default: key)
```

or

```JavaScript
 "type": "transform",
 "dictionary": list,
 "itemTranform": macro with extended context
 "keyName": string (optional, default: key)
 "itemName": string (optional, default: item)
```

Transforms elements of a list or object, using `itemTransform` and `keyTransform`
properties. `keyTransform` is optional; available only if the dictionary is an
object. `itemTransform` and `keyTransform` are `valueWithMacros` and may use
two additional parameters: `key` and `item` (parameter names are configured
with `keyName` and `itemName` properties).

```JSON
 {
   "type": "transform",
   "keyTransform": "%item%",
   "itemTransform": "%key%",
   "dictionary": {
     "a": "b",
     "b": "c"
   }
 }
```

After preprocessing: 

```JSON
 {
   "b": "a",
   "c": "b"
 }
```

####process
Usage:

```JavaScript 
 "type": "process",
 "initialValue": any value,
 "transform": macro with extended context
 "keyName": string (optional, default: key)
 "itemName": string (optional, default: item)
 "valueName": string (optional, default: value)
```

Iterates over an object or array and transforms "value". Literally:

```
  value = initialValue
  for each (key, item) in dictionary:
    value = transform(key, item, value)
  }
  return value
```

`transform` is `valueWithMacros` and may use three additional
parameters: `key`, `item` and `value` (parameter names are configured
with `keyName`, `itemName` and `valueName` properties).

```JavaScript
 {
   "type": "process",
   "initialValue": "",
   "dictionary": ["a", "b", "c", "d"],
   "transform": "%item%%value%"
 }
```

After preprocessing:

```JSON
 "dcba"
```

####foreach
Usage:

```JavaScript 
 "type": "foreach",
 "key": string (optional, default: key)
 "item": string (optional, default: item)
 "from": object or list
 "where": macro with extended context (optional, %key% and %item%)
 "use": macro with extended context (optional, %key% and %item%)
 "top": int (optional)
 "noMatchResult": any value (optional)
```

foreach (key, item) from <from> where <where> use <use> top <int> for top <top> items from dictionary "from" which satisfy "where" condition merge <use> expansions into one dictionary.

For example, to filter dictionary:
```JavaScript
{
 "type": "foreach",
 "from": <dictionary>,
 "where": <condition>
}
```

To convert object to list:
```JavaScript
{
 "type": "foreach",
 "from": <object>,
 "use": [ <list item> ]
 "noMatchResult": []
}
```

To grab at most 2 items from <dictionary> that satisfy <condition>:
```JavaScript
{
 "type": "foreach",
 "from": <dictionary>
 "where": <condition>
 "top": 2
}
```