# json2object - Type safe Haxe/JSON (de)serializer

[![Build Status](https://travis-ci.org/elnabo/json2object.svg?branch=master)](https://travis-ci.org/elnabo/json2object)

This library uses macro and a typed position aware JSON parsing (hxjsonast : <https://github.com/nadako/hxjsonast/>) to create json parser and writer from and to every supported type.

Incorrect json files or mismatch between the object and the json will yield errors or exceptions, with information on the position of the problematic parts.

Requires at least haxe 3.4.1.

## Installation

```
haxelib install json2object
```

## Usage

### Using the parser
```haxe
var parser = new json2object.JsonParser<Cls>(); // Creating a parser for Cls class
parser.fromJson(jsonString, filename); // Parsing a string. A filename is specified for errors management
var data:Cls = parser.value; // Access the parsed class
var errors:Array<json2object.Error> = parser.errors; // Access the potential errors yield during the parsing
```

It is also possible to populate an existing Array with the errors
```haxe
var errors = new Array<json2object.Error>();
var data:Cls = new json2object.JsonParser<Cls>(errors).fromJson(jsonString, filename);
```

To print the errors, you can do
```haxe
trace(json2object.ErrorUtils.convertErrorArray(parser.errors));
```

### Using the writer
```haxe
var value:Cls;
var writer = new json2object.JsonWriter<Cls>(); // Creating a writer for Cls class
var json = writer.write(value);
```

### Using the experimental JsonSchema writer
```haxe
var schema = new json2object.utils.JsonSchemaWriter<Cls>().schema;
```

### Constraints in the parsing

- Variables defined with the `@:jignored` metadata will be ignored by the parser.
- Variables defined with the `@:optional` metadata won't trigger errors if missing.

### Supported types

- Basic types (`Int`, `Float`, `Bool`, `String`)
- `Null` and `Array`
- `Map` with `Int` or `String` keys
- Class (generics are supported)
- Anonymous structure
- Typedef alias of supported types
- Enum values
- Abstract over a supported type
- Abstract enum of String, Int, Float or Bool

### Other

- As of version 2.4.0, the parser fields `warnings` and `object` have been replaced by `errors` and `value` respectively. Previous notations are still supported.

- Anonymous structure variables can be defined to be loaded with a default value if none is specified in the json using the `@:default` metadata
```haxe
typedef Struct = {
	var normal:String;

	@:default(new Map<Int, String>())
	var map:Map<Int,String>;

	@:default(-1) @:optional
	var id:Int;
}
```

- `@:default(auto)` will, by default, initialize each field of the anonymous structure / object to its default value. No effect on non Structure/Object variables.

- Variable defined as `(default, null)` may have unexpected behaviour on some `extern` classes.

- You can alias a field from the json into another name, for instance if the field name isn't a valid haxe identifier.
```haxe
typedef Struct = {
	@:alias("public") var isPublic:Bool;
}
class Main {
	static function main() {
		var parser = new JsonParser<Struct>();
		var data = parser.fromJson('{"public": true }', "file.json");
		trace(data.isPublic);
	}
}
```
If multiple alias metadatas are on the variable only the last one is taken into account.

- As of version 3.4.0, private classes can be parsed except on the CS, Java and HL targets.

## Example

With an anonymous structure:
```haxe
import json2object.JsonParser;

class Main {
	static function main() {
		var parser = new JsonParser<{ name : String, quantity : Int }>();
		var data = parser.fromJson('{"name": "computer", "quantity": 2 }', "file.json");
		trace(data.name, data.quantity);
	}
}
```

A more complex example with a class and subclass:
```haxe
import json2object.JsonParser;

class Data {
	public var a:String;
	public var b:SubData;
	public var d:Array<Int>;
	public var e:Array<Map<String, String>>;
	public var f:Array<Float>;
	public var g:Array<Bool>;
	@:jignored
	public var h:Math;
}

class SubData {
	public var c:String;
}

class Main {
	static function main() {
		var parser = new JsonParser<Data>();
		var data = parser.fromJson('{"a": "a", "b": {"c": "c"}, "e": [ { "c": "1" }, { "c": "2" } ], "f": [], "g": [ true ] }', "file.json");
		var errors = parser.errors;

		trace(data.a);
		trace(data.b.c);

		for (e in data.e) {
			trace(e.get("c"));
		}

		trace(data.e[0].get("c");
		trace(data.f.length);

		for (g in data.g) {
			trace(data.g.length);
		}

		for (e in errors) {
			switch(e) {
				case IncorrectType(variable, expected, pos):
				case UninitializedVariable(variable, pos):
				case UnknownVariable(variable, pos):
				default:
			}
		}
	}
}
```
