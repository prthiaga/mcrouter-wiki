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
JSONM is JSON object with two special optional keys: `consts` and `macros`:

```JSON
 {
 "consts": {
 "constName1": constDefinition,
 "constName2": constDefinition,
 …
 },
 "macros": {
 "macroName1": macroDefinition,
 "macroName2": macroDefinition,
 …
 },
 "customProperty1": valueWithMacros,
 "customProperty2": valueWithMacros
 }
```

<dl>
 <dt>value</dt>
 <dd>any JSON value (string, object, number, etc.)</dd>
 <dt>valueWithMacros</dt>
 <dd>JSON value that may contain macroCall, paramSubstitution, builtInCall</dd>
 <dt>macroCall</dt>
 <dd>"@macroName(param1,param2,…)"
 or

 ```JSON
 {
 "type": "macroName",
 "paramName1": valueWithMacros,
 "paramName2": valueWithMacros,
 …
 }
 ```</dd>
 
 <dt>paramSubstitution</dt>
 <dd>string of form %paramName%. Examples: "%paramName%", "a%paramName%b".
 %paramName% will be replaced with the value of the corresponding parameter.
 If the whole string is a parameter substitution (i.e., "%paramName%") parameter
 values may be any valueWithParams. Otherwise, it should be a string.</dd>
 <dt>builtInCall</dt>
 <dd>Basically the same as template call, but has no short form.
 ```JSON
 {
 "type": "merge|select|slice|transform",
 … params for this call …
 }
 ```</dd>
 <dt>constDefinition</dt>
 <dd>Equivalent to `valueWithMacros`, except only built-in macros and calls are allowed.</dd></dl>

###Comments
JSONM allows C-style comments, which are removed by
the preprocessor. Example:

```JSON
 {
 // some comment here
 "key": /* and one more comment here */ "value"
 }
```

After preprocessing: 

```JSON
 {
 "key": "value"
 }
```
###Macro
Macro is a reusable piece of JSON. One can think about it as a function that
takes an arbitrary list of values and returns `valueWithMacros`.

####Macro definition
The "macros" property should be object with macro definitions. Syntax is as follows:

```JSON
"macros": {
 "macroName": {
 "type": "macroDef",
 "params": macroDefParamList,
 "result": valueWithMacros
 }
}
```
"result" is JSONM that may contain `paramSubstitution`.

<dl>
 <dt>macroDefParamList</dt>
 <dd>[ macroDefParam, macroDefParam, …]</dd>
 <dt>macroDefParam</dt>
 <dd>"paramName" | { "name": "paramName", "default": value }</dd>
</dl>

Example:

```JSON
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

Parameters may include specified defaults. Example:

```JSON
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
Given an object with the macros from  our previous examples, other
properties may call macros like this:

```JSON
 {
 "person": "@fullName(John,Doe)",
 "car": "@car(Mercedes)"
 }
```

After preprocessing:

```JSON
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

###Escaping
These characters have special meaning for the preprocessor: ‘@', ‘%', ‘(',
‘)', ‘,'. Add two backslashes (\\) before any
character to escape the character. It will then be added to the string 'as is' and will not be interpreted as a preprocessor instruction. To escape a backslash, write \\\\. 
Two backslashes are required because JSON uses a single backslash (\) as an escape character.
Example:

```JSON
 {
 "email": "fake\\@fake.fake",
 "valid": "100\\%",
 "backslash": "\\\\"
 }
```

After preprocessing:

```JSON
 {
 "email": "fake@fake.fake",
 "valid": "100%",
 "backslash": "\\"
 }
```

_Note: "backslash" is JSON property, so it will be interpreted as only one backslash_.

###Built-in calls
These calls perform different operations on lists and objects.

####merge
Merge combines multiple lists or objects into one.

```JSON
"list": {
 "type": "merge",
 "params": [
 [1, 2],
 [3, 4]
 ]
}
```

After preprocessing:

```JSON
 "list": [1, 2, 3, 4]
```

If params contains a list of objects, they it will also be merged :

```JSON
 "object": {
 "type": "merge",
 "params": [
 {"a": 1, "b": 2},
 {"b": 3, "c": 4}
 ]
 }
```

After preprocessing:

```JSON
 "object": {
 "a": 1,
 "b": 3,
 "c": 4
 }
```
 
####select
Returns element from list/object.

```JSON
 "value": {
 "type": "select",
 "key": "a",
 "dictionary": {
 "a": 1,
 "b": 2
 }
 }
```

After preprocessing: 

```JSON
 "value": 1
```

####shuffle
Randomly shuffles list.

```JSON
 "list": {
 "type": "shuffle",
 "dictionary": [1, 2, 3, 4]
 }
