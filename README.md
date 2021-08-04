# New-features-in-R-4.1.0

R-4.1.0 is released!

Rejoice! A new R release (v 4.1.0) is due on 18th May 2021. Typically most major R releases don’t contain that many new features, but this release does contain some interesting and important changes.

This post summarises some of the notable changes introduced. More detail on the changes can be found at the R changelog.
Declining support for 32-bit Windows

The 4.1.x series will be the last to support 32-bit Windows systems. This won’t affect most people, but if you have an old laptop you use for occasional R fun, then this might cause an issue.
New syntax

The stability of the base packages is a great strength of the R ecosystem: both underpinning, and contrasting with, the rapid pace at which contributed packages (CRAN, BioConductor) evolve.

Imagine introducing a new feature into the R language. Even if problems arise with the usability of that feature, it would need to be maintained until at least the next major release, by which time thousands of developers and analysts may depend upon it. Unsurprisingly, the R maintainers are exceedingly cautious when introducing new syntax.

Similarly, you should employ caution when using new syntax in your own code. If you do use syntax that was introduced in R-4.1, be aware that your code will not run on versions of R that precede this. For example, this may prevent your new analysis scripts from running on your colleague’s computer, or prevent users from installing your new package.

Do you use RStudio Pro? If so, checkout out our managed RStudio services
The native pipe

When the {magrittr} package introduced the pipe to R in 2014, it was building upon similar syntax in Unix / linux scripting languages (bash) and functional programming languages (like F#), and aimed to

    “decrease development time and improve the readability and maintainability of code” Bache, 2014.

To illustrate the value of pipeline-syntax, here is some code that involves nested function calls:

# Make a density plot of a sampled distribution:
plot(
  density(
    rnorm(100, mean = 4, sd = 1)
  )
)

You might rewrite it in separate steps, to better distinguish the order in which the functions are called (rnorm(), and then density() and then plot()):

# Make a density plot of a sampled distribution
raw_data = rnorm(100, mean = 4, sd = 1)
density_summary = density(raw_data)
plot(density_summary)

But, unless you plan to reuse the samples (raw_data) or their summary (density_summary), code like this may introduce many unnecessary variables into your R environment. The {magrittr} pipe provides a nice middle ground, that limits the use of unnecessary variables and maintains the natural sequence of function-calls in the written code:

# R 4.0.5
library("magrittr")
rnorm(100, mean = 4, sd = 1) %>%
  density() %>%
  plot()

R 4.1.0 introduces a pipe operator |> into the base R syntax

# R 4.1.0
rnorm(100, mean = 4, sd = 1) |>
  density() |>
  plot()

The new operator is nicer to type, and should be both more efficient and easier to debug, than the {magrittr} pipe.
The native pipe vs {magrittr}

Note that, |> is not a drop-in replacement for all uses of %>%. The {magrittr} pipe allowed function-calls to be written with- or without parentheses:

# With parentheses:
letters %>% head()
# [1] "a" "b" "c" "d" "e" "f"

# Without parentheses:
letters %>% head
# [1] "a" "b" "c" "d" "e" "f"

With the native pipe the parentheses must be present:

# R 4.1.0: Make sure your parentheses are present:
letters |> head()
# [1] "a" "b" "c" "d" "e" "f"

The {magrittr} pipe allowed the caller to use the “piped-in” values anywhere in the function call. The piped-in value is represented using a dot (.) as a place holder. For example, you could use those values as the second argument:

# R 4.0.5: Is the string "at" found in any of the animals?
c("dogs", "cats", "rats") %>% grepl("at", .)
# [1] FALSE  TRUE  TRUE

No place holder is provided with the native pipe. To achieve the same result using the native pipe, you would need to define a function. This can be done explicitly with a predefined function:

# R 4.1.0
find_at = function(x) grepl("at", x)
c("dogs", "cats", "rats") |> find_at()
# [1] FALSE  TRUE  TRUE

… or implicitly using an in-place function definition:

# R 4.1.0: make sure you remember the parentheses on the RHS!
c("dogs", "cats", "rats") |>
  {function(x) grepl("at", x)}()
# [1] FALSE  TRUE  TRUE

Shorthand syntax for anonymous functions

Functions are typically defined in R using the syntax

do_something = function(parameters) {
  ...
}

The function that results can be used as follows:

do_something(args)

Functions may also be defined anonymously, and this is common practice when using higher-order functions such as those in the {purrr} package (map(), reduce(), keep(); and their base-R counterparts: Map(), lapply() etc).

# For each letter,
#   - find the name of each dataset in the {datasets} package
#   that starts with that letter
# Using {tidyverse} syntax
purrr::map(
  letters[1:3],
  function(x) {
    ds = ls("package:datasets")
    ds[stringr::str_starts(tolower(ds), x)]
  }
)
#[[1]]
#[1] "ability.cov"   "airmiles"      "AirPassengers" "airquality"
#[5] "anscombe"      "attenu"        "attitude"      "austres"
#[[2]]
#[1] "beaver1"      "beaver2"      "BJsales"      "BJsales.lead" "BOD"
#[[3]]
#[1] "cars"        "ChickWeight" "chickwts"    "co2"         "CO2"
#[6] "crimtab"

# In base R
Map(
  function(x) {
    pattern = paste0("^", x) # eg "^a" to match a leading 'a'
    grep(pattern, ls("package:datasets"), value = TRUE, ignore.case = TRUE)
  },
  letters[1:3]
)

In this setting the function(x) {...} syntax can look rather verbose, especially when chaining together several functions into a pipeline. A new syntax has been introduced into R that may make these anonymous function declarations more succinct:

# R 4.1.0
Map(
  \(x) {
    pattern = paste0("^", x)
    grep(pattern, ls("package:datasets"), value = TRUE, ignore.case = TRUE)
  },
  letters[1:3]
)

With this syntax, the final example in the native-pipe section (above) could be rewritten in this manner:

# R 4.1.0
c("dogs", "cats", "rats") |>
  {\(x) grepl("at", x)}()

Minor changes
Combining factors

In earlier versions of R, if two factors were combined together using c(factor1, factor2), the result was an integer vector:

# R 4.0.5: Combining factors with different levels
c(factor("a"), factor("b"))
# [1] 1 1

# R 4.0.5: Combining factors with identical levels
c(factor("a"), factor("a"))
# [1] 1 1

This happens because factors are stored internally as integers. In R >= 4.1, the results are a little more sensible. Combining two factors together:

    generates a new factor,
    with levels that are a combination of the levels in the original factors.

fac1 = factor(c("a", "b", "d"))
fac2 = factor(c("b", "c"))
c(fac1, fac2)

# [1] a b d b c
# Levels: a b d c

Note that this is consistent with how forcats::fct_c() combines factors together.
Splitting data-frames using formula syntax

A nice shorthand has been introduced for splitting data-frames that uses formula syntax. To do this, you originally had to duplicate the name of the data-frame in the call to split().

# R 4.0.5: Separate the car models in 'mtcars' by their number of gears
split(mtcars, mtcars$gear)

When used in a {magrittr} pipeline, that syntax looks rather clunky:

# R 4.0.5
# Get the car-model with the highest 'mpg' for cars with 3, 4, 5 gears separately
mtcars %>%
  split(., .$gear) %>%
  lapply(function(x) x[which.max(x$mpg), ])

Combining the new pipeline syntax and the function shorthand together with the new syntax for split, the latter can be rewritten as follows:

# R 4.1.0
mtcars |>
  split(~ gear) |>
  lapply(\(x) x[which.max(x$mpg), ])
