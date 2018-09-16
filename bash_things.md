## "" vs ''

TODO

## `source`-ing vs executing a script

Executing a script creates a subprocess and so any changes made, like for
instance to the environment variables, are not visible to the caller shell.
However, when you source a script, the script is loaded like as if you were
typing each of its lines into your shell, so any change is kept to your caller
shell.

## .bash_profile, .bashrc and other . files

One of those two files will be automatically ?sourced? when you start your
shell, which one depending whether you are starting a login shell or not.

Bash will ?source? .bash_profile on login shells and .bashrc on non-login
shells.

A common practice is to add `source ~/.bashrc` to your `.bash_profile` file,
keeping most of your configs (aliases, functions, prompt, some environment
variables, etc) into `.bashrc`.

#### Login shell?

[Check here for the difference between login and non-login shells.](https://unix.stackexchange.com/questions/38175/difference-between-login-shell-and-non-login-shell)


## Scripting

### Variables

```bash
variable_a=foo
echo "$variable_a"
foo
echo "${variable_a}_bar"
foo_
```

Note that, in the example above, the variable name is `variable_a`, without the
`$` prefix. The `$` prefix is a shortcut for `${VARIABLE_NAME}`, which is what
you use to "resolve" the value of a variable.

### Functions

```bash
var_a=foo
var_b=bar

function woot
{
  # `local` makes var_a behave like a local variable
  local var_a
  var_a=wat
  var_b=wot
}

# var_a is still bar here
# var_b is not wot
```

#### "plain" variables vs. environment variables

Environment variables are carried over to subprocesses while plain variables
aren't. Environment variables are defined using the `export` keyword.

```bash
plain_variable=foo
export ENVIRONMENT_VARIABLE=bar
```

#### Positional variables

They contain the parameters passed to a script or function. Their values can be
obtained by **$1**, **$2**, **$3**, etc. The parameter **$0** contain the name
of the script being executed. Note that, as mentioned above, the variable names
here are **1**, **2**, **3**, **0**, etc.

`$#` contains the number of parameters passed. Both `$@` and `$*` resolve to a
list of all parameters passed but with different format/semantics.

`$@` is equal to `"$0" "$1" ... "$N"` (being $N the last parameter passed),
while `$*` is a string with all positional parameters separed by the first
character in the environment variable **IFS** (defaulting to a space).

Note that you can use `$*` and **IFS** to print, for instance, a comma-separated
list of parameters of a script or a function.

### String operators

Use them to modify the value of string variables. They are useful for handling
missing values, removing/replacing parts of a string (like the dirname or
extension of a file).

- `${#variable}`: returns the length of the value of `variable`.
- `${variable:-default_value}`: returns `variable` when not null or empty
  string - otherwise returns `default_value`.
- `${variable:=default_value}`: when `variable` is not null and not empty,
  returns it. Otherwise assigns `default_value` to `variable` and returns
  it.
- `${variable:?'some error message'}`: when `variable` is null or empty, errors
  out with the given error message. Otherwise returns `variable`.
- `${variable:+some_value}`: when `variable` is not null and not empty, returns
  `some_value`. Otherwise returns null.
- `${variable:offset:length}`: returns a substring defined by `offset` (0-based
  starting point) and `length`. Allows negative `length`. If `length` is
  omitted, the end of the string is assumed.
- `${variable#pattern}`: deletes the shortest pattern matching `pattern` at the
  beginning of the variable and returns the rest.
- `${variable##pattern}`: deletes the longest pattern matching `pattern` at the
  beginning of the variable and returns the rest.
- `${variable%pattern}`: deletes the shortest pattern matching `pattern` at the
  end of the variable and returns the rest.
- `${variable%%pattern}`: deletes the longest pattern matching `pattern` at the
  beginning of the variable and returns the rest.
- `${variable/pattern/value}`: returns `variable` with the first matched
  `pattern` replaced with `value`. Similar to `s/pattern/value/` on VIM.
- `${variable//pattern/value}`: returns `variable` with all matched `pattern`
  replaced with `value`. Similar to `s/pattern/value/g` on VIM.

Note that many of those operators can be used with the positional variables (and
the `@` variable).

### Command substitution

Use `$(command)` in order to execute `command` and capture its standard output.

```bash
pwd=$(pwd) # writes the current working directory to the `pwd` variable
contents=$(< filename) # writes the contents of `filename` into `contents`
contents2=$(cat filename) # writes the contents of `filename` into `contents`
```

### Flow control - if, for, case

#### The `if`

```bash
if commands
then
  statements
elif commands
  statements
else
  statements
fi
```

The plain if expects a status code instead of a boolean - that's why we used
`commands` as its condition, since every command has an exit status code. The
status code 0 is considered a "success" while any other non-zero value is a
"failure".

A command in this case can also be a function call, however you must make sure
the function returns a numeric status code. You can use the `return $status`
keyword to return explicitly or just make sure the latest command you execute
returns the status code that your function needs to return, since bash will
return it by default.

On the example above, `commands` can consist of multiple commands chained with
boolean operators `&&` or `||`. They behave as they usually do in most
programming languages.

#### The `test` or `[` constructs

`[` is a shortcut for `test`. You can use `[` to overcome the `if` limitation to
exit codes. For example:

```bash
$variable=abc
if [ $variable='abc' ]; then
  echo 'woot'
fi
```

Here is a list of some operators supported by `[`:

- `[ $variable1=$variable2 ]`
- `[ $variable1!=$variable2 ]`
- `[ $variable1<$variable2 ]`
- `[ $variable1>$variable2 ]`
- `[ -n $variable ]`: "true" if $variable is not null and not empty.
- `[ -z $variable ]`: "true" if $variable is null or empty.
- `[ -a $file ]`: "true" if $file exists.
- `[ -d $directory ]`: "true" if $directory is an existing directory.
- `[ -e $file ]`: same as -a.
- `[ -f $file ]`: "true" if $file exists and is a regular file.
- `[ -r $file ]`: "true" if you have read permission on $file.
- `[ -s $file ]`: "true" if $file exists and is not empty.
- `[ -w $file ]`: "true" if you have write permission on $file.
- `[ -x $file ]`: "true" if you have execute permission on $file (can also be a
  directory).
- `[ -N $file ]`: "true" if $file was modified since it was last read.
- `[ -O $file ]`: "true" if you own $file.
- `[ -G $file ]`: "true" if $file exists.
- `[ $file1 -nt $file2 ]`: "true" if $file1 is newer than $file2.
- `[ $file1 -ot $file2 ]`: "true" if $file1 is older than $file2.

You can chain `[` operators with `-a` and `-o`, which are equivalent to `&&` and
`||`. You can also change their evaluation order by using parenthesis `()`,
however make sure you escape them with `\` so bash don't treat them as a special
character.

#### The `for`

```bash
list="a b c"
for thing in $list
do
  echo $thing
done
```

The example above prints "a", "b" and "c", each on a new line. The `in $list`
part can be ommited, which causes bash to default to `in "$@"`.

The `for` uses the first character on the environment variable `IFS` (defaulting
to a space when `IFS` is null) to "split" each value on the `$list` variable
above.

#### The `case`

```bash
case expression in
  pattern1 ) statements ;;
  pattern2 ) statements ;;
  * ) statements ;; # "default"/catch-all clause
esac
```

On the example above, `statements` can consist of multiple commands on multiple
lines - the branch execution stops at `;;`.

#### `while` and `until`

```bash
while condition do
  statements
done
```

The `until` can be used in the same way, with the only difference that it
repeats until a successful status code is returned. You can use `[` as part of
the condition for both statements.
