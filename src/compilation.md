# Flow of compiling a program

## Main (`eff.ml`):
* line-editing wrapper
* load pervasives
* execution

## Execution (`eff.ml`):
```ocaml
(* Run and load all the specified files. *)
let ctxenv = List.fold_left (Shell.use_file Format.std_formatter) Shell.initial_state !files in
List.iter (compile_file ctxenv) !to_be_compiled;
if !Config.interactive_shell then toplevel ctxenv
```
* Do evaluation: `exec_cmd` (`shell.ml`)
* Compile

## Compilation (`eff.ml`):
### Code
```ocaml
(* Run and load all the specified files. *)
let ctxenv = List.fold_left (Shell.use_file Format.std_formatter) Shell.initial_state !files in
List.iter (compile_file ctxenv) !to_be_compiled;
```
```ocaml
let compile_file st filename =
  (* do_pervasives *)

  (* Parser + Lexex => Sugared Syntax *)
  let cmds = Lexer.read_file (parse Parser.file) filename in
  (* Desugar => Untyped Syntax *)
  let cmds = List.map Desugar.toplevel (pervasives_cmds @ separator :: cmds) in
  (* Type Inference => Typed Syntax *)
  let cmds, _ = Infer.type_cmds st.Shell.typing cmds in
  (* Optimization *)
  let cmds = Optimize.optimize_commands cmds in

  (* print_compiled_file *)
```

### State (Environment)
```ocaml
type state = {
  environment : RuntimeEnv.t;
  typing : Infer.toplevel_state;
}

type Infer.toplevel_state = {
  change : Scheme.dirty_scheme -> Scheme.dirty_scheme;
  typing : Infer.state;
}

type Infer.state = {
  context : TypingEnv.t;
  effects : (Type.ty * Type.ty) Untyped.EffectMap.t
}
```

### Compilation
First the pervasives are lexed and parsed. The pervasives include some extra definitions that are required (such as operators). Then the actual source file is lexed and parsed. The resulting syntax is the sugared syntax (`syntax/sugared.ml`). The pervasives are prefixed to the commands from the source file. In between the pervasives and the source, a command is inserted: `let separator = (Sugared.Term (Sugared.Const (Const.of_string "End of pervasives"), Location.unknown), Location.unknown)`. This is done in order to know where the source file starts (when debugging). The sugared syntax has placeholders for the dirt and region, but they are empty (`None`).

The lexer simply separates a string into tokens. The parser parses these tokens into terms and types. These terms and types are still very simple and basic and do not have much structure. There is only one term and one type.

After this, desugaring is done. Desugaring is a straightforward conversion from sugared syntax to untyped syntax.

inference is done using already filled state. However, the state can also be empty (since inference is redone anyway).

optimization is done by going through the AST Depth First and performing optimizations. For example, an apply is optimized as: `apply ~loc (optimize_expr st e1) (optimize_expr st e2)`. The resulting, optimized, AST is returned.

Finally, the code is printed in OCaml code. printing can be done in two ways. There is simple printing and pure printing. Simple printing goes through the AST and prints each node as-is. The pure printing also performs a purity check. Should a computation be pure, the computation is printed as an expression (which executes faster).

## Inference
Inference is done by going to the Untyped AST. Each node is converted to a Typed node. Typing information is kept using a typing context and a state, which contains the (local) typing environment and the defined effects.

The are 4 different distinctions to be made in Nodes. There are toplevel computations, computations, expressions and patterns. Inferring a node is done in a structured fashion. First, the (typed) term is reconstructed, then a scheme is constructed. This scheme is filled with a context and with constraints. Finally, the term and the scheme are packed together with a location.

### Shadowing
The state is altered in order to store variables and effects
In type definitions, externals or effect definitions, the state is kept when going to the next toplevel computation.
In between other computations, no state is maintained.
eg.
```ocaml
  effect Op : string -> unit (* Op is added to the state, this is persistent *)
  (fun x -> x);;  (* During type inference, x is stored in the state *)
                  (* State changes that occured during this computation are not persistent*)
  (fun x -> x);;
```
