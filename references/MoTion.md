## Description

MoTion is a new object pattern matcher in Pharo. A pattern matcher works on a finit set of objects that we will call a model. Examples of models are: the Pharo AST of a method, the DOM of an XML document, the objects loaded from a JSON file,. . . MoTion can deal with Pharo objects independently of the model containing the data.
MoTion combines both features for graph pattern matching and object matching, and by that it enables expressing patterns declaratively and applying matches over complex object structures.

## Installation

To install MoTion, go to the Playground (`Ctrl+OW`) in your Pharo image and execute the following Metacello script (select it and press Do-it button or `Ctrl+D`):

```Smalltalk
Metacello new
    baseline: 'MoTion';
    repository: 'github://moosetechnology/MoTion:main';
    load.
```

## Syntax

Multiple operators are used to help developers creating MoTion patterns.

1. Literals are declared the way they are in Pharo like ’A sample text here’ asMatcher and 1 asMatcher.
   Literal patterns match exactly their literal value. This is useful for specifying the value that a property of an object must have.
2. The “<=>” operator is a sort of generic matcher. It tries to match its left hand side (lhs) with the pattern on its right hand side (rhs).
   As noted before, it is a polymorphic operator depending on the lhs. If the lhs is an object, it tries to match this object with the rhs. If the lhs is
   a collection, it tries to match any element of the collection with the rhs.
3. To define an object pattern, one specifies its type using the class name followed by the “% {}” operator like in: ClassA % {} . The class expected comes before “% {}” and property values can be specified between the curly braces (see below).
4. A similar operator: “%% {}” is used to match a class or any of its subclasses.
5. These two operators can express sub-patterns on the properties of the matched object. They are specified between the curly braces. Object properties are instance variable accessors. The curly braces act as a conjunction of sub-patterns specifying the value a property should match. It can be seen as a Logical matcher.

```Smalltalk
ClassA % {
#’property1’ <=> aValue1.
#’property2’ <=> aValue2.
}
```

This pattern matches an object of class ClassA, with a property property1 having the value aValue1 and property2 having the value aValue2.
The sub-patterns could also be more complex (see bellow, Structural pattern).
This mechanism contributes to the seamless addition of various properties, in a declarative way.
6. The “% {}” operator, combined with the “<=>” operator, also allows to express Structural pattern where a first object is matched, then a second object in one of the properties of the first is matched and we express a sub-pattern on this second object:

```Smalltalk
ClassA % {
#’property1’ <=> aValue1.
#’property2’ <=> ClassB %% {
        #’property3’ <=> aValue3.
    }
}
```

This pattern matches an instance of ClassA with aValue1 in its property1, and an instance of ClassB in its property2. This second object
must have aValue3 in its property3.

7. Negation in pattern matching is handle with the “<∼=>” operator. It specifies that the lhs should not match the rhs pattern.
8. Non-Linear pattern is obtain using the “@” operator followed by a name (for example: @x). This allows to store a matched object in the “variable” to reuse it somewhere else in the pattern.
9. Wildcard (“_”) can be used to indicate a property whose name is not known, when one only cares for its value:

```Smalltalk
ClassA % {
    #_ <=> aValue.
}
```

This pattern matches an instance of ClassA with an unnamed property matching the value aValue.

10. The “>” operators implements Path traversal by allowing to “chain” multiple properties in a pattern. Such paths help reducing complex patterns expression, by accessing a chain of objects and their properties:

```Smalltalk
ClassA % {
#’property1>property2’ <=> aValue.
}
```

This pattern first match an instance of ClassA, then it takes the object in its property1 and the value in property2 of this second object. This value should match aValue. This notation allows to express in a very concise way a path in a graph of objects. Note that this operator is also polymorphic. Similarly to “<=>”, if one of the objects in the path is a collection, the operator will look for an element of this collection that allows to continue the search, that is to say that has a property matching the remaining part of the pattern.

11.1 MoTion allows to perform Recursive traversal through a “*” operator combined with the Path traversal operator “>”. In a chain of objects, one may know the initial property and the final one, but not know how long the chain of objects is.

