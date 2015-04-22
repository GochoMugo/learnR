

# Apply-family functions

Previously, we talked about using `for` to repeat evaluating an expression with an iterator on a vector or list. In practice, however, `for` loop is almost the last choice because an alternative way is much cleaner and easier to write and read when each iteration is independent with each other.

For example, the following code uses `for` to create a list of 3 independent normally distributed random vectors whose length is specified by vector `len`.


```r
len <- c(3,4,5)
# first, create a list in the environment.
x <- list()
# then use `for` to generate the random vector for each length
for(i in 1:3) {
  x[[i]] <- rnorm(len[i])
}
x
```

```
[[1]]
[1] -0.50219235  0.13153117 -0.07891709

[[2]]
[1]  0.8867848  0.1169713  0.3186301 -0.5817907

[[3]]
[1]  0.71453271 -0.82525943 -0.35986213  0.08988614  0.09627446
```

The above example is simple but the code is a bit redundant compared to the implementation with `lapply`.


```r
lapply(len,rnorm)
```

```
[[1]]
[1] -0.2016340  0.7398405  0.1233795

[[2]]
[1] -0.02931671 -0.38885425  0.51085626 -0.91381419

[[3]]
[1]  2.3102968 -0.4380900  0.7640606  0.2619613  0.7734046
```

The `lapply` version is much simpler: it applies `rnorm()` on each element in `len`, and put each result into a list. 

From the example above, we should realize that it is only possible if R allows us to pass functions as ordinary objects. Fortunately, it is true: Functions in R are treated just like objects and can be passed around as arguments. This feature largely boosts the flexibility of coding.

Each apply-family function is a so-called *higher-order function* that accepts function as an argument. We will cover the this concept in detail later.

## lapply

`lapply`, as we previously demonstrated, takes a vector and a function as its arguments. It simply applies the function to each element in the given vector, and finally returns a list that contains all the results.

This function is useful when each iteration is independent with each other. In this case, we don't have to explicitly create an iterator.

It works with not only vectors but also lists. Suppose we have a list of students,


```r
students <- list(
  a1=list(name="James",age=25,gender="M",interest=c("reading","writing")),
  a2=list(name="Jenny",age=23,gender="F",interest=c("cooking")),
  a3=list(name="David",age=24,gender="M",interest=c("running","basketball")))
```

and we need to create a character vector in which each element is formatted as

```
James, 25 year-old man, loves reading, writing.
```

First, we are sure that an iteration is to work on `students`, and each is independent. In other words, the computation for James has nothing to do with that for Jenny, and etc. Therefore, we can use `lapply` to do the work.


```r
lapply(students, function(s) {
  type <- switch(s$gender,"M"="man","F"="woman")
  interest <- paste(s$interest,collapse = ", ")
  sprintf("%s, %d year-old %s, loves %s.",s$name,s$age,type,interest)
})
```

```
$a1
[1] "James, 25 year-old man, loves reading, writing."

$a2
[1] "Jenny, 23 year-old woman, loves cooking."

$a3
[1] "David, 24 year-old man, loves running, basketball."
```

The code above uses an *anonymous function* which is a function that is not assigned to a symbol. In other words, the function is only temporal and has no name. Of course, we can explicitly bind the function to a symbol, that is, give it a name, and use that name in `lapply`.

Despite this, the code is quite straightforward. For each element `s` in `students`, the function decides the type of the student, and pastes his or her interests together separated by comma, and put the information in a certain format we want.

Fortunately, a major part of how we use `lapply` also works with other apply-family functions, but their iterating mechanism or the type of results may be different.

## sapply

List is not always a favorable container of the results. Sometimes we want the results to be put in a simple vector or a matrix. `sapply` simplifies the result according to its structure.

Suppose we apply a square on each element of `1:10`. If we do it with `lapply`, we will have a list of squared numbers. This result looks a bit heavy and redundant because the resulted list is actually a list of single-valued numeric vectors. However, we might want to keep the results still as a vector.


