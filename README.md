# Description of the UCIM Standard
- Specification is independent of Operating System.
- All communication is done by the standard input and output streams.
- An engine to enable all UCIM functions on receiving the <kbd>uci</kbd> command.
- All command strings sent by both the engine and the GUI should end with `\n`.
- UCIM is an extension of the UCI standard, with some new features added and some old features removed from the new standard.

# UCIM Commands
- A command comprises tokens separated by arbitrary whitespace tokens, that is <kbd>space</kbd> and <kbd>tab</kbd> characters.
- The first token in a command is the command name.
- Each command can accept a variety of flags, which are in the format `<flag> [args]...`
- Errors in the commands from GUIs should not be ignored and should be reported using the `info error <error string>` command.

## Command Specification Syntax
- All command syntax definitions start with the name of the command.
- Following the name, the different flags supported by the command are specified.
- <kbd>()</kbd> are used for grouping flags. <kbd>&</kbd> and <kbd>|</kbd> can be used inside groupings to specify relationships between flags. For example `( fen | startpos )` implies only one of the `fen` and `startpos` flags must be present at any given time.
- <kbd>[]</kbd> are used to specify optional flags. Flags inside the <kbd>[]</kbd> may or may not be present in the command. <kbd>[]</kbd> also acts similar to <kbd>()</kbd>, and can be used for grouping flags.
- Commands can have 3 types of arguments excluding the no arguments case (boolean flag): single argument, a fixed number of arguments and variadic arguments.
- Single arguments are specified using the syntax `<<description>>`. For example: `name <name>`.
- A fixed number of arguments are specified using the syntax `<<description>>...<n>`, where n is the number of arguments. For example: `fen <fen>...6`.
- Variadic flags may only be present as the last flag in the command. All tokens following the variadic flag are consumed as its arguments. They are specified using the syntax `<<description>>...`. For example: `moves <move>...`

## Commands: GUI to Engine

### <samp> uci </samp>
Tells the engine to switch to UCIM mode. The command name has been kept consistent with the UCI standard so that UCIM is backwards compatible.

After receiving the <kbd>uci</kbd> command, an engine **must** identify itself with the <kbd>id</kbd> command.

After completing any necessary setup steps, the engine should send the <kbd>uciok</kbd> command to indicate that it is ready to be used.

------

### <samp> isready </samp>
Used to synchronize the GUI with the engine. This command is usually sent when the GUI has assigned the engine with a time-consuming task and needs to wait for it to finish. This command can also check if an engine is still alive.

The engine must answer the <kbd>isready</kbd> command with <kbd>readyok</kbd> as soon as it's ready to receive and parse commands, including when it's searching.

------

### <samp> setoption name <name> [ value &lt;value&gt;... ] </samp>
Tells the engine to set one of its internal parameters named `name` to `value`. The `value` flag can be dropped when setting a boolean option.

The name of the option **must** be a single token.

`value` can consist of multiple tokens only in the case of a `string` option.

------

### <samp> ucinewgame </samp>
Used to signal to the engine that the next position is from a different game compared to the current position.

The GUI **must** send this command when sending a position from a new game, and thus engines can depend on this command.

The engine may spend some time setting up after receiving this command, so the GUI **must** send an <kbd>isready</kbd> command following this.

------

### <samp> position ( fen &lt;fen&gt;...6 | startpos ) [ moves &lt;move&gt;... ] </samp>
Sets up the given position in the engine's internal board. First, the given fen is setup, and then the provided moves are
played in the given order. `startpos` is equivalent to `fen rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1`
The GUI **MUST** send this command before setting up a position from a new game.

------

### <samp> go [ infinite | ponder | wtime &lt;wtime&gt; btime &lt;btime&gt; [ winc &lt;winc&gt; binc &lt;binc&gt; ] [ movestogo &lt;movestogo&gt; ] ] [ depth &lt;maxdepth&gt; ] [ nodes &lt;maxnodes&gt; ] [ mate &lt;mateinx&gt; ] [ movetime &lt;movetime&gt; ] [ searchmoves &lt;move&gt;... ]</samp>
Start calculating on the current internal position. Provided flags can be separated into two types, search type flags, and
limit flags.

`infinite`, `ponder`, and `wtime and others` are search type flags.

- `infinite` should start an infinite search, where the search doesn't exit till the `stop` command, even when mate is found.
- `ponder` should start a ponder search, where search exits on the `ponderhit` command.
- `wtime and others` should start a normal time bound search.
  - `wtime` and `btime` are the time remaining in milliseconds for white and black respectively.
  - `winc` and `binc` are the increments per moves for white and black respectively.
  - `movestogo` is the number of moves remaining till the next time control.
- If no search type flags are provided, an infinite search is started which exits when search is "complete".

`depth`, `nodes`, `mate`, `movetime`, and `searchmoves` are limit flags.

- `depth` sets the maximum depth limit. Search should exit on reaching that depth.
- `nodes` sets the maximum nodes limit. It is a soft limit and can be exceeded, but immidiate exit is reccommended.
- `mate` sets the mate finding limit. Search should exit when a mate equal to or shorter than the provided number of moves is found.
- `movetime` sets the time to be used for searching. Search should exit on exceeding the allocated time.
- `searchmoves` sets the move searching limits. Only the provided moves should be searched at root.

------

### <samp> stop </samp>
Stops calculating as soon as possible. <kbd>bestmove</kbd> command is sent on exit.

------

### <samp> ponderhit </samp>
Opponent has played the move the engine was pondering on. Switch to a normal search from the ponder search.

------

### <samp> quit </samp>
Quit the program as soon as possible.