```Smalltalk
ClassA % {
#’property1>repeatedProp*’ <=> aValue.
}
```

This pattern will match first an instance of ClassA, then the object in its property property1 then it will match a chain of objects all having a property repeatedProp and one of them containing the value aValue. The match ends on this last
object.

11.2 MoTion allows to perform Limited Recursive traversal through a “*Number” operator combined with the Path traversal operator “>”. In a chain of objects, one may know the initial property and the final one, but not know how long the chain of objects is. Also to avoid looking infinitly (specially when we have loops for properties pointing to each others).

```Smalltalk
ClassA % {
#’property1>repeatedProp*3’ <=> aValue.
}
```

This pattern will matches ans stops on the 3rd layer, whether ther is or isn't a match.

12. The “*” operator may also be combined with a wildcard (“_”).

```Smalltalk
ClassA % {
#’property1>_*>propN’ <=> aValue.
}
```

This pattern will match first an instance of ClassA, then the object in its property property1 then it will match a chain of objects with unknown properties ending with an object having a property propN with value aValue.

13. It is possible to match Complex lists using the “{}” list operator and declaring how the list should look like. Note that this is not the same operator as “% {}” (see above). This operator allows to express that given elements in a list should match specific patterns.
    {#’@x’. #’@x’} This pattern matches a list containing exactly two elements that are the same (use of a named variable).
14. The repetition operator (“*”) may also be used in a list to indicate an unspecified number of elements.
    {#’@x’. #’*_’. #’@x’} This pattern, matches a list with first and last elements equals and of unspecified length (obviously at least 2).
    To express that one element is part of a collection, MoTion offers a shortcut. To check if the value 5 is part of a collection (contained in the property someProperty of an instance of ClassA) one can use the pattern:

```Smalltalk
ClassA % {
    #someProperty <=> {#’*s1’. 5. #’*s2’}
}
```

But the same can be expressed with a shortcut:

```Smalltalk
ClassA % {
    #someProperty<=> 5
}
```

Note that this could also match an instance of ClassA with a property someProperty that matches exactly the value 5 (with no collection).
15. Set matcher is also included. It does the same thing as list matcher but without taking into consideration the order of the elements. For example:

```Smalltalk
"A pattern can be expressed this way using set matcher:"
pattern := { 2. 1. 'a'} orderIgnored.
result := pattern match: #( 1 2 'a').
```

In the above example, result will return a match, because even if order is not respected, the same elements can be found in the array that is matched with the defined pattern.
The below example also works for a set matcher inside an object matcher:

```Smalltalk
pattern := MTTestObjectA % {
			    #lst <=> { 1. 2. #’*others’} orderIgnored 
		    }.
a1 := MTTestObjectA new.
a1 lst add: 2.
a1 lst add: 1. 
a1 lst add: 3.
a1 lst add: 4.
	
result := pattern match: a1.
```

Result will return a match.

16. Finally there is another operator for Logical matcher:  orMatches:. It allows to express a disjunction of two patterns (one or the other match). (Remember that “% {}” implements a conjunction of patterns within the curly braces.)

```Smalltalk
ClassA % {
#someProperty <=> (5 orMatches: 6)
}
```

This pattern matches an instance of ClassA with a property someProperty matching the value 5 or the value 6.

17. Comparison to Numbers can be applied now after the conribution of @AminRannen, using operators like: #higherThan:, #lowerThan:, lessOrEqualTo: and higherOrEqualTo:
    For example:

```Smalltalk
ClassA % {
   #someProperty higherThan: 10
}
```

## How to use the matcher

1. One gets a “matcher” by calling the asMatcher
   method.
   “1 asMatcher” creates a matcher that only matches the value “1”.
2. A matcher as a match: method that allows it to try to match the argument.

```Smalltalk
pattern := #’@foo’ asMatcher.
results := pattern match: ’text’ .
result isMatch.
```

This creates a matcher than matches anything (and associates it with the “foo” symbol) and runs it on the string ’text’. The last line will answer true as the match was successful.
The result of match: is a MatchingResult.
As we just saw, it includes a boolean property isMatch indicating whether the match was successful or not. It also has a property, and matchingContexts which is a collection of MatchingContext objects. Each of these contexts
includes again a boolean field isMatch and a dictionary of its bindings.
To get the binding of foo in the the small example
above, one would do:

```Smalltalk
results matchingContexts first bindings
at: ’foo’.
```

This will return the string ’text’. Bindings can also be created with the as: method.
It is used to bind the result of a pattern that will be kept in the result’s bindings.
Finally to simplify getting the result of the bindings one is mostly interested in, there is a method collectBindings: that accepts a collection of (interesting) keys as parameter, and returns their values matched by a pattern.
In case there is no match, the return is an empty collection.

```Smalltalk
pattern := #’@foo’ asMatcher.
results := pattern collectBindings: {#foo } for: ’text’ .
```

This puts in results a collection of dictionaries (here there is only one) with the binding for the #foo symbol.
The result is a collection because there could be several matchings (for example with a disjunction operator). The collection holds dictionaries because we could ask for several bindings in the first parameter of the method.

## Transformations

MoTion can also be used to perform source-code transformations after identifying elements through pattern matching.

The transformation mechanism is provided by the `MoTionRule` class. A `MoTionRule` uses a MoTion source pattern to find elements in a model and applies transformations to the corresponding regions of the original source code.

A transformation rule is based on the following information:

* `sourcePattern`: the MoTion pattern used to identify the elements to transform.
* `model`: the model containing the objects on which the pattern is matched.
* `source`: the original source code that will be transformed.
* `bindings`: a dictionary associating binding names with their new values.
* `removalBindings`: a collection of binding names identifying elements to remove when using `executeRemoval`.

The transformation therefore combines the matching capabilities of MoTion with the source-position information of the matched nodes.

### MoTionRule

`MoTionRule` is defined as an object with the following slots:

```Smalltalk
Object << #MoTionRule
    slots: {
        #binding.
        #model.
        #newValue.
        #source.
        #sourcePattern.
        #bindings.
        #removalBindings
    };
    package: 'MoTion-FAST-Transformation'
```

The most important slots for creating and executing a transformation are:

* `sourcePattern`
* `model`
* `source`
* `bindings`
* `removalBindings`

The `binding` and `newValue` slots are also part of the class definition.

### Transformation with `executeWithBindings`

`executeWithBindings` applies replacements to the source according to the bindings produced by the source pattern.

The method first executes:

```Smalltalk
self sourcePattern
    collectBindings: bindings keys asArray
    for: self model
```

Therefore, the keys of the `bindings` dictionary determine which values are retrieved from the matching results.

For example, if:

```Smalltalk
bindings := {
    #methodName -> 'sum'
} asDictionary.
```

then `#methodName` is requested from the matching results, and `'sum'` is the new value that will be applied to the corresponding matched source element.

The general workflow is:

```text
sourcePattern
      ↓
match model
      ↓
collect requested bindings
      ↓
resolve matched values to source nodes
      ↓
associate nodes with new values
      ↓
sort source changes
      ↓
replace source ranges
      ↓
new source
```

### Direct AST node bindings

When a binding returned by the matcher directly refers to an AST node, `MoTionRule` detects that the object responds to `startPos`.

The node is then associated directly with the new value:

```text
matched AST node
      ↓
startPos / endPos
      ↓
source range
      ↓
replacement value
```

Before adding the transformation, the implementation checks whether another change already targets the same `startPos`. This prevents the same source node from being transformed multiple times.

### Value bindings with `@`

A binding can also contain a value rather than an AST node.

For example, a pattern can use a named binding:

```Smalltalk
#'@x'
```

If the resulting binding does not directly respond to `startPos`, `MoTionRule` attempts to find the corresponding source node using:

```Smalltalk
self findNodeWithSourceCode: nodeOrValue inModel: self model
```

This allows a value captured by a named binding to be associated with its corresponding source node before applying the transformation.

### Collection bindings with `*rest`

A binding can also contain a collection, which is particularly useful for repeated list patterns such as:

```Smalltalk
#'*rest'
```

When the binding contains a collection, `MoTionRule` iterates over every element.

For each element:

1. if the element responds to `startPos`, it is treated directly as an AST node;
2. otherwise, `findNodeWithSourceCode:inModel:` is used to resolve it to a source node;
3. if a source node is found, it is added to the transformations;
4. duplicate source positions are ignored.

This allows one binding to represent several source elements that can all receive the same replacement value.

### Source positions

Transformations rely on the source positions stored by the matched AST nodes.

The relevant positions are:

```Smalltalk
startPos
endPos
```

`startPos` identifies where the matched node starts in the source, while `endPos` identifies where it ends.

The source transformation is performed with:

```Smalltalk
copyReplaceFrom:to:with:
```

For a matched node, the corresponding source range is replaced with the new value.

### Applying several transformations

A pattern can produce several matching contexts. Therefore, `executeWithBindings` can produce several source changes during a single execution.

Before modifying the source, the changes are sorted by `startPos` in descending order:

```Smalltalk
sortedChanges := sortedChanges asSortedCollection: [ :a :b |
    a key startPos > b key startPos
].
```

This means that transformations are applied from right to left in the source.

This ordering is important because replacing text near the end of the source does not change the source positions of elements located before it.

The resulting process is:

```text
source:
[ earlier element ] ........ [ later element ]

                     ↓

apply later replacement first
                     ↓
apply earlier replacement
```

### Example of a transformation rule

A transformation can be defined by combining a source pattern, a model, source code, and replacement bindings:

```Smalltalk
pattern := FASTTypeScriptMethodDefinition % {
    #name <=> (
        FASTTypeScriptPropertyIdentifier % {
            #sourceCode <=> 'add'
        } as: #methodName
    )
}.

rule := MoTionRule new
    sourcePattern: pattern;
    model: model;
    source: sourceCode;
    bindings: {
        #methodName -> 'sum'
    } asDictionary.

newSource := rule executeWithBindings.
```

In this example:

1. the source pattern searches for a `FASTTypeScriptMethodDefinition`;
2. the method name is matched through `FASTTypeScriptPropertyIdentifier`;
3. `as: #methodName` creates a binding for the matched element;
4. the transformation associates `#methodName` with the new value `'sum'`;
5. `executeWithBindings` finds the matching source element;
6. its source range is determined using `startPos` and `endPos`;
7. the corresponding source code is replaced by `'sum'`;
8. the transformed source is returned.

The important point is that the transformation does not simply perform a global textual search for `'add'`. The MoTion pattern first identifies the corresponding model element, and the source positions of that element are then used to determine which part of the source must be modified.

### `executeRemoval`

MoTionRule also provides `executeRemoval` for removing matched elements from the source.

Instead of associating bindings with replacement values, `removalBindings` specifies which bindings should be removed.

The matching phase is performed using:

```Smalltalk
matches := self sourcePattern
    collectBindings: removalBindings
    for: self model.
```

The matched values are then resolved to source nodes using the same three cases as `executeWithBindings`:

1. direct AST nodes;
2. collections of nodes or values;
3. individual values that must be resolved with `findNodeWithSourceCode:inModel:`.

### Removing direct AST nodes

If a removal binding directly contains an AST node responding to `startPos`, that node is added to the collection of nodes to remove.

Duplicate nodes are avoided using their `startPos`.

### Removing collection bindings

If a removal binding contains a collection, every element is processed.

This is useful when a pattern captures several elements through a repeated binding such as:

```Smalltalk
#'*rest'
```

Each element is resolved to an AST node when necessary and added to the removal list.

### Removing value bindings

If a removal binding contains a value rather than an AST node, `MoTionRule` attempts to find the corresponding source node:

```Smalltalk
self findNodeWithSourceCode: nodeOrValue inModel: self model
```

If a node is found, it is added to the removal list.

### Avoiding duplicate removals

As with transformations, duplicate removals are avoided by comparing the `startPos` of the matched nodes.

This is important when the same source element can be reached by multiple matching paths or multiple matching contexts.

### Ordering removals

Before the source is modified, removal nodes are sorted by decreasing `startPos`:

```Smalltalk
sortedNodes := sortedNodes asSortedCollection: [ :a :b |
    a startPos > b startPos
].
```

Removals are therefore performed from right to left for the same reason as replacements: modifying a later source region does not invalidate the positions of earlier regions.

### Removing comma-separated elements

`executeRemoval` also contains specific handling for comma-separated source elements.

Before removing a node, the method checks whether a comma appears immediately before the node:

```Smalltalk
(newSource at: from - 1 ifAbsent: [ nil ]) = $  and: [
    (newSource at: from - 2 ifAbsent: [ nil ]) = $, ]
```

If so, the removal range is extended to include the preceding comma and space.

Otherwise, it checks whether a comma appears immediately after the node:

```Smalltalk
(newSource at: to + 1 ifAbsent: [ nil ]) = $,
```

If so, the removal range is extended to include the following comma and space.

This makes removal suitable for elements located in comma-separated source structures.

### Transformation and pattern matching

The transformation mechanism builds directly on the existing MoTion matching system.

The pattern is responsible for identifying the relevant model elements, while `MoTionRule` uses the resulting bindings and source positions to modify the original source.

Conceptually:

```text
MoTion pattern
      ↓
match model
      ↓
bindings
      ↓
matched AST nodes
      ↓
source positions
      ↓
source transformation
```

This allows MoTion to be used not only for detecting patterns in AST or object models, but also for applying structured transformations to their corresponding source code.

## Some cool examples

1. Finding superInheritances for 'Remote' Inteface in Java model:

```Smalltalk
pattern := FamixJavaModel % {
 #'allTypes>entities' <=>
  FamixJavaInterface % {
   #'superInheritances>superclass>name'
    <=> 'Remote' .
   #'isStub' <~=> true.
 } as: 'foundInterface'.
}.

pattern match: aFamixJavaModel.

```

2. Looking for methods where get.config(akey) is invoked

```Smalltalk
pattern := FASTJavaMethodEntity % {
 #'children*' <=> FASTJavaMethodInvocation % {
  #'receiver>name' <=> #'config'.
  #name <=> #get.
  #'arguments>primitiveValue' <=> aKey.
 } as: #configInvocation
}.

pattern match: aFASTJavaMethodEntity.

```

3. Decomposing the same pattern for reusage of small patterns

```Smalltalk
childrenPath := #'children*'.
receiverNamePath := #'receiver>name'.
argsVal := #'arguments>primitiveValue'.

subPattern := FASTJavaMethodInvocation % {
 receiverNamePath <=> #'config'.
 #name <=> #get.
 argsVal <=> aKey.
} as: #configInvocation.

pattern := FASTJavaMethodEntity % {
 childrenPath <=> subPattern.
}.

pattern match: aFASTJavaMethodEntity.

```

4. MoTion matching Pharo package

```Smalltalk
pattern := RPackage % {
		#name <=> #MoTion.
		#'definedClasses>methodDict' <=> CompiledMethod % { 
			#'ast>allChildren' <=>  RBMessageNode % { 
                #'selector>value' <=> #ifTrue:ifFalse:
			} as: #Node
		} as: #Method.
	} as: #Package.

pattern collectBindings: {#Node. #Method. #Package.} for: (#MoTion asPackage).
```

5. MoTion matching XML node list

```Smalltalk
pattern := XMLNodeList % {  
			#'_' <=> XMLElement % {
				#'name' <=> #'div' .  
			} as: 'XMLElement'. 
		}.

pattern match: anXMLNodeList.
```

## Finally

Don't hesitate to ask. More examples can be found in tests package.
Also if you are not familiar with MoTion, using it for the first time, no worries; here are some pages with examples: https://github.com/alesshosry/MoTionPatternsBookForAI. You can check them OR provide the files to AI agents so they can create the patterns for you ;)
Also, don't hesitate to have a look at Iguala which is the same matcher implemented in Python:  https://github.com/aranega/iguala
Finally, well, just enjoy it :)
