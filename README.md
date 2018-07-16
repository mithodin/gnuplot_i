# gnuplot_i
gnuplot interface in ANSI C

Forked from Nicolas Devillard's gnuplot_i (http://ndevilla.free.fr/gnuplot/)

gnuplot is a freely available, command-driven graphical display tool for Unix. It compiles and works quite well on a number of Unix flavours as well as other operating systems.

The following module enables sending display requests to a gnuplot session through simple ANSI C calls.

Notice that gnuplot_i talks to a gnuplot process by means of POSIX pipes. This implies that the underlying operating system has the notion of processes and pipes, and advertizes them in a POSIX fashion. Since Windows does not respect this standard, this module will not compile on it, unless you have a compiler that offers a popen call on that platform or simulates it.
