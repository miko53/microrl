microrl - micro read line library for small and embedded devices with basic VT100 support.

# 1. DESCRIPTION

Microrl library is designed to help implement command line interface in small and embedded devices.

Main goal is to write compact, small memory consuming but powerful interfaces, with support navigation through command line with cursor, HOME, END keys, hot key like Ctrl+U and other, history and completion feature.


# 2. FEATURE

## config.h file
- Turn on/off feature for add functional/decrease memory via config files.

## hot keys support

- backspace, cursor arrow, HOME, END keys
- Ctrl+U (cut line from cursor to begin)
- Ctrl+K (cut line from cursor to end)
- Ctrl+A (like HOME)
- Ctrl+E (like END)
- Ctrl+H (like backspace)
- Ctrl+D (delete one character forward)
- Ctrl+B (like cursor arrow left)
- Ctrl+F (like cursor arrow right)
- Ctrl+P (like cursor arrow up)
- Ctrl+N (like cursor arrow down)
- Ctrl+R (retype prompt and partial command)
- Ctrl+C (call 'sigint' callback, only for embedded system)

## history

- Static ring buffer history for memory saving. Number of commands saved to history depends from commands length and buffer size (defined in config)
- can display list of historic commands
- can execute again previous historic commands (e.g >!<historic number>)

## completion
 - via completion callback

## quoting (optional)
 - Use single or double quotes around a command argument that needs to include space characters.

## echo
 - use "microrl_set_echo" function to turn on or turn off echo.
 - could be used to print * instead of real characters.

# 3. SRC STRUCTURE

~~~
src/               - library source
  microrl.c        - microrl routines
  microrl.h        - lib interface and data type
  config.h         - customisation config-file
examples/          - library usage examples
  avr_misc/        - avr specific routines for avr example
  pic_lf4520_misc/ - PIC specific routines for PIC18F4520 chip
  unix_misc/       - unix specific routines for desktop example
  esp8266_example/ - example for esp8266 (platformio) - echo feature
  example.c        - common part of example, for build  demonstrating example for various platform
  example_misc.h   - interface to platform specific routines for example build (avr, unix)
  Makefile         - unix example build (gcc)
  Makefile.avr     - avr example build (avr-gcc)
~~~

# 4. INSTALL

Requirements:

C compiler with support for C99 standard (GNU GCC, Keil, IAR) with standard C library (libc, uClibc or other compatible). Also you have to implement several routines in your own code for library to work.

**NOTE**: need add -std=gnu99 arg for gcc

For embed lib to you project, you need to do few simple steps:

1. Include `microrl.h` file to you project.
2. Create `microrl_t` object, and call `microrl_init()` func, with print callback pointer. Print callback pointer is pointer to function that call by library if it's need to put text to terminal. Text string always is null terminated.
For example on linux PC print callback may be:
```c
	// print callback for microrl library
	void print (char * str)
	{
		fprintf (stdout, "%s", str);
	}
```
3. Call `microrl_set_execute_callback` with pointer to you routine, what will be called if user press enter in terminal. Execute callback give a `argc`, `argv` parameters, like `main()` func in application. All token in `argv` is null terminated. So you can simply walk through argv and handle commands.

4. If you want completion support if user press TAB key, call `microrl_set_complete_callback()` and set you callback. It also give `argc` and `argv` arguments, so iterate through it and return set of complete variants.

5. Look at `microrl_config.h` file, for tune library for you requiring.

6. Now you just call `microrl_insert_char()` on each char received from input stream (usart, network, etc).

Example of code:

```c
	//*****************************************************************************
	int main (int argc, char ** argv)
	{
		// create microrl object and pointer on it
		microrl_t rl;
		microrl_t * prl = &rl;
		// call init with ptr to microrl instance and print callback
		microrl_init (prl, print);
		// set callback for execute
		microrl_set_execute_callback (prl, execute);
		// set callback for completion (optionally)
		microrl_set_complete_callback (prl, complet);
		// set callback for ctrl+c handling (optionally)
		microrl_set_sigint_callback (prl, sigint);

		while (1) {
			// put received char from stdin to microrl lib
			char ch = get_char ();
			microrl_insert_char (prl, ch);
		}
		return 0;
	}
```
See examples library usage.

Author: Eugene Samoylov aka Helius (ghelius@gmail.com)
01.09.2011
