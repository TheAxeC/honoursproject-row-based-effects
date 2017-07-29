# Explanation of Eff source code

## Syntax
`syntax/typed.ml` represents the core/intermediate language used in the Eff compiler. The substitutions are also kept in this file. The smart constructors are defined below the substitutions. smart constructors are the typing rules as defined in the type system.

```ocaml
(** Syntax of the target language. *)

module Variable = Symbol.Make(Symbol.String)
module EffectMap = Map.Make(String)

type variable = Variable.t
type effect = Common.effect * (Type.ty * Type.ty)

type ('term, 'scheme) annotation = {
  term: 'term;
  scheme: 'scheme;
  location: Location.t;
}

type pattern = (plain_pattern, Scheme.ty_scheme) annotation
and plain_pattern =
  | PVar of variable
  | PAs of pattern * variable
  | PTuple of pattern list
  | PRecord of (Common.field, pattern) Common.assoc
  | PVariant of Common.label * pattern option
  | PConst of Const.t
  | PNonbinding

let rec pattern_vars p =
  match p.term with
  | PVar x -> [x]
  | PAs (p,x) -> x :: pattern_vars p
  | PTuple lst -> List.fold_left (fun vs p -> vs @ pattern_vars p) [] lst
  | PRecord lst -> List.fold_left (fun vs (_, p) -> vs @ pattern_vars p) [] lst
  | PVariant (_, None) -> []
  | PVariant (_, Some p) -> pattern_vars p
  | PConst _ -> []
  | PNonbinding -> []

let annotate t sch loc = {
  term = t;
  scheme = sch;
  location = loc;
}

(** Pure expressions *)
type expression = (plain_expression, Scheme.ty_scheme) annotation
and plain_expression =
  | Var of variable
  | BuiltIn of string * int
  | Const of Const.t
  | Tuple of expression list
  | Record of (Common.field, expression) Common.assoc
  | Variant of Common.label * expression option
  | Lambda of abstraction
  | TyLambda of tyabstraction
  | Effect of effect
  | Handler of handler
  | FinallyHandler of (handler * abstraction)
  | Pure of computation

(** Impure computations *)
and computation = (plain_computation, Scheme.dirty_scheme) annotation
and plain_computation =
  | Value of expression
  | LetRec of (variable * abstraction) list * computation
  | Match of expression * abstraction list
  | Apply of expression * expression
  | TyApple of expression * Type.ty
  | Handle of expression * computation

  | Call of effect * expression * abstraction
  | Bind of computation * abstraction

(** Handler definitions *)
and handler = {
  effect_clauses : (effect, abstraction2) Common.assoc;
  value_clause : abstraction;
}

(** Abstractions that take one argument. *)
and abstraction = (pattern * computation, Scheme.abstraction_scheme) annotation

(** Abstraction that take one *type* argument. **)
and tyabstraction = (Type.ty * computation, Scheme.dirty_scheme) annotation

(** Abstractions that take two arguments. *)
and abstraction2 = (pattern * pattern * computation, Scheme.abstraction2_scheme) annotation

type toplevel = plain_toplevel * Location.t
and plain_toplevel =
  | Tydef of (Common.tyname, Params.t * Tctx.tydef) Common.assoc
  | TopLet of (pattern * computation) list * (variable * Scheme.ty_scheme) list
  | TopLetRec of (variable * abstraction) list * (variable * Scheme.ty_scheme) list
  | External of variable * Type.ty * string
  | DefEffect of effect * (Type.ty * Type.ty)
  | Computation of computation
  | Use of string
  | Reset
  | Help
  | Quit
  | TypeOf of computation
```