```

After preprocessing, (one possible example):

```JSON
 "list": [2, 4, 3, 1]
```


####slice
Returns a slice (subrange) of list/object.

```JSON
 "list": {
 "type": "slice",
 "from": 1,
 "to": 2,
 "dictionary": [1, 2, 3, 4]
 }
```

After preprocessing:

```JSON
 "list": [2, 3]
```

####transform
Transforms elements of a list or object, using `itemTransform` and `keyTransform`
properties . `keyTransform` is optional; available only if the dictionary is an
object. `itemTransform` and `keyTransform`
are `valueWithMacros` and may use these additional parameters: `item` (`key` and
`item` in case of dictionary).

```JSON
 "transform": {
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

###Consts
Consts are `valueWithMacros` that may be substituted everywhere. One may use
built-in macros and built-in types in consts.

```JSON
 {
 "consts": [
 {
 "type": "constDef",
 "name": "author",
 "value": "John Doe"
 },
 {
 "type": "constDef",
 "name": "copyright",
 "value": "%author% owns it"
 }
 ],
 "file": "%copyright%. Some content"
 }
```

After preprocessing: 
```JSON
 {
 "file": "John Doe owns it. Some content"
 }
```

###Built-in macros
####import
Allows loading JSONM from external source. Example:

> File: cities.json
```JSON
 {
 "cities": [
 "New York",
 "Washington"
 ]
 }
```

> File: city.json
```JSON
 {
 "city": {
 "type": "select",
 "key": 0,
 "dictionary": "@import(cities.json) "
 }
 }
```

After preprocessing, `city.json` becomes:

```JSON
 {
 "city": "New York"
 }
```

####int, str
`@int` casts its argument to an integer:

```JSON
 {
 "key": "@int(100)"
 }
```

After preprocessing: 

```JSON
 {
 "key": 100
 }
```
`@str` casts its argument to string.

###Advanced
####Preprocessing sequence

* Expand macros in 'consts', except objects with `'type' : 'constDef'`
* Parse 'consts' one by one, add parsed constants to global context.
* Expand macros, except objects with `'type' : 'macroDef'`
* Remove 'consts' and 'macros' properties
* Expand everything else

####Macro context
Global context is the set of all constants. When a macro is called, it is automatically associated with its [??? Wording?]
own context, which is the set of all passed parameters.

When a substitution is performed, parameters are first looked up in the current macro
context, then in the global context. If no parameter is found in either the macro context or global context, an error is returned

The scope of the macro context is total  result of macro definition:

```JSON
 "macros": {
 "sample": {
 "type": "macroDef",
 "params": [ "param" ],
 "result" {
 // one can use %param% here - that is the scope of
 // ‘sample' macro context.
 }
 }
 }
```

Here is a more complex example, with the corresponding preprocessor steps.

> File: constsFile

```JSON
 {
 "constA": "a"
 }
```

> File: macrosFileA
```JSON
 {
 "callInner": {
 "type": "macroDef",
 "params": [ "inner" ],
 "result": "@%inner%(%constA%)"
 }
 }
```

> File: mainFile
```JSON
 {
 "consts": "@import(constsFile)",
 "macros": {
 "type": "merge",
 "params": [
 "@import(macrosFile%constA%)",
 {
 "list": {
 "type": "macroDef",
 "params": [ "param" ],
 "result": [ "%param%" ]
 }
 }
 ]
 },
 "list": "@callInner(list)"
 }
```

Affter preprocessing, mainFile becomes:

```JSON
 {
 "list": [ "a" ]
 }
```

Here are preprocessing steps:

- Expand consts
- Load `constsFile` via `import`
- Add `constA` with value = 'a' to global context
- Expand macros
- Found `merge`, expand params
- Substitute parameter for import: macrosFile%constA% => macrosFileA
- Load `macrosFileA`
- 'callInner' found in macroDef, no need to expand it
- 'list' found in macroDef, no need to expand it
- After merge, 'macros' becomes object with macro definitions
- Parse macro definitions for 'callInner' and 'list'
- Expand "@callInner(list)":
- Found macro call with one parameter
- Macro context: `{ "inner": "list" }`, global context: `{ "constA": "a" }`
- Expand "@%inner%(%constA%)"
- Found macro call with one parameter - the macro name after substitution is:
 "list"; context: { "param" : "a" }
- Macro context: { "param": "a" }
- After substitution [ "%param%" ] becomes [ "a" ]
- Nothing else to substitute, finish. 

The macro call "@macroName(paramList)" is processed like this:

- Expand macroName with current context
- Expand paramList with current context. Note: after this step, paramList will
 be a list of simple JSON values (without macros)
- Create macro context from paramList (inner context)
- Expand 'result' of corresponding macro definition with inner context.