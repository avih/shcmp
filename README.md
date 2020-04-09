# shcmp
Compare outputs using different shells

```
Usage: shcmp [-h|--help] [--NOERR] [--NOTRIM] [--] <arguments>
Run different shells with <arguments> and cluster them by output.
--NOERR: capture and cluster by stdout only, else (default) stdout+stderr.
--NOTRIM: don't trim trailing newlines from the outputs display.
```

## Example:
Not all shells support the non standard `local` command, and some need to quote
assignments in `local`. This shows how different shells behave in this regard:

```
shcmp -c 'f() { local x=$(echo A B); echo "|$x|"; }; f'
```

The above might produce the following output, depending on shells availability
and versions:

```
= sh, dash, posh:
|A|

= bash, bash_posix, busybox_ash, mksh, mksh_posix:
|A B|

= yash, yash_posix:
yash: no such command `local'
||
```

Note that the captured output also includes stderr (the error message for
`yash`). Use `--NOERR` to prevent that and capture only stdout.

`shcmp` can also be used to compare the output of a script. E.g. use
`shcmp ./myscript foo bar` to run `./myscript` with two arguments in all shells
and compare their outputs.

### `--TIME`

Using this option will measure the execution time at each shell and display
a sorted summary. For instance this command:

```
shcmp --TIME -c 'i=50000; while [ $i != 0 ]; do i=$((i-1)); done; echo OK'
```

may have an output similar to this:

```
-----
1283 ms, 1 ms/measure, Linux/dash
-----
 325 ms  25 %  bash
 205 ms  16 %  oksh
 186 ms  14 %  bb_ash
 182 ms  14 %  mksh
 156 ms  12 %  ksh93
 150 ms  12 %  pdksh
  79 ms   6 %  dash
-----
= dash, bash, bb_ash, mksh, pdksh, ksh93, oksh:
OK
```

In order for `--TIME` to work, you will need to place `time.sh` at your `$PATH`
or specify it with `$SHCMP_TIME_FILE`. See https://github.com/avih/time.sh .

Alternatively, you can measure and summarize yourself by defining the functions
`shcmp_time_init`, `shcmp_time_switch_to <name>` and `shcmp_time_finish` at
`~/.shcmprc` (see below). The functions are called from the same shell context.

### `~/.shcmprc`

`shcmp` includes a built-in sample list of common shells and some posix
variants (function wrappers with additional arguments).

If the file `~/.shcmprc` exists, then it's sourced and replaces the list of
built-in shells (and wrapper functions). It should set the variable `shells` to
a space-separated list of shells to run. This list may also include function
names - which this file should define too.

E.g. `~/.shcmprc`:
```shell
myshell()       { /path/to/myshell "$@"; }
myshell_posix() { /path/to/myshell -o posix "$@"; }

shells="bash sh myshell myshell_posix"
```

You could also use `~/.shcmprc` to support setting the list via an environment
variable, e.g. replacing the last line at the file above with:
```
[ "${shells-}" ] || shells="bash sh myshell myshell_posix"
```
Will allow running this: `shells="sh myshell_posix" shcmp ./myscript` .

Additionally, if sourcing `~/.shcmprc` results in a non-0 status (e.g. if its
last command is `false`), then the built-in list is used instead.