`syntax/sugared.ml` contains the source language of Eff. This is the language that is obtained after *lexing and parsing*.
```ocaml
(** Terms *)
type variable = string
type effect = Common.effect (* string *)

type term = plain_term * Location.t
and plain_term =
  | Var of variable
  (** variables *)
  | Const of Const.t
  (** integers, strings, booleans, and floats *)
  | Tuple of term list
  (** [(t1, t2, ..., tn)] *)
  | Record of (Common.field, term) Common.assoc
  (** [{field1 = t1; field2 = t2; ...; fieldn = tn}] *)
  | Variant of Common.label * term option
  (** [Label] or [Label t] *)
  | Lambda of abstraction
  (** [fun p1 p2 ... pn -> t] *)
  | Function of abstraction list
  (** [function p1 -> t1 | ... | pn -> tn] *)
  | Effect of effect
  (** [eff], where [eff] is an effect symbol. *)
  | Handler of handler
  (** [handler clauses], where [clauses] are described below. *)

  | Let of (variable Pattern.t * term) list * term
  (** [let p1 = t1 and ... and pn = tn in t] *)
  | LetRec of (variable * term) list * term
  (** [let rec f1 p1 = t1 and ... and fn pn = tn in t] *)
  | Match of term * abstraction list
  (** [match t with p1 -> t1 | ... | pn -> tn] *)
  | Conditional of term * term * term
  (** [if t then t1 else t2] *)
  | Apply of term * term
  (** [t1 t2] *)
  | Handle of term * term
  (** [with t1 handle t2] *)

and handler = {
  effect_clauses : (effect, abstraction2) Common.assoc;
  (** [t1#op1 p1 k1 -> t1' | ... | tn#opn pn kn -> tn'] *)
  value_clause : abstraction option;
  (** [val p -> t] *)
  finally_clause : abstraction option;
  (** [finally p -> t] *)
}

and abstraction = variable Pattern.t * term

and abstraction2 = variable Pattern.t * variable Pattern.t * term

type dirt =
  | DirtParam of Common.dirtparam

type region =
  | RegionParam of Common.regionparam

type ty = plain_ty * Location.t
and plain_ty =
  | TyApply of Common.tyname * ty list * (dirt list * region list) option
  (** [(ty1, ty2, ..., tyn) type_name] *)
  | TyParam of Common.typaram
  (** ['a] *)
  | TyArrow of ty * ty * dirt option
  (** [ty1 -> ty2] *)
  | TyTuple of ty list
  (** [ty1 * ty2 * ... * tyn] *)
  | TyHandler of ty * dirt option * ty * dirt option
  (** [ty1 => ty2] *)

type tydef =
  | TyRecord of (Common.field, ty) Common.assoc
  (** [{ field1 : ty1; field2 : ty2; ...; fieldn : tyn }] *)
  | TySum of (Common.label, ty option) Common.assoc
  (** [Label1 of ty1 | Label2 of ty2 | ... | Labeln of tyn | Label' | Label''] *)
  | TyInline of ty
  (** [ty] *)

(* Toplevel commands (the first four do not need to be separated by [;;]) *)
type toplevel = plain_toplevel * Location.t
and plain_toplevel =
  | Tydef of (Common.tyname, (Common.typaram list * tydef)) Common.assoc
  (** [type t = tydef] *)
  | TopLet of (variable Pattern.t * term) list
  (** [let p1 = t1 and ... and pn = tn] *)
  | TopLetRec of (variable * term) list
  (** [let rec f1 p1 = t1 and ... and fn pn = tn] *)
  | External of variable * ty * variable
  (** [external x : t = "ext_val_name"] *)
  | DefEffect of effect * (ty * ty)
  (** [effect Eff : ty1 -> t2] *)
  | Term of term
  | Use of string
  (** [#use "filename.eff"] *)
  | Reset
  (** [#reset] *)
  | Help
  (** [#help] *)
  | Quit
  (** [#quit] *)
  | TypeOf of term
  (** [#type t] *)
```

`syntax/untyped.ml` converts the sugared language and distinguishes between values and terms. This is also a representation of the source language. No type inference has been done, as such, no types are available (untyped).

