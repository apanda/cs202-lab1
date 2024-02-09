---
title: C review, gdb, software skills"
---

This lab will reinforce your prior experience in the C programming
language and give you practice using certain command-line tools. The
overall comfort you'll gain (or reinforce) with the Unix computing
environment will be helpful for future labs. 

Getting started
---------------

You will perform the labs using the CS202 Docker environment, and pull/push code
with Git and GitHub.  It\'s time for you to set up that infrastructure:

Before you go further you need to perform some setup. Go to the 
[setup page](https://cs.nyu.edu/~mwalfish/classes/24sp/), and follow the instructions
there to:

1. Install Docker
2. Build and run the CS202 Docker image

Then

1. Get a GitHub account if you don't have one already
2. Create a private repository.

## Workflow.
We have configured Docker so that the directory from which you run
`./cs202-run-docker` on your host machine (usually this is `cs202`) shows up in Docker as
`cs202-labs`. This means that you can use your preferred IDE or editor
on your host machine to edit code while
compiling, running, and testing inside Docker. Depending on your setup,
it may be helpful for you to have at least two windows up (one for
editing and one with Docker).

Section 1: C review
-------------------

All of the projects in this class will be done in C or C++, for two main
reasons. First, some of the things that we want to implement require
direct manipulation of hardware state and memory, which are operations
that are naturally expressed in C. Second, C and C++ are widely used
languages, so learning them is a useful thing to do in its own right.
You have some experience in C from CS201, and you will need to build on
that here. 

If you are interested in why C looks like it does, we encourage you to
look at Ritchie\'s
[history](https://www.bell-labs.com/usr/dmr/www/chist.html). Here,
perhaps, is the key quotation: \"Despite some aspects mysterious to the
beginner and occasionally even to the adept, C remains a simple and
small language, translatable with simple and small compilers. Its types
and operations are well-grounded in those provided by real machines, and
for people used to how computers work, learning the idioms for
generating time- and space-efficient programs is not difficult. At the
same time the language is sufficiently abstracted from machine details
that program portability can be achieved.\"

You will do short exercises in C, as a warmup, and to help review some
of what you learned in CS201 about C. Enter the Docker environment
(`./cs202-run-docker`, per the [setup page](../setup.html)); you'll know
you're in the Docker environment if you see a prompt like this:

    cs202-user@af8b1be95427:~/cs202-labs$ 

then type `cd lab1/mini` after the prompt:

    cs202-user@af8b1be95427:~/cs202-labs$ cd lab1/mini
    cs202-user@af8b1be95427:~/cs202-labs/lab1/mini$

::: {.highlight}
[Shell prompt.]{.header} From here forward, we will not include the `cs202-user@...` part of the
shell prompt in this description, but it should be present on your
machine. If it is not, then you are not
in the Docker environment.
:::

(Depending on your CS201 section, you may have seen very similar, or the
same, exercises; that's fine. If you're not absolutely fluent in C
programming, then doing them again will be useful.)

::: {.required}
[Exercise 1.]{.header} Implement the functions in `part1.c`
:::

For Exercises 1 through 3, you can test with:

    $ make
    $ build/part1

You should have seen `make` in CSO. Recall that it is an automatic way
to build software. In this case, it invokes the compiler and links your
compiled code to a small test harness that we provide. 

    $ ./build/part1
    part1: set_to_fifteen OK
    part1: array_sum OK

If your part1.c is correctly implemented, you will see OK, as above.

You will need to remove the `assert(0);` line. Remove this as you
implement functions in future exercises; it is a reminder that you have
yet to implement a particular function.

As you write your code and improve it (fixing bugs, adding
functionality), you should get in the habit of syncing your
changes to the master copy of your labs repository on GitHub, for
example:

    $ git commit -am 'my solution for lab1 exercise1'
    Created commit 60d2135: my solution for lab1 exercise1
     1 files changed, 1 insertions(+), 0 deletions(-)
    $ git push origin

The `commit` keeps the history of changes to your code, and so allows
you to revert to an older version if you find that a change causes a
regression. The `push` serves to back up your code on GitHub's servers,
so you won't lose work if your local working copy is corrupted or lost. 

**Note**: If `git push origin` does not work from within Docker and you
are on a Mac, then make sure your `ssh-agent` is running. See 
[here](../setup.html#sshkeys). If you are not on a Mac, you can either
push from outside Docker using your host's `git` or you can use GitHub's
[Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).

::: {.required}
[Exercise 2.]{.header} Implement the functions `set_point` and `point_dist` in `part2.c`.
:::

As above, you compile and test with:

    $ make

Now you should see

    $ ./build/part2
    part2: set_point OK
    part2: point_dist OK

Note that you can keep track of your changes by using the `git diff`
command.  Running `git diff` will display the changes to your code since
your last commit, and `git diff origin/main` will display the changes
relative to the initial code supplied for this lab. Here,
`origin/main` is the name of the git branch with the initial code you
downloaded from our server for this assignment.

::: {.required}
[Exercise 3.]{.header} Implement the linked_list utility functions in `part3.c`.
:::

Test it:

    $ make
    $ ./build/part3
    part3: list_insert OK
    part3: list_end OK
    part3: list_size OK
    part3: list_find OK
    part3: list_remove OK

::: {.highlight}
[Debugging note]{.header}: if you are having trouble with this piece, you may wish to
skip to the section covering gdb, below, and come back here. You would
do:

    $ gdb build/part3 core
    (gdb) bt

(Make sure you've issued the `ulimit` command below.)
:::

::: {.highlight}
[Another debugging note]{.header}: the `assert` lines we've been seeing are a
simple application of a powerful tool. You may wish to use asserts
yourself, to help debug your linked list functions. 

In C, an `assert` is a *preprocessor macro* which effectively enforces a
contract in the program. (You can read more about macros in C
[here](http://gcc.gnu.org/onlinedocs/cpp/Macros.html).) The contract
that `assert` enforces is simple: when program execution reaches
`assert(<condition>)`, if `condition` is true, execution continues;
otherwise, the program aborts with an error message.

Assertions, when used properly, are powerful because they allow the
programmer who uses them to guarantee that certain assumptions about the
code hold. For example, you will see assertions like:
    
    assert(head != NULL);

This assertion enforces the contract that the parameter head cannot be
`NULL`. If these assertions were not present and we tried to dereference
`head`, for example with `*head`, we would encounter a type of error
called a *segmentation violation* (or *segmentation fault*). This is
because dereferencing the `NULL` address is invalid; `NULL` points to
\"nothing\". Later in the lab, we will get some experience with
segmentation faults.

But by using assertions, we guarantee that `list_end` (for example) will
never try to dereference a `head` variable at `NULL`, saving us the
headache of having to debug a segmentation fault if some code tried to
pass us a `NULL` value.  Instead, we will get a handy error message
describing exactly what contract of the function was invalidated.

Use your own asserts to make debugging easier!
:::

### Advice on debugging common problems encountered in doing this lab

* **Remember to recompile changed code** Whenever you've changed a file,
always type `make` to re-compile before executing again.

* **Write your own simple test code.** Don't rely solely on the lab's
testing infrastructure (i.e. ./grade-lab) to test the correctness of
your code. We don't distribute the source of the harness code; this makes it hard to debug. So you should write your own tester.
Let's say you want to test the `array_sum` function in the file
`part1.c`. To write your own test code, create a file (e.g. called
`test-part1.c`) with a `main` function that invokes `array_sum` in various ways to test its correctness.  
    
  An example `test-part1.c` might look something like [this](test_part1.c).
  
  Compile your test code by typing:

        $ gcc -pedantic -Wall -std=c11 -g -o test_part1 test_part1.c part1.c

* **Use `gdb`** As we noted above, and as will be covered below.

### C programs from scratch

In the rest of Section 1 of the lab, you will get practice writing and
compiling C programs from scratch. Knowing how to create a standalone C
program is useful in its own right, and it will be helpful context for
lab2.

We will quickly walk through how to write and compile a program in
C, and then you will write and compile several programs of your own.

Using a text editor, create a file called `fun.c`. In this file, type:

    #include <stdio.h>

    int main(int argc, char** argv)
    {
        char* first_arg; 
        char* second_arg; 

        /* this checks the number of arguments passed in */
        if (argc != 3) {
            printf("usage: %s <arg1> <arg2>\n", argv[0]);
            return 0;
        }

        first_arg = argv[1];
        second_arg = argv[2];

        printf("My program was given two arguments: %s %s\n",
               first_arg, second_arg);

        return 0;

    }

This is a complete C program. The `#include` at the top tells the compiler to use
the header files of \"standard I/O\" (the standard input/ouput functions
of the C library; these functions include `printf`). Also, `argc`
contains the number of arguments passed to the program (including the
program name itself) while `argv[0]` contains the name of the program
that was invoked.

You can compile this program using:

    $ gcc -pedantic -Wall -std=c11 -g -o fun fun.c

You can now run this program using:

    $ ./fun

You should see:

    usage: ./fun <arg1> <arg2>

You can also do:

    $ ./fun abc def
    My program was given two arguments: abc def

Note the pattern here:

-   you create a C file and compile it with

        $ gcc [flags] -o <output-file-name> <input-file-name>

-   You run your program using whatever was after the `-o` flag.

-   Inside the program, you gain access to the arguments using the
    `argv` array. (As we'll see in this semester, the OS syscall
    `exec()` takes an argv; the OS takes that array and passes it to the new program\'s `main()`
    function.)


::: {.required}
[Exercise 4.]{.header}
Write, from scratch, a C program that takes two arguments and prints the
first three characters of each string. If either of the two arguments
has fewer than three characters, print an error message, and end
gracefully (as opposed to core dumping).

For example:

    $ ./first3 a b
    ./first3: error: one or more arguments have fewer than 3 characters

    $ ./first3 abcd wxy
    abcwxy

Instructions:

* You may use the function `strlen()`. If you do, include `#include <string.h>` at the
beginning of the program.
* Put all your files in `~/cs202/lab1/fromscratch/first3`.
* Your executable file must be named `first3`.
* You must supply a Makefile. When we type `make`, it needs to create a binary executable named `first3`.
* Be diligent about creating test cases for yourself. You will lose points for not handling corner cases. You may wish to create a script that runs and tests your code on various cases. (You do not have to hand in your testing code, however; we will run our own testing scripts against your code.)
* Pay attention to coding style.
* Remember to add relevant source files and the Makefile to your git
repo. Type `git status`, and if any of the files look relevant, add and
commit them:
    
        $ git status
        $ git add *.c *.h Makefile
        $ git commit -am "first3"
        $ git push origin

:::


::: {.required}
[Exercise 5.]{.header}
Write, from scratch, a C program that takes a single argument and prints
the number of ascii 'a' characters in the string. **Don't use the
function `strlen()`**; programs using `strlen()` will get 0 points. The
purpose of this exercise is in part to give you familiarity with
so-called "null-terminated strings" in C. A null-terminated string is a
pointer to an array of characters (`char *`), in which the end of the
string is indicated by the character '\0', which has value 0, or `NULL`.

For example:

    $ ./countas abbba
    2

    $ ./countas aaaaa
    5

    $ ./countas bcdefghwxyz
    0

    $ ./countas
    usage: countas <arg>

    $ ./countas aaaaa bbbbbb
    usage: countas <arg>

Instructions:

* Again, don't use `strlen()`.
* Put all your files in `~/cs202/lab1/fromscratch/countas`.
* Your executable file must be named `countas`.
* If `countas` receives any number of arguments other than 1, it should
print a help/usage message (see above).
* You must supply a Makefile. When we type `make`, it needs to create a
binary executable named `countas`.
* Be diligent about creating test cases for yourself. You will lose points for not handling corner cases. You may wish to create a script that runs and tests your code on various cases. (You do not have to hand in your testing code, however; we will run our own testing scripts against your code.)
* Pay attention to coding style.
* As above, remember to add relevant files (source file, Makefile) to
your git repo: `git status ; git add [...] ; git commit -am "countas" ;
git push origin`.
:::


Section 2: Debugging
--------------------

This part of the lab will give you practice debugging. 

Put your answers for this section and in the next in the supplied
`answers.txt` file.

### Navigating to syntax errors

Go to the `dbg` directory in `lab1`:

    $ cd ~/cs202-labs/lab1/dbg

Now type:

    $ make

The code has a syntax error; thus, it cannot be compiled.

::: {.required}
[Exercise 6.]{.header} Fix the syntax error.

Use the compiler\'s error message to determine what\'s wrong. Note
that in the `vi` text editor (discussed below) you can navigate to a given line
of code `:<number>`. You can also *launch* `vi` with its cursor in
place:

    $ vi +<number> <filename>

to begin directly on a given line number. For example, `vi +5 foo.c`
begins with the cursor at line 5.
:::

After you fix the syntax error, the code will compile. Use `make` to see
this:

    $ make

Now, try testing the code itself, using the small `test_linked_list`
utility:

    $ ./test_linked_list
    Segmentation fault (core dumped)

Aha! Our code compiled, but it was not correct (core dumps are bad).
Specifically, the segmentation fault means that our program issued an
illegal memory reference, and the operating system ended our process.
Making matters worse, we have no idea what the problem in the code is.
In the following section, you will learn how to use `gdb` to debug this
kind of problem.

**Run gdb:** Use the GNU debugger, or `gdb` to run the program:

    $ gdb test_linked_list
    (gdb)

**Set breakpoints:** One thing that you might want to do is to set a
*breakpoint* before the program begins executing. Breakpoints are a way
of telling `gdb` that you want it to execute your program and then stop,
or *break*, at a place that you define. Use the following command to set
a breakpoint at the `main` function:

    (gdb) b main
    Breakpoint 1 at 0x400963: file test_linked_list.c, line 43.

Then use gdb\'s command `run` to actually start the program (this is the
general pattern in gdb: one invokes the debugger, perhaps sets a
breakpoint, and then starts the program with `run`):

    (gdb) run

The program will be stopped when it reaches the breakpoint (advanced
topic: how does gdb conspire with the hardware to make this work??). At
this point, you will be presented with gdb\'s command prompt again. To
see the "call stack" (or <i>stack trace</i>), which is the list of
functions that have called this one -- literally, the stack frames on
top of the current one -- you issue `backtrace` or `bt` for short:

    (gdb) bt

Experienced developers will often ask for a stack trace as step 0 or 1
of understanding a code problem. Get in the habit of asking `gdb` to
give you a backtrace.

To make the program continue running after a breakpoint, use `continue`,
or `c` for short:

    (gdb) c

**Step through the code:** Of course, if you just `c` every time you hit a
breakpoint, then you will lose control of the program. You often want
the command `next`, or `n`:

    (gdb) n

This \"executes\" the next line of code, for example executing an entire
function. (The command `step` executes a single line of C code. There is
little difference between `step` and `next` unless you are about to
enter a function. `step` steps into the function; `next` \"steps over\"
the function.)

[**Inspect the values of variables:**]{#checkVariable} In gdb\'s command
prompt, the program is stalled. You can query the program\'s current
global and local variables with the `print` command, or `p` for short.

::: {.required}
Run `gdb` on `test_linked_list`. Set a breakpoint at the function
`list_delete`.
:::

At this breakpoint, determine the value of the integer `id`:

    (gdb) print id
    $1 = 1

This means that variable `id` holds the integer 1.

Aside: you can check local variables\' names using:

    (gdb) info local

**Core dump:** If a program terminated abnormally (for example,
`test_linked_list`), the state of the program will be recorded by the OS
and (if core dumps are enabled) saved in a so-called core dump. `gdb`
can use a core dump to inspect a crash situation.

To debug using core dumps, you must first enable core dumps, and then
point gdb at the relevant file. We\'ll do this in several steps:

    # enable core dumps
    $ ulimit -c unlimited 
    $ ./test_linked_list
    $ ls -l core
    $ gdb ./test_linked_list core

The idea here is that the core file gives `gdb` enough information to
recover the memory and CPU state of the program at the moment of the
crash. This will allow you to determine which instructions experienced
the error.

::: {.required}
[Exercise 7.]{.header} Fix the bug in the code. Recompile and rerun to
make sure it is fixed. State the line of code and the fix in
`answers.txt`.
:::

