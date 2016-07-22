# Closhell

[![Build Status](https://travis-ci.org/MateuszKubuszok/Closhell.png)](https://travis-ci.org/MateuszKubuszok/Closhell)

A simple experiment to see how far could I introduce somehow functional style of programming into my Bash scripts.

## Why?

![Some men just want to watch the world burn](http://i0.kym-cdn.com/photos/images/original/000/494/455/f31.jpg)

Because I could.

## How to use it?

First include `closhell` into your script using `source` or something:

```bash
#!/bin/bash

# your stuff

source closhell_dir/closhell

# your stuff
```

### apply

First create a closure:

```bash
closure="$(cat <<-BODY
  # since each touch produces side-effects we might say that this a bad touch
	touch "$test_dir/test_file_apply\$1.tmp"
BODY
)"
```

I know that this syntax is shitty but it's the best thing I came up with. Notice that you have to escape dollar sign in
front of arguments: `\$1`. Unescaped variables (and quotation marks) will be resolved in the context current to
the closure definition. Then such closure can be called using:

```bash
apply1 "argument" "$closure"
```

Alternatively you can pass closure on creation as well:

```bash
apply1 "argument" "$(cat <<-BODY
	touch "$test_dir/test_file_apply\$1.tmp"
BODY
)"
```

There are apply version taking from 0 to 10 arguments (`apply0`, `apply1`, ..., `apply10`). A closure is always passed
as the last one.

(Well, recently I created the universal `apply` but whatever. What could go wrong if someone used specific one and
miscounted?)

Things to remember about:

  * `<<-DELIMITER ... DELIMITER` helpdoc syntax allows us for using nested blocks - it removes leftmost tabs so as long
    as you use tabs for indentation you can nest closures as many times as you want and `DELIMITER` will still work,
  * each nested closure would require one more level of escaping. So `$value` would bind `value` in all blocks,
    `\$value` would resolve to something defined in/passed into your closure (local variable or parameter),
    `\\\$variable` would resolve to variable in cosure nested in closure and so on. Same thing should be remembered
    about quotation marks `"`, `\"`, `\\\"` should be used accordingly. It's a major PITA but I couldn't reasonable
     workaround.

### call

`call` is a twin function to `apply`. The only difference is that it takes closure as first parameter not the last.

```bash
closure="$(cat <<-BODY
	touch "$test_dir/test_file_apply\$1.tmp"
BODY
)"

call "$closure" "argument"
```

As such it is more suited for calling closure stored in variable and/or binding while `apply` works better for applying
closure immediatelly. Which is why in the _suggestions_ section I did it completely the other way.

### bind

To be honest we can use all good `alias` here but that reasoning would prevent me from reinventing the wheel for my own
amusement.

```bash
bind create touch "$test_dir/test_file_binda.tmp"

create
```

 * first argument - name of a created alias,
 * second argument - name of a bounded function,
 * third argument - bounded arguments.

I thought about returning a closure similarly to `apply`, but that would be annoying as `alias` just bind first n
arguments and always allows you to pass as many more arguments as you want (which would be difficult todo when you pass
arguments manually). Maybe in the future I came up with something better :P

### map

It allows one calling some closure for a collection of arguments (by collection I mean something that one can put into
`for` loop).

```bash
closure="$(cat <<-BODY
	touch \"$test_dir/test_file_map\\\$1.tmp\"
BODY
)"

args=("a b")

map "$args" "$closure"
```

The result is... well, one should look after it on output streams. If you assume that possible inputs are lists of
strings and results are outputs produced by called closures then you'll get something like poor man's monade (I was to
lazy to check if it actually follows any monadic laws, probably not :P).

### testing

Primitive test framework. Better than nothing but I wouldn't recommend it to anyone :P

```bash
#!/bin/bash

source "closhell_dir/closhell"
test_dir=...

prepare_suite "tests"

before() {
	rm -rf "$test_dir"/test_file_*
}

after() {
	rm -rf "$test_dir"/test_file_*
}

test_case "my test case description" "$(cat <<-TEST
	# given
	...

	# when
	...

	# then
	assert_equal \$? 0 # compare last command exit code to EXIT_SUCCESS
	something_that_returns_exit_0_for_successful_test_non_0_otherwise
TEST
)"

finish_suite
```

## Suggestions

Since bash lacks any actual scoping one could use `()` to create subshells which would contain all newly created
variables, aliases etc for themselves.

```bash
bind create_file apply "\$create_file_closure" # pollutes whole "scope"

(
	bind update_file apply "\$update_file_closure" # contained within scope
)

```

## Advantages

 * easy to understand - in the beginning, later on hell will break loose,
 * job security as no one will be able to edit your scripts - with all the `$` escaping even you won't fully
   understand how your script works,
 * doesn't modify existing bash grammar - only abuses it to the maximum extent,
 * almost looks nice to functional programmers - except it's still full of side effects and mutabilty,
 * best agile practices - no documentation, useless unit tests, iterations (no long-term planning), driven by the market
   need of setting the world ablaze,
 * perfect revenge tool if you are getting fired and they still require you to write few thousands scripts to hold their
   shit together - if you aren't maintaining it no one will.