```ocaml
(** Syntax of the core language. *)

module Variable = Symbol.Make(Symbol.String)
module EffectMap = Map.Make(String)

type variable = Variable.t
type effect = Common.effect

type 'term annotation = {
  term: 'term;
  location: Location.t;
}

let add_loc t loc = {
  term = t;
  location = loc;
}

type pattern = plain_pattern annotation
and plain_pattern =
  | PVar of variable
  | PAs of pattern * variable
  | PTuple of pattern list
  | PRecord of (Common.field, pattern) Common.assoc
  | PVariant of Common.label * pattern option
  | PConst of Const.t
  | PNonbinding

(** Pure expressions *)
type expression = plain_expression annotation
and plain_expression =
  | Var of variable
  | Const of Const.t
  | Tuple of expression list
  | Record of (Common.field, expression) Common.assoc
  | Variant of Common.label * expression option
  | Lambda of abstraction
  | Effect of effect
  | Handler of handler

(** Impure computations *)
and computation = plain_computation annotation
and plain_computation =
  | Value of expression
  | Let of (pattern * computation) list * computation
  | LetRec of (variable * abstraction) list * computation
  | Match of expression * abstraction list
  | Apply of expression * expression
  | Handle of expression * computation

(** Handler definitions *)
and handler = {
  effect_clauses : (effect, abstraction2) Common.assoc;
  value_clause : abstraction option;
  finally_clause : abstraction option;
}

(** Abstractions that take one argument. *)
and abstraction = pattern * computation

(** Abstractions that take two arguments. *)
and abstraction2 = pattern * pattern * computation


(* Toplevel commands (the first four do not need to be separated by [;;]) *)
type toplevel = plain_toplevel annotation
and plain_toplevel =
  | Tydef of (Common.tyname, Params.t * Tctx.tydef) Common.assoc
  (** [type t = tydef] *)
  | TopLet of (pattern * computation) list
  (** [let p1 = t1 and ... and pn = tn] *)
  | TopLetRec of (variable * abstraction) list
  (** [let rec f1 p1 = t1 and ... and fn pn = tn] *)
  | External of variable * Type.ty * string
  (** [external x : t = "ext_val_name"] *)
  | DefEffect of effect * (Type.ty * Type.ty)
  (** [effect Eff : ty1 -> t2] *)
  | Computation of computation
  | Use of string
  (** [#use "filename.eff"] *)
  | Reset
  (** [#reset] *)
  | Help
  (** [#help] *)
  | Quit
  (** [#quit] *)
  | TypeOf of computation
  (** [#type t] *)
```

## Utils and Utils-old
### Utils
`const.ml` defines print, compare and constant definitions.

`error.ml` defines strings for error messages.

`location.ml` defines methods used to determine the location of code in the source file.
```ocaml
(* location *)
type t =
  | Unknown
  | Known of known

and known = {
  filename: string;
  start_line: int;
  start_column: int;
  end_line: int;
  end_column: int;
}
```

`poset.ml` defines a partially ordered set.

`print.ml` defines some common printing utilities.

`symbol.ml` contains methods that define how some symbols (such as handler arrow, subscripts, etc) need to be printed.

`trio.ml` defines methods related to type_params, dirt_params and region_params.

### Utils-old
`common.ml` defines some auxiliary functions mostly related to map manipulations.

`pattern.ml` contains some general pattern related methods.

`symbols.ml` contains methods that define how some symbols (such as handler arrow, subscripts, etc) need to be printed.

## Typing
`constraints.ml` contains the representation of constraints and unification.

`exhaust.ml` checks for exhaustiveness.

`infer.ml` implements the type inference algorithm. It does the conversion from untyped to typed.

`params.ml` defines the ty_param, dirt_param and region_param

`scheme.ml` defines the type schemes.

`smartPrint.ml` defines a smart (purity) printer.

`tctx.ml` defines type inference contexts.

`type.ml` defines some basic types.

`typingEnv.ml` defines a map (and functions) used for the environment.

## Runtime
`eval.ml` contains the main evaluation functions for the intermediate language.

`external.ml` defines functions that need to be defined externally, such as conversion, comparisons, etc.

`runtimeEnv.ml` defines a map (and functions) used for the environment in the runtime.

`value.ml` contains some definitions related to values.

## Parsing
`desugar.ml` desugars the syntax into the core language (from sugared to untyped).

`lexer.mll` defines how the source code is seperated into tokens.

`parser.mly` parses the tokens generated by the lexer into sugared syntax (from `syntax/sugared.ml`).

# Codegen
This folder contains the code that executes the term rewriting and the purity checks. This is contained in `codegen/optimize.ml`. `commonPrint.ml`, `purePrint.ml` and `simplePrint.ml` are just printers as their name states.

## Other files
`config.ml` contains the default values for the **optional parameters** that are available for the eff compiler.

`eff.ml` is the main file used for the binary (hence the name). It contains the **optional parameters** that can be used. The main function adds the pervasives, parses and lexer the file, infers the types, performs the optimizations and converts to OCaml code.

`shell.ml` is the main file used for the online editor. `exec_cmd` is the important function here. It takes a *parsed* source file, converts it to a typed language and evaluates it.

`version.ml` contains some version definitions. Currently these include the version and the directory of the eff binary.

## Special files
`pervasives.eff` is used in Eff files, while `header.ml` is used for the OCaml code generation.

`header.ml` contains definitions required for the code generation for the OCaml backend.

`pervasives.eff` contains header definitions for Eff including:
* operators
* range / map / ignore / fold / iter / ...
* math functions

## Recommended starting point
start with changing `syntax/typed.ml` so eff still compiles, afterwards change `typing/`.

`exec_cmd` in `shell.ml` also needs to be changed as it implements the toplevel.
