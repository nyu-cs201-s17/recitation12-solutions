Solutions
-----
There is an error in the function `qsort_cmp`. The return statement should read `return strcmp(*(const char **)a, *(const char **)b);`.

We compile as `gcc -g -Wall -Wextra -std=c99 multisort.c -o sorter`. There are two warnings. One is an unused parameter `char *outfile` in the function `mergewords` (this will be used in the second version of this program), and the other is that `mergewords` returns `int` but there is no return statement. We can't fix the first warning (yet), but the second one can be fixed by adding a return. We will add `return 0;`.

The reason that the contents aren't sorted is because of the fact that when a process (an instance of an executable) calls `fork`, its data (eg stack, heap) is copied verbatim into the child process, but neither process can access the other process's memory.

The parent sorts the first half of the list, and the child sorts the second half of the list. Both store their results in their respective halves, then the child exits. All the work the child did in sorting is not seen in the parent. The parent still has an unsorted second half.

### Fixing Ben's multi-process program

One solution is to write the program in a multi-threaded manner rather than multi-process. This technique will not be covered here.

The other solution is to write out the child's sorted data to a file, let the child exit, and then the parent reads in the sorted data from the file. This is implemented in `multisort-file.c`.

You may notice that we call `malloc` a few times in this program, but never `free`. If you didn't notice this, you should always run your program through Memcheck (via `valgrind`). There is also the caveat that the child has memory that was alloc'd, but exits without freeing it. This also needs to be fixed. You can detect such errors through Memcheck as well with the `--trace-children=yes`, `--leak-check=full`, and `--show-leak-kinds=all` flags. These errors are fixed in `multisort-file.c`.

Feel free to run a diff between the two files to see what changes were made.

### Exec a different program

The algorithm is to have the process fork into a parent and a child. The parent will wait for all children to finish, read in the two input files, and merge them. The child will fork into another parent and child. The child's child will sort the second half of the input file, and the child's parent will sort the first half of the input file. Then as the parent waits for all children, the two halves are in the two files (assuming no errors occured).

The solution is implemented in `multisort-exec.c`. Please note that there are memory leaks in the program `sort` that you may see if you run Memcheck on the code. We cannot do anything about this.
