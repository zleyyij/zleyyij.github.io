layout: post
title: "Writing a usable shell in Rust"
date: 2022-10-22


I set out on project Ash for a a variety of small reasons: 
- I wanted to better learn how a shell interacts with an operating system, how it calls different commands 
- I wanted to get better at writing scaleable code

## The Beginnings
I happened upon [this](https://brennan.io/2015/01/16/write-a-shell-in-c/) article about writing a shell in C, and was fascinated by the way it was written. It was fascinated by the extremely approchable way the article was written, with function calls defined first, and the actual contents of the function filled out later. Then it could be explained what each function does and why it's there, without relying too heavily on language specific semantics. This made it a great stepping stone, even though it's intended for the C programming language. 

I started by writing a very basic framework to obtain user input as a string. As of right now, it's not an entire I/O lock, and so features like tab autocomplete or capturing Ctrl + C to stop the program from being exited are not currently functional. It functions as a loop that:
- Captures user input
- Seperates the user input into a list by spaces(this should probably changed later to account for features like `|, >, ;, &&`, which don't necessarily need a space)
- It then checks the first argument to see if it's a builtin shell command(`cd`,  `help`,  `exit`)

 I found it interesting that `cd` is not an operating system utility, it's a shell utility, and when `cd` is run, it tells the next commands run what directory they were run from. In Rust this is implemented as [current_dir()](https://doc.rust-lang.org/std/process/struct.Command.html) for the Command struct. Initially I actually had a lot of trouble with relative and absolute paths. You can create a functional path by simply appending the relative path to the current absolute path, seperated by `/` (or `\` for Windows). While this is technically functional, it's really not elegant at all. I was ending up with valid file paths like `//./home/../home/./../etc/.`, and felt there must be a better solution. I didn't bother checking to see if Rust had a valid method for it, because I didn't know how to put "cleaning up a file pathpath" into a clean, google-able statement, and I felt I could better understand the process behind parsing it if I implemented it myself. I sat down, absolutely stumped, Obsidian open, writing out various logical rules to clean paths up. I ended up with a few simple precepts that *seemed* mostly functional, but ended up missing edge cases, or having flat out unexplained behavior. The nonfunctional rules are below:
 - Paths that are relative can be appended to the current dir, then
- `..` should strip the non-`..` directory before it from the path 
- `.` can be entirely removed from the absolute path and the endpoint will not be changed

This logic was flawed enough that exasperated and tired, I googled it, hoping that someone had made a crate that cleaned it up, or maybe there was some regex I could use. As it turns out, both Windows and Unix have prebuilt functions that handle cleaning up paths, that are implemented under `std::fs::canonicalize` ([docs](https://doc.rust-lang.org/std/fs/fn.canonicalize.html)). 

- If no builtin commands are found, it passes it over to system exec handler. In C, processes must be started by forking the current process to a new thread, creating an exact copy with the `fork()` system call. You then instruct the new thread to replace itself with another process with the `exec()` call. Rust however, has a method that spawns new programs with `std::process::Command`. The first argument in the string is passed as the process to start, and each of the new arguments is passed to the process as an array of arguments with `.args()`, eg: `ls /bin` would start a new `ls` process, and pass `/bin` as an argument. 

### The Future
I would like to improve on this project and make it good enough that it's daily driveable. Plans for new features include:
- Switching to a complete IO lock, this allows new features like:
	- Tab autocomplete
	- Capturing interrrupts
- A fully featured configuration file that allows changing prompts and behaviors
- Implementing the rest of the functionality that I use regularly, including redirects, pipes, and `;` or `&&`
