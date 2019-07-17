# R Markdown Support

<mark>***This document is a work in progress***</mark>

## Philosophy of R Markdown Integration
There are many scenarios we can envision with users who wish to integrate R Markdown with a Word document using StatTag.  Primarily though, we are assuming that one or more R Markdown documents were created for intermediate reporting to the research study team, and the user wishes to repurpose the statistical code for a manuscript.

We are aware of the following user scenarios, and indicate which we support within StatTag.

| Scenario | Supported | Description |
| ---------| --------- | ----------- |
| Integrated| :white_check_mark: Yes | The user sees the RMarkdown file as a source file, and has enough analysis work done in it that they don't want to duplicate the work.  They are planning to integrate it with their manuscript, and any changes to their R Markdown code must be reflected within the manuscript as well. |
| Branched | :x: No | the user sees the code used in the manuscript as a branch of the R Markdown code, and either needs them to deviate or is okay if they deviate. |



## Chunks, Inline Code and Tags
A central part of StatTag is having a uniquely named tag within a code file, with the tag having a clearly defined boundary from wich we know to capture the tag output.  This paradigm changes slighly as we consider markdown-based documents.

With R Markdown, we may have *optionally* named code chunks

````
```{r my_optional_tag}
x <- 5000
print(x)
```
````

or anonymous inline code

````
`r print(x)`
````

We need to ensure StatTag has a way to unqiuely identify pieces of code, so that a field inserted in a Word document can be traced back to the code file.

<mark>StatTag will enforce the convention that inline code is either a Value or Verbatim type.  Code blocks can be any StatTag tag type.</mark>

## Tag Properties
StatTag allows users to specify formatting options for values and tables.  <mark>For R Markdown integration, we have determined that users should perform all formatting (e.g., specify number of digits, suppress table columns) within the R Markdown code.</mark>

## Figures
What to do with fig.opts setting paths, etc. for the figures?

## Patterns
For reference, the list of R Markdown patterns are at: [https://github.com/yihui/knitr/blob/master/R/pattern.R](https://github.com/yihui/knitr/blob/master/R/pattern.R)

## Markdown Options
R Markdown (via knitr) allows the user to [set some options](https://yihui.name/knitr/options/#chunk_options) for how things are rendered.  For example, the user can specify the height and width of a figure.  A challenge for StatTag then is interpreting those parameters and ensuring they are appropriately applied.  This includes considering both global options, as well as individual options on code chunks.

### Displaying Code
<mark>StatTag will never insert the code chunk in a document, even if the R Markdown is explicitly configured to do this (RMD `echo` option)</mark>

### Displaying Output
<mark>StatTag will always display the output of a code chunk in a document (if it is tagged and inserted by the user)</mark>

## Example StatTag-Enabled R Markdown
### Code Chunks
Named code chunk

````
```{r chunk_name, include=FALSE}
x <- rnorm(100)
y <- 2*x + rnorm(100)
cor(x, y)
```
````

<mark>Anonymous code chunk - built-in StatTag comment support</mark>

````
```{r}
##>>>ST:Table(Label="tag_name")
x <- rnorm(100)
y <- 2*x + rnorm(100)
cor(x, y)
##<<<
```
````

<mark>Anonymous code chunk - custom RMD options for StatTag</mark>

````
```{r STLabel="tag_name"}
x <- rnorm(100)
y <- 2*x + rnorm(100)
cor(x, y)
```
````

### Inline Code
````
`r print(x) ##>>>ST:Value(Label="Value Tag")`
````

````
`r print(x) ##>>>ST:Verbatim(Label="Verbatim Tag")`
````