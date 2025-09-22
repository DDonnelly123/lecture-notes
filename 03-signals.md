# Signals

## Signals Overview

* `man 7 signal` for the overview.
    * Signals are notifications with specific meanings. Programs and the kernel can send signals to
      itself or other programs.
     * Wonka Golden Ticket Example:
        * Parent processes spawn children to look for golden ticket
        * Parent creates a signal handler
        * searches, and if it finds the golden ticket, it sends a signal to the parent (e.g., `kill(getppid()`)
    * Some examples (scroll down to `Standard signals`)
        * `SIGINT`: CTRL-C
        * `SIGKILL`: `kill` call
        * `SIGSEGV`: Invalid memory reference
        * Etc.
    * How to send a signal (scroll up to `Sending a signal`)
        * `raise()`: to itself
        * `kill()`: to a process
        * Etc.
    * Signal handler
        * `man sigaction`
        * The important part is filling out `struct sigaction`. The first member is the function
          pointer to your signal handler.
* When using signals, you need to use signal safe functions.
    * `man signal-safety` for the list of async-signal-safe functions

## Activity: `sigaction()`

* Write a program that creates a child process. The program should use `sigaction()` and install
  a signal handler that catches `SIGINT` and prints out "CTRL-C pressed." The parent should run
  an infinite loop with `sleep()`. The child should have an infinite loop that periodically
  sends `SIGINT` to the parent (with `sleep()`). Experiment with `CTRL-C` as well.
    * Use `write` instead of `printf`. The file descriptor for stdout is `STDOUT_FILENO`.
      `printf` is not signal safe.
    * No need to set any flags, i.e., `0` is fine.
    * No need to set the masks either. `man sigsetops`.
    * Use `btop`  or `pkill` to kill the processes.
* Example code

  ```c
  #define _POSIX_C_SOURCE 1

  #include <signal.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include <string.h>
  #include <sys/types.h>
  #include <unistd.h>

  void handle_sigint(int);

  static char *buffer = "CTRL-C pressed.\n";
  void handle_sigint(int signum) { write(STDOUT_FILENO, buffer, strlen(buffer)); }

  int main() {
    pid_t pid;

    pid = fork();
    if (pid == -1) {
      perror("fork");
      exit(EXIT_FAILURE);
    }

    if (pid) {
      struct sigaction handler;

      handler.sa_handler = handle_sigint;
      handler.sa_flags = 0;
      sigemptyset(&handler.sa_mask);
      sigaction(SIGINT, &handler, NULL);

      while (1) {
        sleep(5);
      }
    } else {
      while (1) {
        if (kill(getppid(), SIGINT) == -1) {
          perror("kill");
          exit(EXIT_FAILURE);
        }
        sleep(5);
      }
    }
  }
  ```
