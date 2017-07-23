# Explanation of Eff source code

## Syntax
`syntax/typed.ml` represents the core/intermediate language used in the Eff compiler. The substitutions are also kept in this file. The smart constructors are defined below the substitutions. smart constructors are the typing rules as defined in the type system.

`syntax/sugared.ml` contains the source language of Eff.

`syntax/untyped.ml` converts the sugared language and distinguishes between values and terms. This is also a representation of the source language. No type inference has been done, as such, no types are available (untyped).

## Utils and Utils-old
### Utils
`const.ml`

`error.ml`

`location.ml`

`poset.ml`

`print.ml`

`symbol.ml`

`trio.ml`

### Utils-old
`common.ml`

`pattern.ml`

`symbols.ml`

## Typing
`constraints.ml`

`exhaust.ml`

`infer.ml`

`params.ml`

`scheme.ml`

`smartPrint.ml`

`tctx.ml`

`type.ml`

`typingEnv.ml`

## Runtime
`eval.ml`

`external.ml`

`runtimeEnv.ml`

`value.ml`

## Parsing
`desugar.ml`

`lexer.mll`

`parser.mly`

# Codegen
This folder contains the code that executes the term rewriting and the purity checks. This is contained in `codegen/optimize.ml`. `commonPrint.ml`, `purePrint.ml` and `simplePrint.ml` are just printers as their name states.

## Other files
`config.ml`

`eff.ml`

`shell.ml`

`version.ml`

## Special files
`header.ml`

`pervasives.eff`

## Recommended starting point
start with changing typed.ml so it compiles
then change `typing/`
