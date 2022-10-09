# Running our Commands with Bash

# 4. Using `bash`

_The lab system has already been configured so that every user profile is using Bash by default. Follow these steps when doing the lab on your own system, or if you are following this lab at a later date._

Before we get ahead of ourselves, lets do some review: A shell is a program that interprets the commands you give to the system to do some chunk of work. There are many different shells out there, but the most popular by far is `bash`. `bash` is short for the "**B**ourne **A**gain **Sh**ell", an industry-standard shell that has many many functional improvements over alternatives like `Qshell`, `sh`/`ksh` (the default shell), or `csh`.

Now that we know how to SSH onto the system, we want to make sure that we are using `bash` as our default shell. Why? It has lots of helpful features like tab-completion, command searching, unlimited command history, and more.

The first thing we need to do is ensure that we have `bash` installed on the system. You should be able to run a command like `which bash`, which should tell you if that command can be found in any of the locations on your `PATH` environment variable. If it can't be located, that probably means you don't have it installed on your system.

_Because you need *ALLOBJ authority to install open-source software on IBM i, the installation of `bash` has already been done for you in this workshop. You may need to install it on your own system when you get back home._

To install open-source software, we _can_ use ACS in the same way we installed `yum` on the system. But that is for people who don't want to develop open-source applications, not a budding power-user like you! We are going to use the command line to install `bash`. To do so, we simply run the command:
```bash
yum install bash
```
This will tell `yum`, our package manager, to go out to the repositories it knows about (in our case, only the official IBM repository) and look for a package named `bash`. It should find such a package, and then tell you what it is about to download. This may include not only the package you are downloading, but any dependencies that are required to run your desired package. It will also tell you how large the download will be, and once decompressed, how much space in the IFS it will take. Finally, it will prompt you to press `y` if you want to continue, or `N` (or enter) if you don't want to continue.

Once you accept the download, it will download and install the software on your system. Most open-source packages will install binaries into `/QOpenSys/pkgs/bin`, and it will be the same for `bash`.

But just because we have `bash` on the system, doesn't mean we will be using it by default when we SSH onto the system! In order to set our default shell, we need to use a handy procedure that has been created to do just that: `QSYS2.SET_PASE_SHELL_INFO`. This procedure takes two string parameters: The first is the name of the user you want to change the shell for, and the second is the location of the that shell program. Hopefully you have ACS so you can run SQL scripts (otherwise, let me know and I will run the procedure for you), and run:

```
QSYS2.SET_PASE_SHELL_INFO('USERID', '/QOpenSys/pkgs/bin/bash')
```

After you run this procedure, you can quit your SSH session, then reconnect. When you do, you should have a prompt indicating that you are now using `bash`! Of course, `bash` isn't absolutely required. But if you want to develop open-source apps like an open-source developer, than `bash` is definitely the way to go. Using the default shell is also acceptable. But using 5250 and QSH is _not_ a good way to interact with PASE or to develop open-source software.

## File editing in the terminal

`nano` is an editor that runs in your terminal. To run it, you can simply type the nano command and press enter. However, you will have to provide a file name when you save the file. To create a new file / open an existing file when you open nano, just type the name of the file after the command:
```
nano myfile.txt
```
This will open a file with that name, that you can then save with a simple shortcut.

To save a file in nano, use Ctrl + o, then exit with Ctrl + x. To make sure the file changes took, you can always run something like:
```
cat myfile.txt
```
Which will spit out the entire contents of your file to the terminal.

---
Next: [Creating our LoopBack Application](d.loopback-application.md)