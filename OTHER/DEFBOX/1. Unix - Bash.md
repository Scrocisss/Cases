# 1. Exit code

### Task

Determine the exit code returned by the program /defbox-course/Foundations/bin/exit_code.

### Theory

In Unix, an exit code (or exit status) is a numeric value returned by a command or script to indicate its outcome.
A successful execution typically returns 0, while non-zero values indicate errors or specific failure conditions. The exit code can be accessed using $? in the shell. Standard conventions include 1 for general errors, 2 for misuse of shell commands, and values 128+N for signals (e.g., 137 for SIGKILL).

### Solve

Задача учит тому, что мы можем получить код завершения программы используя символ &, вот так:

```bash
./exit_code &

[1]+  Exit 47                 ./exit_code
```

### TIP 

You can chain commands based on the success or failure of the previous command using && (only runs the next command if the previous one succeeded) or || (only runs the next command if the previous one failed).

# 2. Pipes

### Task
In the /defbox-course/Foundations/bin/pipes/ directory, use a pipe to redirect output from pipes_generator to pipes_reader.

### Theory
In Unix, a pipe (|) is a mechanism that allows the output of one command to be used as the input for another. It enables chaining commands together to perform complex tasks efficiently. Pipes facilitate inter-process communication without creating temporary files.

### Example

```bash
$ ls -l | grep "txt" | wc -l
```

In this example, ls -l lists files, grep "txt" filters the results, and wc -l counts the number of lines.
Pipes operate in a unidirectional, sequential manner and process data in a streaming fashion, improving performance by handling data in chunks.

### Solve

```bash
defbox@bash-execs-defbox:/defbox-course/Foundations/bin/pipes$ file pipes_reader 
pipes_reader: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=61f01f99ed80921bcdd7e89cef0dbd75099aab4f, for GNU/Linux 3.2.0, not stripped
defbox@bash-execs-defbox:/defbox-course/Foundations/bin/pipes$ file pipes_generator 
pipes_generator: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=2dd3647472fc177459b590f220a515e84479a672, for GNU/Linux 3.2.0, not stripped
defbox@bash-execs-defbox:/defbox-course/Foundations/bin/pipes$ ./pipes_generator  | ./pipes_reader 
p1p32 0V3rl0RD
```

Сначала я увидел, что оба файла исполняемые, значит нам нужно перенаправить результат одной программы в другую.

### TIP

You can use the "tee" command to redirect output to both the terminal and a file simultaneously. Example: $ command | tee output.txt

# 3. Parallel run

### Task
Run the /defbox-course/Foundations/bin/parallel_run command at least 10 times within 1 second.

### Theory
You can run multiple commands in parallel in a Unix shell using the ampersand (&), subshells, or xargs/parallel utilities.

### 1. Using & (Background Execution)
Run commands in the background by appending & at the end of each command.

```bash
command1 & command2 & command3 &
wait  # Waits for all background jobs to finish
```

### 2. Using Subshells
Group commands in subshells and run them in parallel.

```bash
(command1 &) (command2 &) (command3 &)
wait
```

### 3. Using xargs for Parallel Execution
Run commands in parallel using xargs with the -P option to control concurrency.

```bash
echo "command1 command2 command3" | xargs -n 1 -P 3 bash -c
```

Each method allows efficient parallel execution depending on your use case.

### Solve

Суть задачи заключается в том, чтобы запустить программу 10 раз менее чем за 1 секунду, сделать это можно следующим образом:

```bash
defbox@bash-execs-defbox:/defbox-course/Foundations/bin$ ./parallel_run & ./parallel_run & ./parallel_run & ./parallel_run & ./parallel_run
 & ./parallel_run & ./parallel_run & ./parallel_run & ./parallel_run & ./parallel_run
[1] 2026
[2] 2027
[3] 2028
[4] 2029
[5] 2030
[6] 2031
[7] 2032
[8] 2033
[9] 2034
P4R4LL3l M19HtY
```

Нашёл более компактное решение, которое позволит запустить 10 процессов одновременно:

```bash
seq 10 | xargs -P10 -I{} ./parallel_run
```

seq 10 генерирует числа от 1 до 10.
xargs -P10 запускает до 10 процессов одновременно.
-I{} позволяет игнорировать входные данные.

### TIP

You can use the "ps" command to check if your commands are running in the background. This can help you confirm that all processes were started correctly.

# 4. Background job

### Task
Run the /defbox-course/Foundations/background.sh script in the background so it continues running even after you close your terminal session. The check will terminate all SSH sessions and then inspect the process list.
 

### Theory
You can run programs in the background using screen and tmux, which provide persistent, detachable terminal sessions.

### 1. Using screen
screen allows you to start a session, run programs, detach from the session, and reattach later.

Start a new session and run a program:

```bash
screen -S mysession my_program
```

Detach from the session (press Ctrl + A, then D).
Reattach to the session later:

```bash
screen -r mysession
```

### 2. Using tmux
tmux (Terminal Multiplexer) works similarly to screen but offers more advanced features.

Start a new session and run a program:

```bash
tmux new -s mysession my_program
```

Detach from the session (press Ctrl + B, then D).
Reattach to the session later:

```bash
tmux attach -t mysession
```

### 3. Using nohup
If you don’t need a full persistent session manager like screen or tmux, you can use nohup to run a single command that continues after you log out. For example:

```bash
nohup my_script.sh &
```

This prevents the process from stopping when the terminal closes, making it ideal for simpler, long-running tasks.
All this tools allow your programs to keep running even if you disconnect from the system, making them useful for long-running tasks and remote work.

### Solve

```bash
defbox@bash-execs-defbox:~$ nohup /defbox-course/Foundations/background.sh &
[1] 4004
defbox@bash-execs-defbox:~$ nohup: ignoring input and appending output to 'nohup.out'

defbox@bash-execs-defbox:~$ ps aux | grep back
root          24  0.0  0.0      0     0 ?        I<   10:29   0:00 [writeback]
defbox      4004  0.0  0.0   2892   980 pts/0    S    11:14   0:00 /bin/sh /defbox-course/Foundations/background.sh
defbox      4007  0.0  0.2   7008  2156 pts/0    S+   11:14   0:00 grep --color=auto back
defbox@bash-execs-defbox:~$
```

Решение простое исходя из теории, но автоматическая проверка почему-то не работает :(

Проверка сработала, когда выполнил:

```bash
screen -S mysession ./background.sh
```

ctrl+A
D

И проверка:

```bash
defbox@bash-execs-defbox:/defbox-course/Foundations$ ps aux | grep back
root          24  0.0  0.0      0     0 ?        I<   10:29   0:00 [writeback]
defbox      4348  0.0  0.2   7740  2736 ?        Ss   11:17   0:00 SCREEN -S mysession ./background.sh
defbox      4349  0.0  0.0   2892   952 pts/1    Ss+  11:17   0:00 /bin/sh ./background.sh
defbox      4615  0.0  0.2   7740  2680 ?        Ss   11:19   0:00 SCREEN -S mysession ./background.sh
defbox      4616  0.0  0.1   2892   992 pts/2    Ss+  11:19   0:00 /bin/sh ./background.sh
defbox      4622  0.0  0.2   7008  2148 pts/0    S+   11:19   0:00 grep --color=auto back
defbox@bash-execs-defbox:/defbox-course/Foundations$
```

### TIP

You can list your background jobs with the "jobs" command. If you want to bring a background job to the foreground, use the "fg" command.















