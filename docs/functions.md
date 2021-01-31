# Functions
## Functions with one argument
You can declare functions very simply using Haskell-like syntax:
```
>>>
id x: Int = x
(Int -> Int) <func 0x7f7b1c34f360>
>>>
double x: Float = x + x
(Float -> Float) <func 0x7f7b1c354360>
```

Note that arguments must be annotated with their type; however, the return type is implied. We can then apply the function as so (note the absence of parentheses):
```
>>>
id 42
42
>>>
double 23.596
47.19200
```

As you probably can expect, applying an argument of an incompatible type to a function gives an error:
```
>>>
double true
error: Wrong type passed as an argument
  ┌─ <stdin>:1:8
  │
1 │ double true
  │        ^^^^ Expected `Float`, got `Bool`
```

Note that `Int` and `Float` are incompatible types at the moment. In the future, automatically casting certain types will be allowed.

## Functions with multiple arguments
Functions with multiple arguments have their arguments separated by commas. For example:
```
>>>
sum x: Int, y: Int = x + y
(Int -> Int -> Int) <func 0x7f7b1c34a370>
>>>
product x: Float, y: Float, z: Float = x * y * z
(Float -> Float -> Float -> Float) <func 0x7f7b1c345370>
```

We can apply them quite similarly:
```
>>>
sum 5 4
9
>>>
product 5.3 1.5 0.27
2.14650
```

Note the lack of parentheses or commas.

## Currying / partial application
Here's an interesting question: why is the type of a function that takes in multiple arguments `A -> B -> C` instead of something like `(A, B) -> C`? This is because of something powerful called *currying*. Let's take a look at the product function we introduced earlier:
```
product x: Float, y: Float, z: Float = x * y * z
```

What happens if we call this function with only one argument? Well, let's try it out in the repl!
```
>>>
product 4.5
(Float -> Float -> Float) <func 0x7f7b1c345370>
```

Interesting. Instead of getting an error, we get back a function, except this function is missing one argument in its type signature! This is called currying. By applying only some of the arguments to the function, we curry the function.

We can assign this new function to another variable and get a new function that we can apply:
```
>>>
product_4_5 = product 4.5
(Float -> Float -> Float) <func 0x7f7b1c345370>
>>>
product 4.5 6.0 7.0 == product_4_5 6.0 7.0
true
```

Here's a more complex example. Let's say we want to create a function that calculates the squared distance between two points in a plane. We can define a function like so:
```
>>>
distSqrd x1: Float, y1: Float, x2: Float, y2: Float =
	with dx = x2 - x1,
		dy = y2 - y1,
		dx * dx + dy * dy
(Float -> Float -> Float -> Float -> Float) <func 0x7f7b1c0f7380>
```

This is pretty neat, but what if we want to define a function that calculates the distance from the origin? Well, we could define a second function like so:
```
distToOriginSqrd x: Float, y: Float =
	distSqrd 0.0 0.0 x y
```

But this could be made better. We can rewrite this to take advantage of our newfound knowledge of currying!
```
>>>
distToOriginSqrd = distSqrd 0.0 0.0
(Float -> Float -> Float) <func 0x7f7b1c0f7380>
```

Then we can use this just like a regular function!
```
>>>
distToOriginSqrd 3.0 4.0
25.00000
```

## Lambda functions
Lambda functions are cool. If you don't know what lambda functions are, they are just anonymous functions. You can define them like so:
```
>>>
lambda x: Float = x + x
(Float -> Float) <func 0x7f7b1c0e3340>
```

Do not confuse this with assigning to a function called `lambda`! `lambda` is a keyword, and as such, this will not create a function called lambda! In fact, if you try to use `lambda` as a function, you will get a syntax error:
```
>>>
lambda 2.3
error: Unexpected `lambda`
  ┌─ <stdin>:1:1
  │
1 │ lambda 2.3
  │ ^^^^^^
```

So what happened to this new function we just created? Well, we didn't do anything with it, so now it's gone forever. We can apply a lambda function directly to an argument if we want:
```
>>>
(lambda x: Float = x + x) 3.4
6.80000
```

Or we could assign it to a variable:
```
>>>
double' = lambda x: Float = x + x
(Float -> Float) <func 0x7f7b1c0de360>
>>>
double' 0.5
1.00000
```

### Closures
Closures are pretty neat too. They let you capture local variables. Here's an example:
```
func = with x = 2,
	lambda y: Int = x + y
```

Here, `func` captures the local variable `x`. If we try to call `func` later on, we get this:
```
>>>
func 3
5
```

Amazing right!

## Church numerals
Church numbers are really cool. They're a representation of numbers using only functions, and because of partial application, we can do this in Curly! Here's with partial application:
```
>>>
func_to_int n: (Int -> Int) -> Int -> Int =
	n (lambda x: Int = x + 1) 0
(((Int -> Int) -> Int -> Int) -> Int) <func 0x7f52d7b223e0>
>>>
succ n: (Int -> Int) -> Int -> Int, f: Int -> Int, x: Int =
	f (n f x)
(((Int -> Int) -> Int -> Int) -> (Int -> Int) -> Int -> Int) <func 0x7f52d7b1d390>
>>>
zero f: Int -> Int, x: Int = x
((Int -> Int) -> Int -> Int) <func 0x7f52d7b18380>
>>>
func_to_int (succ zero)
1
>>>
func_to_int (succ (succ (succ zero)))
3
```

And here's with closures:
```
>>>
func_to_int n: (Int -> Int) -> Int -> Int =
	with to_int x: Int = x + 1,
		n to_int 0
(((Int -> Int) -> Int -> Int) -> Int) <func 0x7f575f9c43e0>
>>>
succ n: (Int -> Int) -> Int -> Int =
	with next f: Int -> Int, x: Int = f (n f x),
		next
(((Int -> Int) -> Int -> Int) -> (Int -> Int) -> Int -> Int) <func 0x7f575f9bf3d0>
>>>
zero f: Int -> Int, x: Int = x
((Int -> Int) -> Int -> Int) <func 0x7f575f9ba380>
>>>
func_to_int (succ zero)
1
>>>
func_to_int (succ (succ (succ zero)))
3
```

As you can see, currying and closures are incredibly powerful. Although you likely won't be using numbers from only functions, you will still be using currying and closures often enough in your programs that it'll be important to know what they are.
