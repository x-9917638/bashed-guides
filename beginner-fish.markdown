# Writing Scripts with Fish

This guide will teach you through the basics of the fish scripting language.
Together, we will write a simple CSV parsing and pretty-printing utility!!

> [!TIP]
>
> - Fish has very thorough documentation, which can be found
>   [here][1]
> - If you follow this guide 1 to 1, i.e. copy paste all sample code without
>   adding anything new, you will not be able to ship the project.
> - If you have complaints, suggestions, or otherwise need help, you can
>   contact me on Slack: @Some1 (U091HKKKJRH)

## Contents
* [Setup](#setup)
* [Basic Concepts](#basic-concepts)
* [Project](#the-project)
* [Command Line Interface](#command-line-interface)
* [Parsing with `argparse`](#parsing-with-argparse)
* [Exercise 1](#exercise-1)

## Setup

We will need a few dependencies. Of course, we need the **fish shell** to run
the script, which you can install
[here][2]. We also need an
implementation of POSIX commands, which should already be installed.

We will also need a terminal (emulator), which should already be installed. On
Windows, this should the terminal you installed fish through, either Cygwin,
MSYS2, or WSL. On mac, this should be the Terminal app. For those on other
operating systems, you probably already know what your terminal is.

Finally, we need a CSV file to parse. I've taken a sample CSV file from the site
samplelib.com, and it can be downloaded [here][3].

## Basic Concepts

> [!TIP]
> 
> - You can test things in interactive mode by running `fish -i`.
> - You can get help about a command at any time by running `help [COMMAND]` or `man [COMMAND]`. You can also search manual pages through `apropos [KEYWORD]`.

Here's a glossary of some terms I'll be mentioning later in the guide:

- Standard input, standard output and standard error

Under normal circumstances every UNIX program has three streams opened for it 
when it starts up, one for input, one for output, and one for printing
diagnostic or error messages. The input stream is referred to as
"standard input"; the output stream is referred to as "standard output"; and
the error stream is referred to as "standard error". (From the manual page.)

- Exit status

Whenever a process exits, it hands a number to the operating system in the
range of \[0, 255\]. This number is the exit status. Traditionally, the exit
status 0 signifies success, whilst any nonzero exit status signals failure.

- `argv`

`argv`, or argument vector, is an array of command-line arguments that were
passed to a program. **In the context of fish, it is a variable which contains
the arguments to a function or script**


## Project

To teach you the basics of fish, we will build a CSV parser which converts CSV
to JSON together. I will introduce new concepts and constructs as we move
through, and ask you to implement parts of the script, before showing you my
solution. 

Our final goal will be to write a script that can be called like this:
```fish
csv2json [OPTIONS] [FILE ...]
```

Which will produce output like this:
```json
[
 {
   "col1": "value1",
   "col2": "value2"
 },
 {
   "col1": "value3",
   "col2": "value4"
 }
]
```

So, lets get started!

## Command Line Interface

The first thing we will do is to implement a basic command line interface; that
is, we must parse the options and arguments that a user can give when they run
our script.

At this point, we want to be able to:
- Print help if requested, or if the script was invoked wrong.
- Accept 0 or more file paths as arguments.

Here is some starter code, which you should save as `csv2json`.

```fish
#!/usr/bin/env fish
function csv2json
    echo "Hello, world!"
end

csv2json $argv
```
`#!/usr/bin/env fish` is a shebang, which tells the operating system to execute
the file with fish.

The rest of the script defines the function `csv2json`, and then immediately 
calls it with the arguments passed to the script. (To learn more about 
functions, run `help function`.)

First, make the script executable
```fish
chmod +x csv2json
```

And then you can run it with `./csv2json`.

### Creating and Accessing Variables

Like other programming languages, fish allows us to set and access variables.

- To set a variable, use
```fish
set [NAME] [VALUE]
```
- To access a variable, we can simply prepend `$` to its name, for instance, 
```fish
set WORD cat

# TIP: `echo` will echo standard input to standard output.
echo The word is $WORD               # "The word is cat"

echo The plural of $WORD is {$WORD}s # <- Here, we surround $WORD with {} to
                                     # stop fish from thinking we want the 
                                     # variable "WORDs". 
                                     
echo The plural of $WORD is "$WORD"s # We could also surround $WORD with "",
                                     # but not ''.

echo The plural of $WORD is '$WORD's # This wont work! It prints: 
                                     # "The plural of cat is $WORDs"
```

### The `if` Statement

An `if` statement runs a number of commands if a condition was met. In fish, 
this condition can be any command, and it may have any number of `else if`
clauses and 0--1 `else` clause.

This is the syntax:
```fish
if CONDITION; COMMANDS_TRUE ...;
[else if CONDITION2; COMMANDS_TRUE2 ...;]
[else; COMMANDS_FALSE ...;]
end
```

`if` will execute the command `CONDITION`. If the condition’s exit status is
`0`, the commands `COMMANDS_TRUE` will execute. If the exit status is not `0`
and `else` is given, `COMMANDS_FALSE` will be executed.

### Parsing With `argparse`

The main tool we can use to achieve this is the builtin utility `argparse`. 
We give `argparse` a list of valid options and input, and it will read the
input to parse options for us! For instance, this will return an error unless
`-v` or `--version` was passed:

```fish
# Note that $argv is set when your script is executed, so if we run
# ./csv2json a b c, $argv will be 'a b c'

argparse v/version -- $argv # <-- accept "v" or "version" flags, and read $argv
or return 1                 # <-- if we got bad input, exit with status of 1

if set -q _flag_v           # <-- check if _flag_v was set (will be true if we
                            # were given "-v" or "--version)
    return 0                # <-- exit with status 0, indicating success
end
```

### Exercise 1

Your exercise is to:
- Read the extensive documentation for `argparse` using `help argparse`.
- Accept any number of positional arguments for filenames and store them 
  in the variable `paths`
- Accept a `-h` or `--help` option, and if given, print out this string:
```
csv2json [OPTIONS] [FILE ...]

Options:
    -h, --help
      Print this help
```
- Handle invalid options being passed (e.g., `-notvalid`) by behaving as though
  `-h` was passed, but also print failures and exit with a status of 1.

<details>
<summary>My solution</summary>

```fish
#!/usr/bin/env fish

function _print_help
    echo 'Usage: csv2json [OPTIONS] [FILE ...]

Options:
    -h, --help
        Print this help'
end

function csv2json
    argparse h/help -- $argv
    or set -f failure true # set a failure flag, but otherwise fail silently.

    # check if we've failed
    if set -q failure
        _print_help
        return 1
    end

    # check if the user requested help
    if set -q _flag_h
        _print_help
        return 0
    end
    
    # as argparse goes through $argv, it removes all the options but leaves
    # arguments behind, which are our files.
    set files $argv

    return 0
end

csv2json $argv
```

</details>

## Reading a File

### Redirecting Streams

The destination of a stream can be changed using something called redirection.
Redirecting streams is one of the most useful things we can do in a shell, as it
lets us read, write, and append to files easily.

From the fish documentation, here is what we can do:
- To read standard input from a file, use `<SOURCE_FILE`.
- To read standard input from a file or `/dev/null` if it can’t be read, use
  `<?SOURCE_FILE`.
- To write standard output to a file, use `>DESTINATION`.
- To write standard error to a file, use `2>DESTINATION`.
- To append standard output to a file, use `>>DESTINATION_FILE`.
- To append standard error to a file, use `2>>DESTINATION_FILE`.
- To not overwrite (“clobber”) an existing file, use `>?DESTINATION` or
  `2>?DESTINATION`.

### The `read` Command

From the manual, "`read` reads from standard input and stores the result in
shell variables." This lets us prompt the user for input (or read input 
line-by-line from a file). For example, here we ask the user for their age and
store it in the `age` variable.

```fish
echo "How old are you?"

# Note: `-p ''` suppresses the default prompt that `read` uses.
read -p '' age

echo "Your're $age years old!"
```

### The `while` Command

The `while` command repeatedly executes a body while a condition is met. It
acts almost exactly like `if`, except it runs the body repeatedly, and
`else if` commands are not allowed. For example, if we wanted to poll until
a user responds with "hello":

```fish
read response

# `test` has many different operations it can do, one of which is comparing
# strings. To see all the available options, run `help test`
while test $response != "hello" 
    echo "You didn't say hello!"
    read response 
end 
```

### Exercise 2

One way to read files in a shell is through the utility `cat`. `cat` will read,
concatenate, and print files to standard output. However, this means that we
can only operate on the entire file itself. 

When processing a CSV file, it'd be much better for us to read a file
line-by-line instead. Your exercise is to implement this yourself. (Hint: the
concepts explained to you in this chapter are enough to do this.)

Specifically, 
- Assume that 1 file was given on the command line, which you should have
  stored in `$files` during the previous exercise.
- Open the file, read it line by line, and `echo` it back into standard
  output.


<details>

<summary>Expected Output</summary>

If implemented correctly, the output for `./csv2json beginner-sample.csv`
should be: (if you haven't got the sample file yet, get it [here][3].)
```csv
id,first_name,last_name,email,age,city,salary,joined
1,Liam,Williams,liam.williams1@sample.net,49,Riverside,156973.24,2020-04-18
2,Jane,Williams,jane.williams2@example.com,71,Fairview,50435.05,2024-01-27
3,Emma,Moore,emma.moore3@test.org,76,Fairview,128705.61,2022-01-06
4,Emma,Martinez,emma.martinez4@example.com,43,Burlington,106198.11,2016-01-05
5,Benjamin,Rodriguez,benjamin.rodriguez5@sample.net,62,Manchester,189641.05,2023-04-19
6,Amelia,Williams,amelia.williams6@demo.io,34,Manchester,26628.09,2016-08-19
7,Mia,Jones,mia.jones7@demo.io,21,Ashland,33038.53,2021-01-03
8,Emma,Lopez,emma.lopez8@demo.io,30,Greenville,146867.05,2015-12-15
9,Isabella,Lopez,isabella.lopez9@sample.net,29,Greenville,175981.13,2015-01-27
10,Amelia,Johnson,amelia.johnson10@demo.io,54,Madison,22669.84,2020-03-09
```

</details>

<details>
<summary>My solution</summary>

```fish
# as argparse goes through $argv, it removes all the options but leaves
# arguments behind, which are our files.
set files $argv

# ... continued ...

while read -l line # <- `read` puts each line into `$line`.
    echo $line
end < $files # <- We send the file in `$files` into the while loop.

# ... continued ...

return 0
```

</details>

## Handling Multiple Files

Soooo, remember when I said `csv2json` would accept any number of files to
print? Well, now is the time for that!

### Lists

In fish, every variable is a list, which can contain 0 or more elements. This
is very useful for us handling multiple files, because we can easily iterate
over them! (For other cool things lists can do, run `help language#lists`).

### `for` Loops

fish, like other shells and scripting languages, provides a way to loop over
lists. A `for` command has the following form: 

```fish
for VARNAME in [VALUES ...]
    COMMANDS ...
end
```

Where for every value in `VALUES`, fish will run the body `COMMANDS`, with the
variable `$VARNAME` available within the body.

For instance, we can print out all the filenames we got:

```fish
for filename in $files
    echo $filename
end
```

### Exercise 3

Your exercise is to modify the existing code so that it will handle multiple
files.

<details>

<summary>Expected Output</summary>

If implemented correctly, the output for
`./csv2json beginner-sample.csv beginner-sample.csv` should be:

```csv
id,first_name,last_name,email,age,city,salary,joined
1,Liam,Williams,liam.williams1@sample.net,49,Riverside,156973.24,2020-04-18
2,Jane,Williams,jane.williams2@example.com,71,Fairview,50435.05,2024-01-27
3,Emma,Moore,emma.moore3@test.org,76,Fairview,128705.61,2022-01-06
4,Emma,Martinez,emma.martinez4@example.com,43,Burlington,106198.11,2016-01-05
5,Benjamin,Rodriguez,benjamin.rodriguez5@sample.net,62,Manchester,189641.05,2023-04-19
6,Amelia,Williams,amelia.williams6@demo.io,34,Manchester,26628.09,2016-08-19
7,Mia,Jones,mia.jones7@demo.io,21,Ashland,33038.53,2021-01-03
8,Emma,Lopez,emma.lopez8@demo.io,30,Greenville,146867.05,2015-12-15
9,Isabella,Lopez,isabella.lopez9@sample.net,29,Greenville,175981.13,2015-01-27
10,Amelia,Johnson,amelia.johnson10@demo.io,54,Madison,22669.84,2020-03-09
id,first_name,last_name,email,age,city,salary,joined
1,Liam,Williams,liam.williams1@sample.net,49,Riverside,156973.24,2020-04-18
2,Jane,Williams,jane.williams2@example.com,71,Fairview,50435.05,2024-01-27
3,Emma,Moore,emma.moore3@test.org,76,Fairview,128705.61,2022-01-06
4,Emma,Martinez,emma.martinez4@example.com,43,Burlington,106198.11,2016-01-05
5,Benjamin,Rodriguez,benjamin.rodriguez5@sample.net,62,Manchester,189641.05,2023-04-19
6,Amelia,Williams,amelia.williams6@demo.io,34,Manchester,26628.09,2016-08-19
7,Mia,Jones,mia.jones7@demo.io,21,Ashland,33038.53,2021-01-03
8,Emma,Lopez,emma.lopez8@demo.io,30,Greenville,146867.05,2015-12-15
9,Isabella,Lopez,isabella.lopez9@sample.net,29,Greenville,175981.13,2015-01-27
10,Amelia,Johnson,amelia.johnson10@demo.io,54,Madison,22669.84,2020-03-09
```

</details>

<details>
<summary>My solution</summary>

```fish
# as argparse goes through $argv, it removes all the options but leaves
# arguments behind, which are our files.
set files $argv

# ... continued ...

for file in $files # <- do this for every file we get on the command line
    while read -l line # <- `read` puts each line into `$line`.
        echo $line
    end < $file # <- We send the file in `$file` into the while loop.
end

# ... continued ...

return 0
```

</details>

## Reading files... But better?

Before, we read every file directly into a while loop, and operated directly
there. However, we can store lines in a list to use later, which is more 
convenient for our purposes..

### The `string` Command

In more traditional shells, like *bash* or *zsh*, processing strings is usually
handed to external programs, like `sed` or `awk`. However, fish ships with
the builtin `string` command, which is like a swiss army knife for working with
strings. 

For now though, the most important subcommand for us is
`string split`. `string split` will split each given string on a separator to
give us an array. For instance, this will split the string `"abc"` and store it
into an array named `$ARRAY`:

```fish
set ARRAY (string split '' abc)
```

We can use the `string` command to easily store each file in a list. We can
simply direct the file we want to read into the command:

```fish
# as argparse goes through $argv, it removes all the options but leaves
# arguments behind, which are our files.
set files $argv

# ... continued ...

for file in $files # <- do this for every file we get on the command line
    # Split the file by newlines to get an list of lines, then store it in the
    # variable `$lines`.
    # 
    # Note that we also use `string trim` to get rid of trailing whitespace.
    set lines (string trim (string split '\n' < $file)) 
    
    for line in $lines # We can iterate through our list of lines
        echo $line
    end
end

# ... continued ...

return 0
```

### Exercise 3

There's nothing to implement this chapter, just make sure your code now also
reads the file into an array and iterates over it.

(You should also read the `string split` and `for` documentation with
`help string-split` and `help for`.)

## Parsing CSV

CSV is a format to store tables of data. It's a very simple format, and is
readable as plaintext. It consists of rows and columns, where each column is
separated by a comma, and each row is separated by a newline. For instance,

```csv
column1,column2,column3
1,2,3
4,5,6
```

This makes it relatively easy to parse! For our next exercise, we will
implement the parsing of CSV files.


### Exercise 4

Wait, but there were no new concepts explained? Why's it the exercise already??

It turns out, you already know everything you need to parse a CSV file! Your
task is to:

- Parse the files we read in exercise 2 as CSV files, and print every value on
  its own line

(Extension: The main `csv2json` function is getting quite complex! Maybe we
could simplify it by putting things into their own functions?)

<details>

<summary>Expected Output</summary>

If implemented correctly, the output for `./csv2json beginner-sample.csv`
should start with:

```
id
first_name
last_name
email
age
city
salary
joined
1
Liam
Williams
liam.williams1@sample.net
49
Riverside
156973.24
2020-04-18
2
Jane
Williams
jane.williams2@example.com
71
Fairview
50435.05
2024-01-27
3
Emma
Moore
emma.moore3@test.org
76
Fairview
128705.61
2022-01-06
4
Emma
Martinez
emma.martinez4@example.com
43
Burlington
106198.11
2016-01-05
5
Benjamin
Rodriguez
benjamin.rodriguez5@sample.net
62
Manchester
189641.05
2023-04-19
6
Amelia
Williams
amelia.williams6@demo.io
34
Manchester
26628.09
2016-08-19
7
Mia
Jones
mia.jones7@demo.io
21
Ashland
33038.53
2021-01-03
8
Emma
Lopez
emma.lopez8@demo.io
30
Greenville
146867.05
2015-12-15
9
Isabella
Lopez
isabella.lopez9@sample.net
29
Greenville
175981.13
2015-01-27
10
Amelia
Johnson
amelia.johnson10@demo.io
54
Madison
22669.84
2020-03-09
```

</details>

<details>
<summary>My solution</summary>

```fish
#!/usr/bin/env fish

function _print_help
    echo 'Usage: csv2json [OPTIONS] [FILE ...]

Options:
    -h, --help
        Print this help'
end

# _process_line gets $line as an argument
function _process_line -a line
    # Split the line with seperator ',', and loop over each value in the line
    for value in (string split ',' $line)
        echo $value
    end
end

function _process_file -a file
    # Split the file by newlines to get an list of lines, then store it in the
    # variable `$lines`.
    # 
    # Note that we also use `string trim` to get rid of trailing whitespace.
    set lines (string trim (string split '\n' < $file)) 

    for line in $lines # We can iterate through our list of lines
        _process_line $line
    end
end

function csv2json
    argparse h/help -- $argv
    or set -f failure true # set a failure flag, but otherwise fail silently.

    # check if we've failed
    if set -q failure
        _print_help
        return 1
    end

    # check if the user requested help
    if set -q _flag_h
        _print_help
        return 0
    end

    # as argparse goes through $argv, it removes all the options but leaves
    # arguments behind, which are our files.
    set files $argv

    for file in $files # <- do this for every file we get on the command line
        _process_file $file
    end

    return 0
end

csv2json $argv
```

</details>

## Converting to JSON

### Slices

Before, we touched on lists and how every variable is a list. A slice is a 
portion of a list. In fish, we can make slices with this syntax:

```fish
# Will give us a new slice with elements 1 to 10 of `$list`
$list[1..10]
```

We can also slice from the start or until the end by omitting an index:

```fish
# Will give us a new slice with elements 1 to 12 of `$list`
$list[..12]
```

We cam also access individual elements. Indexing starts at 1 from the
beginning, and -1 from the end:

```fish
# Will give us the last element of $list
$list[-1]
```

### The `count` Command

`count` prints the number of arguments that were passed to it (and the number
of newlines from `stdin`), so we can use it to calculate the length of a list:

```fish
set list 1 2 3

count $list # prints 3
```

### The `seq` Command

`seq` simply prints a sequence of numbers, from `FIRST` to `LAST`:

```fish
seq 1 10
```

will print

```fish
1 2 3 4 5 6 7 8 9 10
```

### The `printf` Command

`printf` prints arguments according to a format string. Note that unlike
`echo`, it doesn't emit a trailing newline, and we can easily format our
values. To print the newline, we need to add the `\n` escape sequence to the
end of the format string.

To use `printf`, we give it a format string and some arguments:

```fish
printf "%s " (seq 1 3)
# prints "1 2 3 "
```

Some useful format specifiers are:

- `%s`: a string
- `%d`: a decimal number

Also, `\n` is a useful escape sequence for new line

See `help printf` for more details!

### Expected Format

Let's revise our script's expected output.

If we are given one file to print, we should convert to JSON and print, like
this:

```json
[
 {
   "col1": "value1",
   "col2": "value2"
 },
 {
   "col1": "value3",
   "col2": "value4"
 }
]
```

If we are given more than one file, we convert each file, but add a
banner with the filename:

```json
<== foo.csv ==>
[
 {
   "col1": "value1",
   "col2": "value2"
 },
 {
   "col1": "value3",
   "col2": "value4"
 }
]

<== bar.csv ==>
[
 {
   "col3": "value1",
   "col4": "value2"
 },
 {
   "col3": "value3",
   "col4": "value4"
 }
]
```

This means:
- We need to check how many files we were given. 
- We need to format our values so that they are valid JSON.
  (As an extension, you could decide to format numerical values as numbers
  instead of strings)
- We need to keep track of our column names, which will be in row 1.


### Exercise 5

This exercise will be a *challenge*. I'm sending you into the deep end, without
much explanation. You best bet will be to use the reading skills you
(hopefully) have gained, and read the documentation for the commands I
mentioned above to implement the program.

If you desperately need help, you can find the sample script below. Also, you
can ask me for help on Slack: @Some1 (U091HKKKJRH)

<details>

<summary>Expected Output</summary>

If implemented correctly, the output for `./csv2json beginner-sample.csv`
should be:

```json
[
  {
    "id": "1",
    "first_name": "Liam",
    "last_name": "Williams",
    "email": "liam.williams1@sample.net",
    "age": "49",
    "city": "Riverside",
    "salary": "156973.24",
    "joined": "2020-04-18"
  },
  {
    "id": "2",
    "first_name": "Jane",
    "last_name": "Williams",
    "email": "jane.williams2@example.com",
    "age": "71",
    "city": "Fairview",
    "salary": "50435.05",
    "joined": "2024-01-27"
  },
  {
    "id": "3",
    "first_name": "Emma",
    "last_name": "Moore",
    "email": "emma.moore3@test.org",
    "age": "76",
    "city": "Fairview",
    "salary": "128705.61",
    "joined": "2022-01-06"
  },
  {
    "id": "4",
    "first_name": "Emma",
    "last_name": "Martinez",
    "email": "emma.martinez4@example.com",
    "age": "43",
    "city": "Burlington",
    "salary": "106198.11",
    "joined": "2016-01-05"
  },
  {
    "id": "5",
    "first_name": "Benjamin",
    "last_name": "Rodriguez",
    "email": "benjamin.rodriguez5@sample.net",
    "age": "62",
    "city": "Manchester",
    "salary": "189641.05",
    "joined": "2023-04-19"
  },
  {
    "id": "6",
    "first_name": "Amelia",
    "last_name": "Williams",
    "email": "amelia.williams6@demo.io",
    "age": "34",
    "city": "Manchester",
    "salary": "26628.09",
    "joined": "2016-08-19"
  },
  {
    "id": "7",
    "first_name": "Mia",
    "last_name": "Jones",
    "email": "mia.jones7@demo.io",
    "age": "21",
    "city": "Ashland",
    "salary": "33038.53",
    "joined": "2021-01-03"
  },
  {
    "id": "8",
    "first_name": "Emma",
    "last_name": "Lopez",
    "email": "emma.lopez8@demo.io",
    "age": "30",
    "city": "Greenville",
    "salary": "146867.05",
    "joined": "2015-12-15"
  },
  {
    "id": "9",
    "first_name": "Isabella",
    "last_name": "Lopez",
    "email": "isabella.lopez9@sample.net",
    "age": "29",
    "city": "Greenville",
    "salary": "175981.13",
    "joined": "2015-01-27"
  },
  {
    "id": "10",
    "first_name": "Amelia",
    "last_name": "Johnson",
    "email": "amelia.johnson10@demo.io",
    "age": "54",
    "city": "Madison",
    "salary": "22669.84",
    "joined": "2020-03-09"
  }
]
```

</details>

<details>

<summary>Example Script</summary>

```fish
#!/usr/bin/env fish

function _print_help
    echo 'Usage: csv2json [OPTIONS] [FILE ...]

Options:
    -h, --help
        Print this help'
end

# _process_line gets $line as an argument
function _process_line -a line col_count
    # Because header was a list, we can't pass it by name here, and we have to
    # get it from $argv, which looks like this: [line, col_count, header...]
    set header $argv[3..]
    # Split the line with seperator ','
    set values (string split ',' $line)

    # Because we want to get the right column for each value, we loop over
    # indexes instead. Note that we need to treat the last value specially -
    # it can't have a trailing comma!
    for index in (seq 1 (math $col_count - 1))
        set col $header[$index]
        set value $values[$index]

        printf '    "%s": "%s",\n' $col $value # <- Note indent level 2
    end

    # Last value shouldn't have a comma
    printf '    "%s": "%s"\n' $header[$col_count] $values[$col_count]
end

function _process_file -a file
    # Split the file by newlines to get an list of lines, then store it in the
    # variable `$lines`.
    set lines (string trim (string split '\n' < $file))

    # Our header is the first line
    set header (string split ',' $lines[1])
    # We use this later with `seq` to index into `header`
    set col_count (count $header)

    # We need to skip line 1, since that's the header. We also need to skip the
    # last line, since it can't have a trailing comma.
    for line in $lines[2..-2]
        echo "  {" # <- Each line is a JSON object, with 1 level indent
        _process_line $line $col_count $header
        echo "  },"
    end

    echo "  {"
    _process_line $lines[-1] $col_count $header # Last line
    echo "  }"
end

function _print_header -a filename
    printf "<== %s ==>\n" $filename
end

function csv2json
    argparse h/help -- $argv
    or set -f failure true # set a failure flag, but otherwise fail silently.

    # check if we've failed
    if set -q failure
        _print_help
        return 1
    end

    # check if the user requested help
    if set -q _flag_h
        _print_help
        return 0
    end

    # as argparse goes through $argv, it removes all the options but leaves
    # arguments behind, which are our files.
    set files $argv

   for file in $files # <- do this for every file we get on the command line
        # If we have more than 1 file, we need to print headers
        if test (count $files) -gt 1
            _print_header $file
        end

        echo "[" # <- Each CSV is a list of JSON objects

        _process_file $file

        echo "]"
    end

    return 0
end

csv2json $argv
```

</details>

## Shipping

So, you've made it to the end! *bashed* has a shipping guide. HOWEVER, the
example project we've just made cannot be shipped by itself!! You will need to
extend it, so that it differs from this tutorial.

Some ideas:
- Maybe parse TSV too (Tab Separated Values)
- Add more options, maybe print the JSON minified?
- Use `curl` to support fetching the CSV from a URL?

Once you've extended the project, find the shipping guide [here][4].

PS: You can use `source csv2json` to be able to use the function within the
shell :3

[1]: <https://fishshell.com/docs/current/>
[2]: <https://fishshell.com/#platform_tabs:~:text=Go%20fish>
[3]: </assets/beginner-sample.csv>
[4]: </guides/ship>
