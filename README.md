# gnuplot_i
gnuplot interface in ANSI C

Forked from Nicolas Devillard's gnuplot_i (http://ndevilla.free.fr/gnuplot/)

gnuplot is a freely available, command-driven graphical display tool for Unix. It compiles and works quite well on a number of Unix flavours as well as other operating systems.

The following module enables sending display requests to a gnuplot session through simple ANSI C calls.

Notice that gnuplot_i talks to a gnuplot process by means of POSIX pipes. This implies that the underlying operating system has the notion of processes and pipes, and advertizes them in a POSIX fashion. Since Windows does not respect this standard, this module will not compile on it, unless you have a compiler that offers a popen call on that platform or simulates it.

# Reference Manual
## Introduction
gnuplot_i (formerly known as gnuplot_pipes) is yet another attempt to provide a programmer-friendly set of routines to use gnuplot as a data displayer from a C program. Looking into the code, you will see that it is actually talking to gnuplot through a Unix pipe (ports to Windows involve simulating a Unix pipe as well), instead of using a kind of API to gnuplot's plotting features.

Why is that so? Simply because gnuplot has not been programmed with a C API in mind, but to be an interactive plotting program. There have been several attempts at trying to extract a set of communication functions to the core plotting engine, without success. Your best bet is to talk to a gnuplot session as you would type commands on the keyboard. This command interface is clearly defined, hard to get wrong, and offers all the functionalities you have in gnuplot. The only drawback is that since a Unix pipe is unidirectional, it is very hard to get an error status back from gnuplot. gnuplot_i is only implementing this thin layer around the pipe, with no error reporting.

The idea of talking to a gnuplot session through a pipe is not new, various implementations have been circulating on the Usenet since a while. gnuplot_i is only a factorization of the best efforts seen so far to achieve such an interface, it is nothing particularly original or fancy. gnuplot_i is placed in the public domain in the hope that it can be useful to somebody. Thanks to the many contributors for bug reports, suggestions and additional code!

## Reference
You can browse here the reference documentation for the two source files:

    gnuplot_i.c
    gnuplot_i.h

## Installing gnuplot_i
Follow these steps to compile the module:

Edit the Makefile to indicate the C compiler you want to use, the options to provide to compile ANSI C. If you want to use gcc, you have nothing to change.
Type make to make gnuplot_i.o.
Type make tests to make the test programs.
Type 'test/anim' or 'test/example' to get an overview of the possibilities offered by this module.
To use the module in your programs, add the following line on top of your module:
    
    #include "gnuplot_i.h"
And link your program against the gnuplot_i module by adding gnuplot_i.o to the compile line.

## Tutorial
The procedure to follow to display graphics in a gnuplot session is:

### Open a new session
Open a new gnuplot session, referenced by a handle of type (pointer to) `gnuplot_ctrl`. This is done by calling gnuplot_init():

    gnuplot_ctrl * h ;
    h = gnuplot_init() ;
`h` will be used as the handle to the gnuplot session which was just opened, by all further functions.

Notice that the default behaviour of gnuplot_i is to check for the presence of an environment variable called DISPLAY, which is mandatory under X11 to display with gnuplot. If the module cannot found any DISPLAY variable in the environment, you get a warning on stderr.

### Send optional configuration commands
Send optionally display configuration orders. The following functions are available:

`gnuplot_setstyle()` sets the plotting style of the next plots
`gnuplot_set_xlabel()` sets the X label for the next plots
`gnuplot_set_ylabel()` sets the Y label for the next plots
Examples:

    gnuplot_setstyle(h, "impulses") ;
    gnuplot_set_xlabel(h, "my X label") ;
    gnuplot_set_xlabel(h, "my Y label") ;
The most useful routine is probably `gnuplot_cmd()`, which allows you to send character strings to gnuplot as though they were typed in by a user. This routine works in a printf fashion, accepting the same kind of format string and variable number of arguments. The prototype is:

    gnuplot_cmd(handle, command, ...)
Example:

    char myfile[] = "/data/file_in.dat" ;
    int  i ;
    
    gnuplot_cmd(handle, "plot '%s'", myfile);
    for (i=0 ; i<10 ; i++) {
        gnuplot_cmd (handle, "plot y=%d*x", i);
    }
The following commands request the output to be done to a postscript file named 'curve.ps':

    gnuplot_cmd(h, "set terminal postscript") ;
    gnuplot_cmd(h, "set output \"curve.ps\"") ;
Using `gnuplot_cmd()`, it should be fairly easy to add up some more configuration-related functions whenever needed.

### Send display commands
Send some display orders: either functions or doubles, or double points. The following functions are available:

`gnuplot_plot_slope()` to display a slope
`gnuplot_plot_equation()` to display an equation
`gnuplot_plot_x()` to display user-defined 1d data with a single variable. Input is a list of values, assumed regularly spaced on the X-axis.
`gnuplot_plot_xy()` to display user-defined 1d data with two variables. Provide x and y through two arrays of doubles.
`gnuplot_resetplot()` requests the current gnuplot display to be cleared before next display is done.

### Close the session
Close the gnuplot session by calling gnuplot_close on the session handle. This is very important, since otherwise all temporary files will NOT be cleaned for /tmp and /var/tmp.

    gnuplot_close(h) ;
Notice that it is possible to open several gnuplot sessions from the same program. Just be careful then to which session you are talking when using functions. Example:

    h1 = gnuplot_init() ;
    h2 = gnuplot_init() ;

    gnuplot_plot_equation(h1, "sin(x)", "sine on first session");
    gnuplot_plot_equation(h2, "log(x)", "log on second session") ;
    sleep(3) ;
    gnuplot_close(h1) ;
    gnuplot_close(h2) ;
Never forget to close opened sessions! This would leave some temporary files. Hopefully, your temporary directories should be cleaned automatically every now and then, but best is simply not to forget to close all windows before leaving the house.

User interactions are not part of gnuplot_i. Feel free to use your own.

No timing mechanisms are provided to leave the outputs on screen until e.g. a key is pressed. I leave it up to gnuplot_i users to provide that kind of functionality in their own application, depending on the kind of interaction they need.

No programming so far of 'splot' related commands. Should you implement some, I would recommend making use of the 'plot' related commands as a canvas.

## Example: generating web plots on the fly
gnuplot can be used to generate PNG images of plots instead of plotting to a normal window session. This requires to have linked your version of gnuplot against the PNG library (freely available). Look up the gnuplot documentation about PNG support.

The procedure is simple: open a new session, set the terminal type to PNG, set the output file name, and plot your data. Close the session, you are done. This translates to something like:

    gnuplot_ctrl * g = gnuplot_init();

    gnuplot_cmd(g, "set terminal png");
    gnuplot_cmd(g, "set output \"sine.png\"");
    gnuplot_plot_equation(g, "sin(x)", "Sine wave");
    gnuplot_close(g);
