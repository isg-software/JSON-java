# Fork of [stleary/JSON-java](https://github.com/stleary/JSON-java)

In this fork, I added a constructor for JSONObject allowing the user to choose the Map implementation (instead of the default HashMap).
By choice of map implementations like LinkedHashMap or some sorted map the user can influence the iteration order and thus especially the order of the toString-methods' output. While this is not important for machine-readable JSON, for a pretty-printed, human-readable JSON created by #toString(int) (probably meant for humans to edit the data), a custom order may increase readability and maintainability of the generated code.

Since support for ordering `JSONObject`'s properties [is not intended in the original project](https://github.com/stleary/JSON-java/wiki/FAQ#why-isnt-ordering-allowed-in-jsonobjects) and this change is therefore not accepted as pull request, I was encouraged to maintain an own fork. 

The reason, as mentioned in the [FAQ](https://github.com/stleary/JSON-java/wiki/FAQ#why-isnt-ordering-allowed-in-jsonobjects), is that the JSON spec defines an object to be an unordered collection. While I agree that this means that no JSON consumer must rely on any ordering of an object's properties, any JSON markup necessarily enumerates this unordered collection sequentially. The output always defines the same JSON object regardless of the order of this sequence. That means, anyone writing JSON markup may do so in any order he likes, only the reader must not expect a specific order. That said, using this implementation to customise the order of the properties for certain JSON objects when serialising them does not conflict with the JSON specs.

Since I did not modify the existing API but only _added_ to it, I decided to keep the namespace (package `org.json`) unchanged.

## Background

I needed to create JSON files which should be easily readable and editable by humans. The JSON should meet two criteria:

* It should be pretty-printed, and
* the properties of certain JSON objects should be printed in a certain order.

While the `JSONStringer` outputs all properties in the order of the `.key(…).value(…)`-calls, it does not support pretty printing. The alternative is to create `JSONObjects` with the default constructor, add properties by its `put()` methods and then call the `toString(int)` method, which performs pretty-printing – but the default `HashMap` container of `JSONObject` does not support outputting the properties in insertion order (order of the `put()` calls).

That's why I added a possibility to create a new, empty `JSONObject` with a new constructor allowing to select a different Map implementation, like a `SortedMap` oder a `LinkedHashMap` (the latter preserving insertion order, which is precisely what I needed).

Instead of passing a newly created map as parameter I decided to use a supplier argument in order to reduce the risk that the caller passes a non-empty map or retains a reference to the created map, enabling him to manipulate it other than by the `JSONObject`'s methods. Also, a new constructor with a Map argument would have clashed with the existing one taking a map and putting all it's content into a new internal HashMap.

Normally I'd have used `java.util.function.Supplier<Map<String, Object>>` as argument type, but since that would have meant to drop support for Java 1.6 and 1.7, I instead introduced a new (functional) interface `MapSupplier`. For users of Java 8 or newer, this does not mean any disadvantage, since calls with Function References like `new JSONObject(LinkedHashMap::new)` look exactly the same regardless of the interface.

## Note on maintenance

Newer versions of the [original project](https://github.com/stleary/JSON-java) will probably be merged into this fork from time to time, but not regularly.

