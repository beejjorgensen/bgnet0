# Trusting User Data

> "You won't know who to trust..."
>
> --Gregor Ivanovich, _Sneakers_

When your server receives data from someone on the Internet, it's not
good enough to trust that they have good intentions.

There are lots of bad actors out there. You have to take steps to
prevent them from sending things that crash your server processes or,
worse, give them access to your server machine itself.

In this chapter we're going to take a high-level look at some issues
that might arise from users trying to send malicious data.

The two big ideas here are:

* _Think like a villain_. What could someone pass your code that would
  crash it or make it behave in an unexpected way?

* Don't trust **anything** from the remote side. Don't trust that it
  will be a reasonable length. Don't trust that it will contain
  reasonable data.

## Buffer Overflow/Overrun

This is something that mostly affects memory-unsafe languages like C or
C++.

The idea is:

1. Your program allocates a fixed size region of memory to use.

2. Your program reads some data into that memory region over a network
   connection.

3. The attacker sends more data than can fit in that memory region. This
   data is carefully constructed to contain a payload.

4. Your program, without thinking about it, writes the data from the
   attacker, filling the memory region and overflowing into whatever is
   in memory after that.

5. Depending on how things are written, the attacker could overwrite the
   return address value on the stack, causing the function to return
   into the attacker's payload and run that. The payload manipulates the
   system to install a virus or change the system to otherwise allow
   remote access.

Modern OSes try to mitigate by making the stack and heap regions of
memory non-executable, and the code region of memory non-writable.

As a developer, you need to write code in C that properly performs
bounds-checking and never writes to memory it doesn't intend to.

## Injection Attacks

These attacks are ones where you build a command using some data the
user has provided to you. And then run that command.

A malicious user can feed you data that causes another command to be
run.

### System Commands

In many languages, there's a feature that allows you to run a command
via the shell.

For example, in Python you can run the `ls` command to get a directory
listing like this:

``` {.py}
import os

os.system("ls")
```

Let's say you write a server that receives some data from a user. The
user will send `1`, `2` or `3` as data, depending on the function they
want to select.

You run some code in your server like this:

``` {.py}
os.system("mycommand " + user_input)
```

So if the user sends `2`, it will run `mycommand 2` as expected and
return the output to the user.

To be safe, the `mycommand` program verifies that the only allowable
inputs are `1`, `2`, or `3` and returns an error if something else is
passed.

Are we safe?

No. Do you see how?

The user could pass the input:

``` {.default}
1; cat /etc/passwd
```

This would cause the following to be executed:

``` {.default}
mycommand 1; cat /etc/passwd
```

The semicolon is the command separator in bash. This causes the
`mycommand` program to execute, followed by a command that shows the
contents of the Unix password file.

To be safe, all the special characters in the input need to be stripped
or escaped so that the shell doesn't interpret them.

### SQL Commands

There's a similar attack called SQL Injection, where a non-carefully
crafted SQL query can allow a malicious user to execute arbitrary SQL
queries.

Let's say you build a SQL query in Python like this, where we get the
variable `username` over the network in some way:

``` {.py}
q = f"SELECT * FROM users WHERE name = '{username}'
```

So if I enter `Alice` for the user, we get the following perfectly valid
query:

``` {.sql}
SELECT * FROM users WHERE name = 'Alice'
```

And then I run it, no problem.

But let's _think like a villain_.

What if we entered this:

``` {.sql}
Alice' or 1=1 --
```

Now we get this:

``` {.sql}
SELECT * FROM users WHERE name = 'Alice' or 1=1 -- '
```

The `--` is a comment delimiter in SQL. Now we've put together a query
that shows all user information, not just Alice's.

Not only that, but a naive implementation might also support the `;`
command separator. If so, an attacker could do something like this:

``` {.sql}
Alice'; SELECT * FROM passwords --
```

Now we get this command:

``` {.sql}
SELECT * FROM users WHERE name = 'Alice'; SELECT * FROM passwords -- '
```

And we get the output from the `passwords` table.

To avoid this trap, use _parameterized query generators_. This will be
something in the SQL library that allows you to safely build a query
with any user input.

Never try to build the SQL string yourself.

### Cross-Site Scripting

This is something that happens with HTML/JS.

Let's say you have a form that accepts a user comment, and then the
server appends that comment to the web page.

So if the user enters:

``` {.default}
This site has significant problems. I feel uncomfortable using it.

Love, FriendlyTroll
```

That gets added to the end of the page.

But if the user enters:

``` {.html}
LOL
<script>alert("Pwnd!")</script>
```

Now everyone will see that alert box whenever they view the comments!

And that's a pretty innocuous example. The JavaScript could be anything
and will be executed on the remote site (meaning it can perform API
calls and fetches as that domain). It could also rewrite the page so
that the login/password input submitted that information to the
attacker's website.

> "It would be bad."
>
> Egon Spengler, _Ghostbusters_

Most HTML-oriented libraries come with a function to sterilize strings
so that the browser will render them and not interpret them (e.g. all
`<` replaced with `&gt;` and so on).

Definitely run any user-generated data through such a function before
displaying it on a browser.

## Reflect

* In general terms, what's the problem with trusting user input?

* Why aren't buffer overflows as much of a problem with languages like
  Python, Go, or Rust as they are with C?

