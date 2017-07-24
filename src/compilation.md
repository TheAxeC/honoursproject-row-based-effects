# Flow of compiling a program

Main (`eff.ml`):
* line-editing wrapper
* load pervasives
* execution

Execution (`eff.ml`):
```ocaml
(* Run and load all the specified files. *)
let ctxenv = List.fold_left (Shell.use_file Format.std_formatter) Shell.initial_state !files in
List.iter (compile_file ctxenv) !to_be_compiled;
if !Config.interactive_shell then toplevel ctxenv
```
* Do evaluation: `exec_cmd` (`shell.ml`)
* Compile

Compilation (`eff.ml`):
```ocaml
let compile_file st filename =
  (* do_pervasives *)

  (* Parse + Lex *)
  let cmds = Lexer.read_file (parse Parser.file) filename in
  (* Desugar *)
  let cmds = List.map Desugar.toplevel (pervasives_cmds @ separator :: cmds) in
  (* Type Inference *)
  let cmds, _ = Infer.type_cmds st.Shell.typing cmds in
  (* Optimization *)
  let cmds = Optimize.optimize_commands cmds in

  (* print_compiled_file *)
```
