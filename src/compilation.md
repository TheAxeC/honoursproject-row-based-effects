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

### Compilation
In the beginning the state is created:
```ocaml
type state = {
  environment : RuntimeEnv.t;
  typing : Infer.toplevel_state;
}
```

First the pervasives are lexed and parsed. The pervasives include some extra definitions that are required (such as operators). Then the actual source file is lexed and parsed. The resulting syntax is the sugared syntax (`syntax/sugared.ml`). The pervasives are prefixed to the commands from the source file. In between the pervasives and the source, a command is inserted: `let separator = (Sugared.Term (Sugared.Const (Const.of_string "End of pervasives"), Location.unknown), Location.unknown)`. This is done in order to know where the source file starts (when debugging). The sugared syntax has placeholders for the dirt and region, but they are empty (`None`).

The lexer simply separates a string into tokens. The parser parses these tokens into terms and types. These terms and types are still very simple and basic and do not have much structure. There is only one term and one type.

After this, desugaring is done. Desugaring is a straightforward conversion from sugared syntax to untyped syntax.

inference ....

optimization ....

printing ....