```r
sapply(1:10, function(i) i^2)
```

```
 [1]   1   4   9  16  25  36  49  64  81 100
```

If the applying function returns a multi-element vector each time, `sapply` will put the results into a matrix in which each returned vector occupies a column.


```r
sapply(1:10, function(i) c(i,i^2))
```

```
     [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
[1,]    1    2    3    4    5    6    7    8    9    10
[2,]    1    4    9   16   25   36   49   64   81   100
```

## vapply

`sapply` is very handy and smart, but sometimes the smartness may become a downside risk. Suppose we have a list of input numbers. 


```r
x <- list(1, 5, 2, c(2,3), 4)
```

If we want to get a numeric vector of the squared numbers for each number in `x`, `sapply` can be easy to use.


```r
sapply(x, "^", 2)
```

```
[[1]]
[1] 1

[[2]]
[1] 25

[[3]]
[1] 4

[[4]]
[1] 4 9

[[5]]
[1] 16
```


`vapply` is the safer version of `sapply`. 


```r
vapply(1:10, function(i) c(i,i^2), numeric(2))
```

```
     [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
[1,]    1    2    3    4    5    6    7    8    9    10
[2,]    1    4    9   16   25   36   49   64   81   100
```

## mapply

`mapply` is a multivariate version of `sapply`.


```r
mapply(function(a,b,c) a*b + b*c + a*c, 
  a=c(1,2,3), b=c(5,6,7), c=c(-1,-2,-3))
```

```
[1] -1 -4 -9
```


```r
df <- data.frame(x=c(1,2,3),y=c(3,4,5))
mapply(function(xi,yi) c(xi,yi,xi+yi), df$x, df$y)
```

```
     [,1] [,2] [,3]
[1,]    1    2    3
[2,]    3    4    5
[3,]    4    6    8
```


`Map` is the multivariate version of `lapply`.


```r
Map(function(xi,yi) c(xi,yi,xi+yi), df$x, df$y)
```

```
[[1]]
[1] 1 3 4

[[2]]
[1] 2 4 6

[[3]]
[1] 3 5 8
```


## rapply

`rapply` is a recursive version of `lapply`.



## tapply

`tapply`

## apply


```r
mat <- matrix(c(1,2,3,4),nrow = 2)
mat
```

```
     [,1] [,2]
[1,]    1    3
[2,]    2    4
```

`apply` applies a function on a given *margin*, or *dimension* of a given object. For example, to calculate the sum of each row, which is the first dimension, we need to specify `MARGIN=1` so that `sum` is applied to a row (numeric vector) sliced from `mat` in each iteration.


```r
apply(mat,1,sum)
```

```
[1] 4 6
```

To calculate the sum of each column, which is the second dimension, we need to specify `MARGIN=2` so that `sum` is applied to a column sliced from `mat` in each iteration.


```r
apply(mat,2,sum)
```

```
[1] 3 7
```

`apply` also supports array input and matrix output. 


```r
mat2 <- matrix(rnorm(25),nrow = 5)
mat2
```

```
           [,1]        [,2]       [,3]       [,4]        [,5]
[1,] -0.8143791  0.24707599 -0.6900143  0.9702020 -0.52228335
[2,] -0.4384506 -0.09111356 -0.2217942 -0.1016292  1.32223096
[3,] -0.7202216  1.75737562  0.1829077  1.4032035 -0.36344033
[4,]  0.2309445 -0.13792961  0.4173233 -1.7767756  1.31906574
[5,] -1.1577295 -0.11119350  1.0654023  0.6228674  0.04377907
```

To build a matrix that shows the max and min for each column, we can call


```r
apply(mat2,2,function(col) c(min=min(col),max=max(col)))
```

```
          [,1]       [,2]       [,3]      [,4]       [,5]
min -1.1577295 -0.1379296 -0.6900143 -1.776776 -0.5222834
max  0.2309445  1.7573756  1.0654023  1.403203  1.3222310
```
