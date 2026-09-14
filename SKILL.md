---
name: motion-pharo-ast-patterns
description: Create MoTion patterns and source transformations in Pharo for FAST TypeScript, Java, and XML models. Use when a user asks to find, analyze, rename, replace, or remove AST elements with MoTion.
---

# MoTion Pharo AST Patterns and Transformations

Create valid, minimal Pharo code using MoTion and the relevant FAST metamodel.

## Workflow

1. Identify the target AST domain: TypeScript, Java, or XML.
2. Load `references/MoTion.md`.
3. Load exactly one domain guide:
   - TypeScript: `references/FASTTypeScript-MoTion.md`
   - Java: `references/FASTJava-MoTion.md`
   - XML: `references/FASTXML-MoTion.md`
4. If required dependencies are unavailable, consult `references/Repositories.md`.
5. Build or revise the pattern in valid Pharo syntax.
6. For source transformations:
   - Use `MoTionRule`.
   - Use `executeWithBindings` for replacements or renames.
   - Use `executeRemoval` for removals.
   - Bind AST nodes directly when possible so `startPos` and `endPos` identify the correct source range.
7. Return the code, assumptions, and a short explanation of the relevant selectors and bindings.

## Domain Routing

- TypeScript, JavaScript, TS, or FAST TypeScript nodes: use `MoTion.md` and `FASTTypeScript-MoTion.md`.
- Java or FAST Java nodes: use `MoTion.md` and `FASTJava-MoTion.md`.
- XML, tags, attributes, or FAST XML nodes: use `MoTion.md` and `FASTXML-MoTion.md`.

If the target domain is ambiguous, ask the user to choose before writing a pattern.

## Output Rules

1. Return valid Pharo/MoTion code.
2. Keep patterns minimal and composable.
3. Use direct AST-node bindings for transformations whenever possible.
4. Preserve the user’s intent when revising existing patterns.
5. For removals, account for comma-separated syntax when applicable.
6. When asked to test an existing implementation, do not modify it unless the user explicitly requests a fix.
7. If a test fails, explain the cause and evidence before proposing changes.

## References

Treat these as source of truth:

- `references/MoTion.md`
- `references/FASTTypeScript-MoTion.md`
- `references/FASTJava-MoTion.md`
- `references/FASTXML-MoTion.md`
- `references/Repositories.md`
