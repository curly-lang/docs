# Union Types and Sum Types
## Union types
Static typing is pretty neat, but what if you want multiple types to be passed in? For example, what if you wanted to make a function that takes in either an integer, a float, or a boolean? Well, this is where union types come in! Here's a simple example:
```
>>>
id x: Int | Bool | Float = x
<func 0x7f6b5721b370> : Int | Bool | Float -> Int | Bool | Float
>>>
id 2
2 : Int | Bool | Float
>>>
id 3.4
3.40000 : Int | Bool | Float
>>>
id false
false : Int | Bool | Float
>>>
```

Whoa, we have a function that takes in any of an integer, a float, or a boolean and outputs it back to us! That's pretty neat! But what's up with the returned value? Well, since `id` can return any of `Int`, `Float`, or `Bool`, the value must be of the type `Int | Float | Bool`. What happens if we try to do something to a return value of `id`?
```
>>>
(id 2) + 3
error: Undefined infix operator
  ┌─ <stdin>:1:2
  │
1 │ (id 2) + 3
  │  ^^^^^^^^^ `Add` is undefined on `Int | Bool | Float` and `Int`
```

Oh no, we get an error! This is because the return value of `id 2` is wrapped in a union type. To unwrap it, we must use a `match` expression.

## Match expressions
Let's look at the following code:
```
match id 2
to i: Int => i + 3
to f: Float => f / 3.2
to b: Bool => b xor true
5
```

As you can see, the `match` expression breaks up a union type into separate cases for each subtype and binds a variable to them. The syntax `a: Type` is used to state the type of a value. You can do this anywhere, not just in assignments and `match` expressions. For example, this is valid:
```
>>>
id (2 : Int | Float)
2 : Int | Bool | Float
```

Using `:` in this way isn't necessary most of the time. Curly automatically casts values whenever needed; for example, when you pass in an `Int` to `id`, `id` actually expects a value of the type `Int | Float | Bool` instead of taking in any of them. What Curly is actually doing under the hood is something like this:
```
id (2 : Int | Float | Bool)
```

Anyway, back to `match` expressions. We also don't have to specify the variable name if we don't care about the value. Here's an example:
```
>>>
match id 2
to Int => 42
to Float => 0
to Bool => 0
42
```

Currently, there is no need to specify all subtypes of a union type. This is temporary, and doing this will cause undefined behaviour when you run into a case that isn't specified at the moment. In the future, an error will be thrown if you do not specify all subtypes of a type.

## Type aliases
Do we have to write `Int | Float | Bool` every single time we want to use this type? Of course not! We can use something called a type alias! Here's how we define one:
```
type IFB = Int | Float | Bool
```

As you see, it's similar to assignment, with the extra keyword `type` in the front to tell Curly that it's not a regular assignment but a type alias. We can then use this type as usual:
```
>>>
id' x: IFB = x
<func 0x7f4e5814e370> : IFB -> IFB
```

Curly still does automatic casting for us! We don't need to do `id (2: IFB)` because Curly knows that `IFB` is actually just an `Int | Float | Bool`, and it knows how to deal with regular union types.

## Sum types
Sum types are pretty neat. They're an extension of union types that let you label the subtypes used. They also let you include the same type multiple times. For example, let's say we want to make a safe division function for floats. We want it to return a `Float` if the division was successful and a `Float` with the denominator if the division was not. Let's start off by defining a type alias:
```
>>>
type DivResult = Float | Float
error: Duplicate type in union type declaration
  ┌─ <stdin>:1:26
  │
1 │ type DivResult = Float | Float
  │                  -----   ^^^^^ Type `Float` used a second time here
  │                  │
  │                  Type used here first
```

Oh no, what did we do wrong?! Well, union types only let you have one of each type, so the type `Float | Float` is invalid. What can we do to fix this problem? Well, we can instead use sum types! Here's how we define a sum type:
```
type DivResult = Ok: Float | Err: Float
```

Cool! Now let's define a function that safely divides two numbers!
```
>>>
safeDiv a: Float, b: Float =
    if b == 0 or b != b then
        DivResult::Err b
    else
        DivResult::Ok (a / b)
<func 0x7f0aba15f490> : Float -> Float -> Err: Float | Ok: Float
```

Don't worry about the weird `b != b` thing; that is just to test for `NaN`. We can then apply this function just as normal:
```
safeDiv 2.3 5.6
{ Ok = 0.41071 } : Err: Float | Ok: Float
>>>
safeDiv 4.5 0.0
{ Err = 0.00000 } : Err: Float | Ok: Float
```

This time, the value returned is of the form `{ label = value }`. To unwrap this, we use a match expression once again, but slightly differently:
```
>>>
match safeDiv 4.5 0.0
to v: (Ok: Float) => v
to _: (Err: Float) => 0
0
```

Note that you cannot convert to a sum type using `:`. This is what happens if you try:
```
>>>
2 : DivResult
error: Invalid cast
  ┌─ <stdin>:1:5
  │
1 │ 2 : DivResult
  │ -   ^^^^^^^^^ Cannot convert `Int` to `DivResult`
  │ │
  │ Value has type `Int`
```

This is because it doesn't know which `Int` type to convert to, so instead of trying to infer it and possibly getting the wrong one, it just crashes. You must always use `Type::Subtype` to create a sum type. Speaking of which, what's the type of `DivResult::Ok`?
```
>>>
DivResult::Ok
<func 0x7f0aba150390> : Float -> DivResult
```

Whoa it's just a function! This means you can actually pass it into other functions if you want!

## Enums
Sometimes, you want a value in a sum or union type that doesn't actually hold a value, but instead represents a possible option. For example, in Haskell there's the `Maybe` type, and in Rust there's the `Option<T>` type, since neither of these support `null` or `nil` values. Similarly, Curly does not support a `null` or `nil` value. So how would you define an `Option` type? Well, you use an enum!
```
type Option = Some: Int | enum None
```

We can make an instance of this enum using `Option::None` and match it using `enum None`. For example:
```
>>>
match Option::None
to s: (Some: Int) => s
to enum None => 0
0
```
