

# Function

Function is an object that you can *call*. Basically, it is a box with internal logic that takes a group of inputs (parameters, or arguments) and returns a value as output.

In the previous sections, we have already encountered some built-in functions of R. For example, `is.numeric()` function takes an argument that can be any R object and returns a logical value that indicates whether it is a numeric vector.

In fact, in R environment, everything we *use* is an object, and everything we *do* is a function. Even `<-` and `+` are both functions that takes two arguments. Although they are called *binary operators*, they are not too special compared with other functions.

When we do casual, interactive data analysis, we often do not need to write any function since the built-in functions and those provided by thousands of packages are usually enough. 

However, if we need to repeat some of our logic or process in the data manipulation or analysis, those functions may not fully serve our purpose because they are not designed to meet our specific needs for a particular dataset. Now we need to create our own functions targeting on a specific set of demand.

## Defining a function

It is easy to define a function in R. Suppose we define a function called `add` that simply adds two numbers named `x` and `y` respectively.


```r
add <- function(x,y) {
  x+y
}
```

The syntax `function(x,y)` specifies the signature of the function, that is, the function takes two arguments named `x` and `y`. `{ x+y }` is the function body that contains a series of expressions given `x` and `y`. The value of the last expression determines the value returned by the function unless `return()` is called inside the function. Finally, the function is bonded to the symbol `add` so that we can refer to this function with this symbol in the environment.

Defining such a simple function, or any more complicated functions does not impose any difference from evaluating a vector. The function in R just acts like an object or value.

## Calling a function

Once the function is defined, we can call the function like how we call a function in maths. The calling requires the same syntax: `name(arg1,arg2,...)`.


```r
add(2,3)
```

```
[1] 5
```

The procedure is quite transparent. When we make such a call, R will find out if there is a function named `add` in the environment. Then it will figure out that `add` refers to the function we just defined and creates a local environment in which `x` takes 2 and `y` takes 3. The expression in the function body is then evaluated given the definition of the arguments. Finally, the function returns the value of that expression, `5`.

## Dynamic typing

Functions in R can be very flexible since it is not strongly typed. In other words, the type of inputs are not fixed prior to the calling. Even if the function is originally designed to work for scalar numbers, it is automatically generalized to also work with all vectors for which `+` is meaningful. For example, we can run the following code without any change of the function.


```r
add(c(2,3),4)
```

```
[1] 6 7
```

The example above does not really demonstrate the flexibility from weak-typing because scalar is also a vector in R. A more qualified example is


```r
add(as.Date("2014-06-01"),1)
```

```
[1] "2014-06-02"
```

The function put the two arguments into the expression without any type-checking. The function fails only when `+` is not well defined for the two arguments.


```r
add(list(a=1),list(a=2))
```

```
Error in x + y: non-numeric argument to binary operator
```

## Generalizing a function

Functions are well-defined abstraction of a particular set of logic or process intended for solving some particular problem. We often want a function to adapt to a wider scenario so that we can use it to solve similar problems without writing too many highly specialized functions.

To make a function more widely applicable is called *generalization*. It is very handy to generalize a function in a weakly-typed programming language like R, but it can be error-prone if it is incorrectly implemented.

To make `add` function more general that can handle various primitive algebraic operations, we can define another function called `calc`. This new function accepts three arguments where `x` and `y` are the two vectors, and `type` accepts a character vector  the kind of algebraic operation the user wants to perform.

The following code implements such a function by using *flow control*, which we will cover in the next chapter. In the code, which expression is to be evaluated depends on the value of `type`.


```r
calc <- function(x,y,type) {
  if(type == "add") {
    x+y
  } else if(type == "minus") {
    x-y
  } else if(type == "multiply") {
    x*y
  } else if(type == "divide") {
    x/y
  } else {
    stop("Unknown type of operation")
  }
}
```

Once the function is defined, we can call it by supplying appropriate arguments.


```r
calc(2,3,"minus")
```

```
[1] -1
```

The function automatically works with numeric vectors.


```r
calc(c(2,5),c(3,6),"divide")
```

```
[1] 0.6666667 0.8333333
```

The function is also generalized to work with non-numeric vectors but `+` is well-defined.


```r
calc(as.Date("2014-06-01"),3,"add")
```

```
[1] "2014-06-04"
```

If we supply some invalid arguments like the following


```r
calc(1,2,"what")
```

```
Error in calc(1, 2, "what"): Unknown type of operation
```

no conditions are satisfied so that the expression in the last `else` block will be evaluated. That is a `stop` call which yields an error message and terminate the function immediately.

The functions seems to work fine and consider all possible situations with invalid arguments. It is not true.


```r
calc(1,2,c("add","minue"))
```

```
Warning in if (type == "add") {: the condition has length > 1 and only the
first element will be used
```

```
[1] 3
```

Here we didn't consider the case where `type` is given as a multi-entry vector. The problem occurs when a multi-entry vector is compared with another vector, it will also results in a multi-entry logical vector, which makes it ambiguous to condition on with `if`. Consider what it means by `if(c(TRUE,FALSE))`?

To avoid such ambiguity explicitly, we need to refine the function so that the error will be more informative and transparent. To proceed, we just need to check whether the vector has a length 1.


```r
calc <- function(x,y,type) {
  if(length(type) != 1L) stop("Only a single type is accepted")
  if(type == "add") {
    x+y
  } else if(type == "minus") {
    x-y
  } else if(type == "multiply") {
    x*y
  } else if(type == "divide") {
    x/y
  } else {
    stop("Unknown type of operation")
  }
}
```

Then we retry the trouble-making call and see how the exception is handled by pre-examination of arguments.


```r
calc(1,2,c("add","minue"))
```

```
Error in calc(1, 2, c("add", "minue")): Only a single type is accepted
```

## Default value for argument

A powerful function is often able to accept a wide range of input and meet a variety of demands. In most cases, it also means an increasing number of arguments. 

If each time we have to specify tens of arguments for a powerful function, it would certainly be a mess. In this case, reasonable default values for arguments will largely simplify the code to call a function.

The set the default value of an argument, use `arg=value` like the following example:


```r
increase <- function(x,y=1) {
  x+y
}
```

The newly defined function `increase` allows us to call it with only `x`. In this case, `y` automatically takes the value 1 unless it is explictly specified.


```r
increase(1)
```

```
[1] 2
```

```r
increase(1,2)
```

```
[1] 3
```

```r
increase(c(1,2,3))
```

```
[1] 2 3 4
```

```r
increase(1,c(2,3,4))
```

```
[1] 3 4 5
```
