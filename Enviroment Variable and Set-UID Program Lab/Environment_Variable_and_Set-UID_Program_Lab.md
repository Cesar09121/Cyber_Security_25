# Environment Variable and Set-UID Program Lab

## Environment Variable Operations

![](../lab3_images/task1/printenv.jpg)

After applying command `printenv`, what I see are:

- The list of environment variables
- USER is root at the time because I'm switched to root

Command `printenv` shows all of the environment variables that are currently in the shell. These variables are used by programs for knowing the user name, where files are located and which shell is being used.

![](../lab3_images/task1/printPWD.jpg)

Command "printenv PWD" shows where we are in the files system, which means I was currently in the root home directory.

![](../lab3_images/task1/export.jpg)

After exporting, command `printenv my_lab` prints `lab3`. `export` makes the variable part of the environment so other programs and child shells can use it.

![](../lab3_images/task1/unset.jpg)

After using `unset`, `printenv my_lab` prints nothing, which means `unset` removes a variable from shell environment. This surprises me because the variable's deleted without any notification or sign.

## Passing Environment Variable from Parent Process to Child Process

This task shows how a child process inherits its environment variables from its parent process.

![](../lab3_images/task2/setup.jpg)

Command `gcc myprintenv.c` is compiled and shows no errors so the code is ready to run. I use the command `./a.out > file` to run the program and save the environment variables into a text file named file.

![](../lab3_images/task2/code.jpg)

I use `nano` command to open file `myprintenv.c` to edit it. The editing includes commenting out `printenv()` statement in the process case and uncommenting "printenv()" statement in the parent process case.

![](../lab3_images/task2/commentout.jpg)

I use the command `./a.out > file2` to run the program again. This run save only the parent's environment so I can compare it with the first output. After using command "diff file file2" to compare 2 outputs, I see that there's no differences which mean the environment variables are identical. This confirms the child inherits a copy of the parent's environment.

## Environment Variables and `execve()`

This task demonstrates how the environment variables are affected by executing a program using `execve()` function.

![](../lab3_images/task3/step1.jpg)

This is the program `myenv.c`. After compiling, this program prints nothing (no environment variables are printed because of NULL for environment), which means the child process didn't receive the environment.

![](../lab3_images/task3/step2.jpg)

Next, I use command `nano myenv.c` to open the file "myenv.c" and change the invocation of execve() by passing environment `environ`.

![](../lab3_images/task3/step3.jpg)

After compiling, I can see the output shows environment variable such that PATH, USER and HOME. I also notice that after passing `environ`, the program prints out full env. This means the child inherited parent environment variables. Therefore, the environment variables are not automatically inherited and they must be explicitly passed.

## Environment Variables and system()

This task includes how `system()` affects environment variables and confirm if a program invoked `system()` inherits environment variables.

![](../lab3_images/task4/code.jpg)

I created a program named `mysystem.c` to test out `system()` function. Then, I compiled and run the program and watch the output.

![](../lab3_images/task4/code2.jpg)

I can see the shell started by `system()` prints the environment list. `system()` runs a shell that inherited the caller's environment. So, the program run by `system` sees the same environment variables as the calling process.

## Environment Variables and `Set-UID` Programs

This task shows how environment variables behave when running `Set-UID` root program.

![](../lab3_images/task5/test.jpg)

I wrote a program named `test.c` that prints all environment variables. Then I compiled it, changed its owner to root.

![](../lab3_images/task5/step2task5.jpg)

Now, the files shows the `Set-UID` bit (s in the permission).

![](../lab3_images/task5/test2.jpg)

I export the environment variables as the regular user:

`export PATH=/cesarle/test:$PATH`
`export LD_LIBRARY_PATH=/test/lab:$LD_LIBRARY_PATH`
`export LAB=lab5`

Then, I run the `test` program. It printed a list of environment variables, including the variables I set earlier.

## The `PATH` Environment Variable and `Set-UID` Programs

This task shows how modifying the `PATH` environment variable can be used attack a `Set-UID` program. If a privileged program uses `system()`, the shell will search for commands based on `PATH`. If the attacker place a fake program earlier in the search path, the `Set-UID` program may run it instead of the legitimate one.

I wrote a file named `task6.c` that calls `system("ls")`.

![](../lab3_images/task6/step1.jpg)

Then I compiled the program and made it `Set-UID` and owned by root.

![](../lab3_images/task6/step4.jpg)

This gives the binary the `s` bit which means it will run with root privileges.

![](../lab3_images/task6/step6.jpg)

I created a fake program named `ls.c` that tries to execute the root shell when executed.

![](../lab3_images/task6/step7.jpg)

I can see that `ls` exists in the working directory.

![](../lab3_images/task6/step8.jpg)

Next, I modified the `PATH` environment variables, run command `export PATH=.:$PATH` to trick the system into running the fake program that I created.
When `ls` is called, the shell wil try the fake `ls` first. Then, I execute the vulnerable `Set-UID` binary and I can see the message `Attack!!`.
Then, I check if the privileged file action occurred by using command `/bin/ls -la /tmp/successful.txt`. The files exists which mean the attack worked.

## The `LD_PRELOAD` Environment Variable and `Set-UID` Programs

This task shows how the dynamic loader handles environment variable `LD_PRELOAD` and whether a preloaded library can override a function when the target program is run normally and in privileged (`Set-UID`) contexts.

![](../lab3_images/task7/s1.jpg)

I used the program named `mylib.c` to implement a fake `sleep()` function.

![](../lab3_images/task7/s2.jpg)

Then, I compiled the fake program into a library using command `gcc -fPIC -g -c mylib.c` and `gcc -shared -o limylib.so.1.0.1 mylib.o -lc`.

![](../lab3_images/task7/s4.jpg)

I wrote a program named `myprog.c` that calls `sleep()` function and compiled it.
