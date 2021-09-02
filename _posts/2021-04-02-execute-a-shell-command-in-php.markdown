---
layout: post
title:  "Executing shell commands from a PHP script"
date:   2021-04-02 20:00:00 +0100
permalink: /execute-a-shell-command-in-php/
redirect_from: /2021/04/02/execute-a-shell-command-in-php/
description: All the details of calling an external process from PHP.
excerpt: "If you need to call an external program from your PHP script, for example to create a PDF file or convert images, there are several ways to do that. I strongly recommend using the Symfony Process Component. It wraps around native PHP functions like proc_open() and it provides extra level of security. It is also very convenient because of an object-oriented interface."
tags:
  - php
---

If you need to call an external program from your PHP script, for example to create a PDF file or convert images, there are several ways to do that.

I strongly recommend using the [Symfony Process Component](https://symfony.com/doc/current/components/process.html). It wraps around native PHP functions like `proc_open()` and it provides **extra level of security**. It is also very convenient because of an **object-oriented interface**. Take a look at this example:

```php
use Symfony\Component\Process\Exception\ProcessFailedException;
use Symfony\Component\Process\Process;

// call the wkhtmltopdf program with two arguments; specify input and output
$process = new Process(['wkhtmltopdf', '-', '-']);

// send something to the command's input
$process->setInput($html);

try {
    // wait for process execution
    $process->mustRun();

    // get output from the command
    $pdf = $process->getOutput();
} catch (ProcessFailedException $exception) {
    echo $exception->getMessage();
}
```

Most guides around the web will simply tell you about functions like `exec()`, but that's not how you should do it. You can either end up with security issues in your application, or just lack features.

This has been a short introduction. If you have more time to read, let me show you **all the details** of calling an external process from a PHP script.

## Input, output and exit codes

A program running under an operating system is a *process*. This is the word we are going to use. We can run processes for example by entering *commands* inside a terminal.

A command usually consists of the program file name followed optionally by a set of arguments. Every program expects different arguments, and if we don't know them, we simply ask the program for help:

```
ls --help
```

![Processes accept several inputs and can produce multiple outputs](/assets/unix-process-diagram.svg)

**A process has several connectors to the surrounding environment.** It's using them to transfer data over *streams*.

A stream is just a sequence of bytes. There are three default streams in a terminal:
* standard input (STDIN), connected to the keyboard
* standard output (STDOUT), connected to the screen
* standard error output (STDERR), either displayed on the screen or written to a log file

Using streams gives us great flexibility. Instead of operating on a real console or real files in PHP, we can **send a string variable to a process** and then **read the output into another variable**. We don't need to remember to delete a file. This will help a lot for example with PDF conversions.

When calling processes that operate on files by default, sometimes a hyphen (`-`) is used in place of a file name argument to indicate that the process should read from STDIN instead, or write to STDOUT instead of a real file.

Additional input to a process consists of *environment variables*. These can be some user-specific data stored in their home directories, or variables provided at the process startup. They make the arguments list shorter because we don't have to specify common settings on every command call. Perhaps the most known environment variable is `PATH` which stores a list of directories where commands are searched.

A process can also return a special code - *exit code* - which indicates success or failure. The convention is to use 0 in case of success and any other code from 1 to 255 to show a different situation.

You can connect several processes in a chain using the *pipe* operator. This means that the output of the first process is tied to the input of the second process, and so on. Such mechanism is commonly used in terminals, for example to paginate long output:

```
ls -al | less
```

In the example above, the output of the `ls` command was sent to the `less` command. If the `ls` failed, the chain would break and the second command would not be called.

By default, we have to wait until each process terminates. This means our PHP script will also be paused when executing an external command. If you add an ampersand (`&`) at the end of the command, the command will run independently. It won't block your script and will last even after the script stops. You might need this especially when launching lengthy processes like generating a 100-page report.

It's good to have some control over such a background action. Fortunately, every process receives an identifier after being opened. The process identifier (PID) can be used later for example to check if the process is still running or to shut it down.

Now that we have the general rules covered, we can switch to the PHP world.

## Basic execution from a PHP script

There are four (!) PHP functions which purpose is to [run an external command and return output](https://www.php.net/manual/en/ref.exec.php):

* `exec()` accepts command as input and returns the last line from the result of the command. Optionally, it can fill a provided array with every line of the output and also assign the return code to the variable. On failure, the function returns `false`.
* `passthru()` executes a command and passes the raw output directly to the browser. The PHP documentation recommends it in case if *binary* output has to be sent without interference.
* `shell_exec()` executes a command and returns the complete output as a string. It does not provide the exit code. The function return value is confusing because it can be `null` both if an error occured or if the command produced no output.
* `system()` acts like `passthru()`, but it also returns the last line of the output. This function works well only with *text* output.

To confuse you even more, PHP has a [backtick operator](https://www.php.net/manual/en/language.operators.execution.php) which works just like `shell_exec()`:

```php
$output = `ls -al`;
```

**I don't use *any* of these functions because none of them provides full control over streams.**

## Escaping arguments

Sometimes the full command is made from several parts, for example a file name coming from a user. **We have to filter such input data properly** to make sure it does not contain any unescaped special characters like spaces, quotes, backticks, slashes, and so on. They could either break the command or cause security issues.

An attacker could inject any other command and perhaps access protected data:

```php
// this is terribly unsafe
system('touch ' . $_POST['filename']);

/*
 * $_POST['filename'] could be equal to something like:
 *   a || cat /etc/passwd
 * so the full command would become:
 *   touch a || cat /etc/passwd
 * which would reveal the contents of a protected file.
 */
```

Every command argument should be filtered by the `escapeshellarg()` function. This will ensure that your input data will be properly treated as a single safe argument:

```php
// this is safer, but still ugly
system('touch ' . escapeshellarg($_POST['filename']));
```

Of course the `filename` parameter should be further filtered to make sure an attacker cannot access any other directories outside the current one:

```php
// this is safe assuming we are in a special directory for uploaded content
system('touch ' . escapeshellarg(basename($_POST['filename'])));
```

## Opening and controlling a process

The `proc_open()` function provides the most possibilities to control a process execution. Its usage requires a lot more code than the one-liners mentioned earlier, but it pays off.

Here's an example of calling a `wkhtmltopdf` program which converts an input HTML document to a PDF. We'll supply the HTML contents to STDIN and read the output from STDOUT:

```php
$html = '<html><body>Test</body></html>';

$descriptors = [
    0 => ['pipe', 'r'],  // we will write to stdin
    1 => ['pipe', 'w'],  // we will read from stdout
    2 => ['pipe', 'w'],  // we will also read from stderr
];

// this array will contain three pointers to all three pipes
$pipes = [];

// we're starting the process now
$process = proc_open('wkhtmltopdf - -', $descriptors, $pipes);
if (is_resource($process)) {
    // the process has been opened, we can send input data
    fwrite($pipes[0], $html);

    // you have to close the stream after use
    fclose($pipes[0]);

    // now we're reading binary output
    // PHP will wait until the stream is complete
    $pdf = stream_get_contents($pipes[1]);
    fclose($pipes[1]);

    $errors = stream_get_contents($pipes[2]);
    fclose($pipes[2]);

    // all pipes must be closed now to avoid a deadlock
    $exitCode = proc_close($process);
}
```

Here you can see how we invoked the `wkhtmltopdf` process and told it to operate on standard input/output streams instead of real files (notice the two hyphens). Our script was halted until the external program returned full output and terminated. If everything went fine, `$exitCode` should equal 0.

There are three optional arguments to `proc_open()`, in consecutive order:
1. `$cwd` - current working directory; if not specified, the process will operate in the same directory as the current PHP process.
2. `$env` - an array of environment variables. If not provided, the child process will inherit all the environment of the PHP process.
3. `$other_options` - at the moment this can only contain Windows-specific console options. Nothing to see here.

If you only need a unidirectional pipe, you can use the `popen()` function (isn't PHP function naming confusing?). A one-way communication is easier to handle:

```php
// we will send HTML contents to STDIN and save PDF output to a file
$process = popen('wkhtmltopdf - output.pdf', 'w');
fwrite($process, $html);
pclose($process);
```

## The most convenient solution: The Process Component

All the code demonstrated above looks like low-level C programming. It's not really comfortable in the modern age of object-oriented programming and abstractions. Today you shouldn't worry about resources, pointers and streams.

To include [the Process component](https://symfony.com/doc/current/components/process.html) in your project, just use Composer in the command line:

```
composer require symfony/process
```

To remind you, the basic usage consists of just creating an instance of the `Process` class, providing input arguments and getting output:

```php
use Symfony\Component\Process\Exception\ProcessFailedException;
use Symfony\Component\Process\Process;

$process = new Process(['wkhtmltopdf', '-', '-']);
$process->setInput($html);

try {
    // wait for process execution
    $process->mustRun();

    $pdf = $process->getOutput();
} catch (ProcessFailedException $exception) {
    echo $exception->getMessage();
}
```

Notice that we pass input arguments as an *array*. We no longer create a long command invocation by hand; the Process component assembles the call, taking care of **proper argument escaping**. Instead of exit codes, we use exceptions just like it should be done in a modern object-oriented environment.

> An alternative to `$process->mustRun()` is just using `$process->run()` and then checking the result of `$process->isSuccessful()`.

The process will inherit all environment variables from the PHP process running the script. You can provide additional variables (or override them) at runtime:

```php
$process = new Process(['ls', '-al']);
$process->run(null, ['SOME_VARIABLE' => 'value']);
```

Refer to the documentation of the specific process you are calling to know all the input rules.

### Asynchronous and background processes

As we know, running a child process blocks the parent process by default. This is the easiest and safest behavior.

Some processes take considerable amount of time and it would be good to let the user know what is happening. You can provide an anonymous function which is going to receive every piece of output coming from a child process:

```php
// we need to receive the current unbuffered output of a process
ini_set('output_buffering', 0);

$process = new Process(['wkhtmltopdf', '-', '-']);
$process->setInput($veryLongHtml);
$process->run(function ($type, $buffer) {
    if (Process::ERR === $type) {
        $errorOutput .= $buffer;
    } else {
        $mainOutput .= $buffer;
    }
});
```

If our main script logic does not strictly depend on the complete child process execution, we can run things in parallel. While a child process does its job, we can do other things in the meantime and occassionally check on that process:

```php
$process = new Process['wkhtmltopdf', '-', '-']);
$process->setInput($veryLongHtml);
$process->start();
$pid = $process->getPid();

// do some other things here...

$process->wait();
echo $process->getOutput();
```

### Timeouts

To prevent a process from hanging forever, two types of timeout mechanisms were introduced:
* **a general timeout**, measured from the process start,
* **an idle timeout**, measured from the last output received from a process.

By default, a process has a general timeout of 60 seconds. You can change it with the `setTimeout()` method of the `Process` class. The other clock can be adjusted with `setIdleTimeout()`.

When running a lengthy command asynchronously, you must use `checkTimeout()` to see if the timeout is reached.

**Remember there are plenty of other timeouts** in the surrounding environment. PHP has also its own maximum script execution times - different for a web server and CLI (Command-line Interface).

> If your child process stops unexpectedly, this might mean that you're exceeding some timeout, either set by yourself, the PHP environment, an operating system, a database or anything else.

## Reporting progress of time-consuming tasks

When a user requests a report, a package or any other piece of data which preparation takes more time, you should not make people stare at the loading icon forever. They will either think that their internet connection is broken, or the server went down. **They might panically hit the "Refresh" button**, and thus make even more trouble by causing multiple requests to start. They might even hang your server.

The basic solution is to add a message which says "This might take a few minutes." However, a user might hit a browser timeout while watching that loading icon.

**It's better to send the results in an e-mail**, or make some *push notification* which says "ok, you can download your file here." The user knows that they don't have to wait until the process is done, they can just leave the computer for a while and come back later.

If the process produces a series of files, let's say a hundred PDFs, it is fairly easy to **track progress**. The worker process simply has to report the number of finished items, for example by writing it to a file. Your frontend will simply read that file and render a nice progress bar. You can also track time of every item preparation to make fancy estimations about the remaining time.

Why not go even further and have anonymous statistics of all user requests? You can then tell users: "this usually takes 3 to 5 minutes."

## Queueing tasks

**Your server has limited resources.** What happens if a thousand users suddenly request a freshly generated PDF document?

In bars, restaurants and shops, people wait in line to be served. There might be for example three people on the counter, and every one of them can serve one customer at a time.

Same rules apply to computing. A processor has a finite number of cores. Running more processes than the number of cores requires your processor to **switch between tasks**. Your PHP installation when using [FPM](https://www.php.net/manual/en/install.fpm.configuration.php) also has some *pool* settings which defines **how many requests can be handled simultaneously**.

When you know how much tasks your system can take at once, you should enforce limits and use some queue system. It can be [RabbitMQ](https://www.rabbitmq.com/) or [Kafka](https://kafka.apache.org/), for example. These are battle-tested tools which are going to control your queues. I'm not covering them in this book.

Having a queue means that your user will have even more waiting time. You should take this into account when informing users about estimated delivery time. However, this is a basic way to ensure that your customers will be served at all, eventually. If you let everyone in at the same time, chances are no one will be served successfully and you'll get bad reviews.

## Wrapping up

There's a lot of low-level PHP functions to call external commands, but today the best option is to use a wrapper library like Symfony Process.

When running other processes from your PHP scripts, you need to know how a process works, how to read and write data, and how much tasks your server can handle at once. Try queueing time-consuming tasks.
