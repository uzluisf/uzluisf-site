---
title: "Perl 6 code in rmarkdown"
author: "Luis F. Uceta"
date: "2018-08-13"
draft: false
---

**Note:** In October 2019, Perl 6 was renamed to Raku. Whenever you
come across Perl 6, replace it with Raku. Learn more about the
[path to Raku](https://github.com/Raku/problem-solving/blob/master/solutions/language/Path-to-Raku.md).

## Installation

First, start off by installing the `R` programming language. After this, run `R` and
execute the following command to install `rmarkdown`: `install.packages("rmarkdown")`.
While installing rmarkdown, I got the following error message:

```r
Error: .onLoad failed in loadNamespace() for 'tcltk', details:
  call: dyn.load(file, DLLpath = DLLpath, ...)
  error: unable to load shared object '/usr/lib/R/library/tcltk/libs/tcltk.so':
  libtk8.6.so: cannot open shared object file: No such file or directory
```

This was solved by installing the package `tk` and then proceeding with the
installation of rmarkdown.

## rmarkdown's code chunks

Almost everything you can do with rmarkdown you can do it with "regular" markdown.
However, one of the outstanding features of rmarkdown is its capability to execute
chunks of code and spit back their results. To do this, rmarkdown make use of 
of the [knitr](https://yihui.name/knitr/) package, an engine for dynamic report
generation with R. In addition to R, it also supports other language engines 
which you can use to evaluate code from other languages. To list the name of the
available engines, execute the command `names(knitr::knit_engines$get())` in the `R` 
REPL. Just like "regular" markdown, code chunks can be created with three
back ticks, followed by the code and ending with another three back ticks.
If you want a code chunk to be evaluated, specify the language inside the curly
braces '`{}`', placed after the first three back ticks. For example, to execute 
Perl 5 code, one would specify `perl` inside `{}`:

    ```{perl}
    sub factorial {
        my ($n) = @_;
        return 1 if $n == 0;
        return factorial($n - 1) * $n;
    }

    for (1..4) {
	    print "Factorial of $_: ", factorial($_), "\n";
    } 
    ```

This would be processed as follows, with the result beneath the source
code if any expression was evaluated:

```perl
sub factorial {
    my ($n) = @_;
    return 1 if $n == 0;
    return factorial($n - 1) * $n;
}

for (1..4) {
    print "Factorial of $_: ", factorial($_), "\n";
} 
```

```
## Factorial of 1: 1
## Factorial of 2: 2
## Factorial of 3: 6
## Factorial of 4: 24
```

Options for the code chunk can be specified within the curly braces. For
example, to prevent the evaluation of the code chunk you must set `eval` to
`FALSE`:

    ```{perl, eval=FALSE}
    my $name = 'Nemy';
    print "Hello, $name!\n";
    ```

This prevents the evaluation of the code chunk and only the source code is shown:

```perl
my $name = 'Nemy';
print "Hello, $name!\n";
```

To hide the source code and still display its evaluation, you can set `echo` to `FALSE`:

    ```{perl, echo=FALSE}
    my $name = 'Nemy';
    print "Hello, $name!\n";
    ```

Only the result is displayed:

```
## Hello, Nemy!
```

There is a bunch of options you can specify within the curly braces. 
Here's a small list of the ones I've commonly used:

+ **`engine`** - `R` by default. knitr will evaluate the chunk in the named
  language, e.g. `engine = 'python'`. 
+ **`eval`** - `TRUE` by default. If `FALSE`, knitr will not run the code in the
  code chunk.
+ **`echo`** - `TRUE` by default. If `FALSE`, knitr will not display the code in the
  code chunk above its result in the final document.
* **`collapse`** - `FALSE` by default. If `TRUE`,  knitr will collapse all the
  source and output blocks created by the chunk into a single block.
* **`comment`** - `##` by default. A character string which knitr will append to the
  start of each line of results in the final document, e.g. `comment = '#=>'`.

Find more information about the [different options here](https://yihui.name/knitr/options/).


## Perl 6 in rmarkdown

As of now, knitr doesn't support a language engine for Perl 6. However, you can
use knitr's engine language extensibility to execute Perl 6 code chunks in
rmarkdown by adding the following R code chunk in your file:

    ```{r setup}
    library(knitr)
    eng_perl6 <- function(options) {
      # create a temporary file
      f <- basename(tempfile("perl6", '.', paste('.', "perl6", sep = '')))
      on.exit(unlink(f)) # cleanup temp file on function exit
      writeLines(options$code, f)
      out <- ''
    
      # if eval != FALSE compile/run the code, preserving output
      if (options$eval) {
        out <- system(sprintf('perl6 %s', paste(f, options$engine.opts)), intern=TRUE)
      }
    
      # spit back stuff to the user
      engine_output(options, options$code, out)
    }
    
    knitr::knit_engines$set(perl6=eng_perl6)
    ```
This workaround was taken from
[here](https://stackoverflow.com/questions/45857934/executing-perl-6-code-in-rmarkdown/45864801#45864801)
and
[here](https://www.r-bloggers.com/running-go-language-chunks-in-r-markdown-rmd-files/).

This will allow you to do this:

    ```{r, engine='perl6'}
    say [*] 1..5;
    ```

However, instead of adding that piece of R code in every file where Perl 6 code is 
being processed, you can make use of the engine language for Perl 5 and change
its path to the Perl 6 executable by setting `engine.path` to `perl6`. 
Then, you can process Perl 6 code by making the chunk header look as follows: 

    ```{perl, engine.path='perl6'}
    say [*] 1..5;
    ```

Now you should be able to evaluate Perl 6 code chunks:

    ```{perl, engine.path='perl6', comment='#=>', collapse=TRUE}
    class Point {
        has $.x = 0;
        has $.y = 0;
        method distance-to-center() {
            return sqrt($!x**2 + $!y**2);
        }
    }

    my $p = Point.new(x => 3, y => 4);
    say $p.distance-to-center();
    ```

would be processed as follows:

```perl6
class Point {
    has $.x = 0;
    has $.y = 0;
    method distance-to-center() {
        return sqrt($!x**2 + $!y**2);
    }
}

my $p = Point.new(x => 3, y => 4);
say $p.distance-to-center();

#=> 5
```


## Addendum

This post was inspired from my desire for having an easy setup 
to transcribe lecture notes using vim. The notes might be sparkled with LATEX,
some code snippets, and converted to .pdf file for reviewing along the way. 
rmarkdown's easiness of use and extensive capabilities make it the right tool
for this job.

To do the conversion from a .rmd to a .pdf file directly from vim, I have
the following line, mapped to F5, in my .vimrc to ease the process:

```vim
autocmd Filetype rmd map <F5> :!echo<space>"require(rmarkdown);
<space>render('<c-r>%')"<space>\|<space>R<space>--vanilla<enter>
```
As you can see, this is just loading the rmarkdown package in `R`, passing the
current .rmd file to the function `render` and piping its result to `R --vanilla`
to make the program non-interactive.

Note that you might need to have `pandoc` installed for this to work. As for the
pandoc latex template, I use a slightly modified version of
[eisvogel.latex](https://github.com/Wandmalfarbe/pandoc-latex-template).

This is a sample of what the YAML preamble (for metadata) might look like
for any of the notes:

```yaml
---
title: "Perl 6 in rmarkdown"
author: "Luis F. Uceta"
date: Aug 13, 2018

output: 
    pdf_document:
        latex_engine: pdflatex
        toc: true
        template: eisvogel.latex

fontsize: 12pt
geometry: margin=1in 
linkcolor: red
urlcolor: blue
---
```

And this is the [resulting pdf](https://www.scribd.com/document/386099184/2018-08-13-rmarkdown-vim-perl6#) for this file with the previous YAML preamble
and using the mentioned latex template. Please note that this pdf might not reflect
the latest changes made to this post.

