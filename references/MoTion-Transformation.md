# MoTion Source Transformations

## Description

MoTion can be used not only to match objects in a model, but also to transform the original source code associated with matched model elements.

The transformation mechanism is provided by `MoTionRule` from the `MoTion-FAST-Transformation` package.

A transformation rule connects:

* a MoTion source pattern;
* a model containing the matched objects;
* the original source code;
* bindings identifying the elements to transform;
* replacement values or removal targets.

The transformation operates on the original source code using the source positions of the matched model elements.

## MoTionRule

A `MoTionRule` can be configured with the following properties:

* `sourcePattern`: the MoTion pattern used to find the elements to transform.
* `model`: the model on which the pattern is matched.
* `source`: the original source code.
* `bindings`: the bindings used for source replacements.
* `removalBindings`: the bindings identifying elements to remove.

A basic replacement rule has the following structure:

```Smalltalk
rule := MoTionRule new
    sourcePattern: pattern;
    model: model;
    source: sourceCode;
    bindings: {
        #bindingName -> newValue
    } asDictionary.

updatedSource := rule executeWithBindings.
```

## Replacement transformations

`executeWithBindings` applies replacements to the original source code based on the bindings collected from the source pattern.

The transformation follows these steps:

1. The source pattern is matched against the model.
2. The requested bindings are collected from the matches.
3. Matched values are resolved to source nodes when necessary.
4. The corresponding source ranges are identified using `startPos` and `endPos`.
5. Duplicate nodes are removed using their `startPos`.
6. Changes are sorted from right to left.
7. Each source range is replaced with the corresponding new value.

Applying changes from right to left is important because replacing a later source range does not modify the positions of earlier ranges.

The source replacement is performed using:

```Smalltalk
copyReplaceFrom:to:with:
```

## Binding matched AST nodes

A transformation can capture an AST node directly using `as:`.

For example:

```Smalltalk
nodePattern := SomeASTNode % {
    #someProperty <=> someValue
} as: #node.
```

The `#node` binding contains the matched AST node.

When the matched object responds to `startPos`, `MoTionRule` can use its source range directly for the transformation.

This is particularly useful when only a specific part of a source element has to be modified.

## Value bindings

A pattern can also capture a value rather than a direct AST node.

For example, a value can be captured using the `@` operator:

```Smalltalk
#someProperty <=> @value
```

When the resulting binding is not directly an AST node, `MoTionRule` can resolve the value to the corresponding node in the model using:

```Smalltalk
findNodeWithSourceCode:inModel:
```

This allows transformations to work with bindings that represent source values.

## Collection bindings

A collection of matched elements can also be transformed.

This can occur when using a repeated pattern such as `*rest`.

For each element of the collection, `MoTionRule`:

1. checks whether the element is already an AST node;
2. otherwise resolves it using `findNodeWithSourceCode:inModel:`;
3. identifies its source range;
4. adds the transformation if the node has not already been processed.

This allows one transformation rule to operate on multiple matched elements.

## Duplicate matches

The same source element can potentially be obtained through several matches.

To avoid applying the same transformation multiple times, matched nodes are deduplicated using their `startPos`.

Two matches referring to the same source position therefore produce only one transformation.

## Transformation order

When several source elements have to be transformed, changes are sorted by decreasing `startPos`.

In other words, transformations are applied from the end of the source toward the beginning.

For example, if three matched nodes start at positions:

```text
120
80
25
```

the transformations are applied in this order:

```text
120 → 80 → 25
```

This preserves the original positions of the elements that have not yet been modified.

## Removing source elements

`MoTionRule` also supports removing matched elements from the original source.

For this purpose, use `removalBindings` and:

```Smalltalk
updatedSource := rule executeRemoval.
```

A removal rule has the following structure:

```SmallTalk
rule := MoTionRule new
    sourcePattern: pattern;
    model: model;
    source: sourceCode;
    removalBindings: { #nodeToRemove }.

updatedSource := rule executeRemoval.
```

The removal process follows the same general resolution strategy as replacements:

1. collect the requested removal bindings;
2. resolve values to AST nodes when necessary;
3. remove duplicate nodes;
4. sort nodes by decreasing `startPos`;
5. remove their corresponding source ranges.

The source range is determined from:

```SmallTalk
node startPos
node endPos
```

The removal is then performed using:

```Smalltalk
copyReplaceFrom:to:with:
```

with an empty replacement string.

## Removing elements from lists

When removing an element from a list, the transformation also handles the surrounding comma when appropriate.

If a comma and its surrounding spacing appear immediately before the node, the removal range can be extended to include them.

Otherwise, if a comma appears immediately after the node, the removal range can be extended to include that comma and its spacing.

This allows removal transformations to preserve valid source syntax when removing elements from comma-separated structures.

## Transformation and source preservation

MoTion transformations operate on the original source code rather than reconstructing the complete source from the model.

Only the source ranges corresponding to matched nodes are modified.

For example, if a pattern captures a TypeScript method name, the transformation can replace only the identifier while preserving:

* the method body;
* whitespace;
* indentation;
* comments;
* surrounding source code.

This makes source-position-based transformations suitable for precise source modifications.

## Domain-specific transformation guides

This document describes the general transformation mechanism.

Domain-specific transformation examples should be documented in the corresponding FAST domain guide:

* TypeScript AST transformations → `FASTTypeScript-MoTion.md`
* Java AST transformations → `FASTJava-MoTion.md`
* XML AST transformations → `FASTXML-MoTion.md`
