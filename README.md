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

If sourcing `~/.shcmprc` results in a non-0 status, then the built-in list is
used instead.
