# Basic Expressions and Assignments
## Expressions
Curly, like most programming languages, supports commonly used infix operators like `*` and `+`. Try it out in the repl!
```
>>>
2 - 3
-1
>>>
2 * 3.3 + 4
10.60000
>>>
2 * (3.3 + 4)
14.60000
>>>
true xor true
false
>>>
true and true
true
>>>
true or false
true
```

## Assignments
Like most programming languages, Curly supports assigning variables. Unlike many programming languages, however, all variables are immutable by default.
```
>>>
my_awesome_variable = 3.14159
3.14159
>>>
my_awesome_variable = 2.71828
error: Redefinition of previously declared variable
  ┌─ <stdin>:1:1
  │
1 │ my_awesome_variable = 2.71828
  │ ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  │ │
  │ `my_awesome_variable` is already defined and not declared as mutable
  │ `my_awesome_variable` previously defined here
```

Note: Currently, mutability is illegal, but in the future, mutable variables will be able to be declared.

You can also declare variables with types:
```
>>>
truthy: Bool = true
true
>>>
my_float: Float = 0.618
0.61800
>>>
amazing_int: Int = 42
42
>>>
amazing_int' = 3
3
```

Let's see what happens if we try to declare a variable whose value does not match the given type:
```
>>>
error: Int = true
error: Nonmatching types in assignment
  ┌─ <stdin>:1:14
  │
1 │ error: Int = true
  │        ---   ^^^^ Expected `Int`, got `Bool`
  │        │
  │        Assignment is declared with type `Int`
```

Oh no, we get a big scary error message! The Curly compiler makes sure you provide the type that matches the value assigned to the variable. Most of the time, you don't need to provide a type; however, sometimes it's useful to provide a type in the assignment, as we will see later on.

## With expressions
If you want to define a local scope, you can use a with expression! Here is an example:
```
>>>
with x = 2 * 3,
	y = 4 * 3,
	x*x + y*y
180
```

Note that any variables defined within a with expression are not available outside of the with expression:
```
>>>
with x = 2, x
2
>>>
x
error: Symbol not found
  ┌─ <stdin>:1:1
  │
1 │ x
  │ ^ Could not find symbol `x`
```
