# Imperium
My project
Imperium- VCS
Introduction
In software engineering, version control (also known as revision control, source control, or source code management) is a class of systems responsible for
managing changes to computer programs, documents, large web sites, or other collections of information. Version control is a component of software
configuration management.
~ The ever reliant Wikipedia.
In this 10 day programming exercise, this guide isn't meant to be a tutorial for you. Think of it as a document which will provide you a gentle nudge in the right
direction.
This guide will help you visualise the procedural complexity of a VCS and we will try to emulate the functionality of the OG VCS of them all -
GIT.
We will go through the following steps in the course of this tutorial:
Shell Scripting With Bash
Initialising an empty Imperium Repo (Init)
Staging Changes for the Commit (Add)
Commiting them (Commit)
Checking out to a specific commit (Checkout)
REVERTING a COMMIT (revert)
Status
Conclusion and further innovations possible.
Making your own CLI
The first step towards making a functional version control system is to prepare your own command-line interface (imperium add, imperium commit, etc).
So to proceed we need two shell scripts, one to build your project and another to run your commands in the terminal. The final goal is that the user will download your
source project, and run the build script to “install” the VCS. Once this is done, the imperium terminal commands become available to be used. How exactly this can be
achieved will be discussed shortly.
The second shell script is what will be executed every time you type an imperium command in your terminal. This script basically acts as a middleware between your
commands and the C++ executable that you built with the build script.
In this project, we will have a main executable file which needs to be run each time your self-made commands are used. We do not want the user to manually compile
and run the C++ program themselves every time they want to perform an operation. This needs to be automated through a shell script.
We also want our commands to run from any directory, not just the root directory of our project, and hence it needs to be installed in our system.
Steps in the build script
1.
2.
3.
4.
5.
6.
First you need to install two C++ libraries required for SHA-1 hashing, openssl and libssl-dev.
Create a directory in the $HOME folder to store the compiled C++ program binary and our command script.
Copy the command script from VCS source code to the created directory.
Go the VCS root where the Makefile is and make the project.
Now cd to the new $HOME directory and change permission of the binary to executable.
Put the command script in the ~/.bashrc to permanently add it to your system PATH .
The build.sh file will look something like this:#!/bin/bash
sudo apt-get update
sudo apt-get install openssl -y
sudo apt-get install libssl-dev -y
mkdir -p ~/imperium/bin
cp imperium.sh ~/imperium
cd ..
make
cd ~/imperium/bin || echo "error"
chmod +x main
cd ..
if grep -q "source $PWD/imperium.sh" "$PWD/../.bashrc" ; then
echo 'already installed bash source';
else
echo "source $PWD/imperium.sh" >> ~/.bashrc;
fi
In Unix and Unix-like operating systems, chmod is the command and system call which is used to change the access permissions of file system objects
The +x marks it as an executable file which is necessary to run the scripts.
The make utility requires a file, Makefile (or makefile ), which defines set of tasks to be executed. You may have used make to compile a program from source
code
Our Makefile is supposed to look something like this:
You are adviced to use gcc version 9 and above
CXX
:= g++
CXX_FLAGS := -std=c++17 -ggdb
BIN := bin
SRC := src
INCLUDE :=
LIB
:= lib
LIBRARIES :=
EXECUTABLE := main
all: $(BIN)/$(EXECUTABLE)
run: clean all
@echo "run called"
./$(BIN)/$(EXECUTABLE)
$(BIN)/$(EXECUTABLE): $(SRC)/*.cpp
$(CXX) $(CXX_FLAGS) -I$(INCLUDE) -L$(LIB) $^ -o $ ~/imperium/$(BIN)/$(EXECUTABLE) $(LIBRARIES)
-lssl -lcrypto
clean:
@echo "clean called"
@rm -rf -d ~/imperium/$(BIN)/*
Steps in the command script
1. Create a function called imperium(or whatever you wanna name it). We are using a function here because we can source it in the .bashrc and that allows
us to call imperium without writing the .sh extension, thus giving a real command line feel.The .bashrc entry will look something like this(call this in the
build script).
echo "source $PWD/imperium.sh" >> ~/.bashrc;
1. The imperium function will looks something like this:function imperium(){
DIR=$PWD
export dir=$DIR
cd ~/imperium/bin || echo "Error"
./main "$@"
cd "$DIR" || echo "Error"
}
As you can see, we first store and export the path of the current working directory as an environment variable. This is required as all our paths being used have
this directory as the root.
After that we cd into the $HOME/imperium/bin folder where the compiled C++ binary is. The next step is to execute the binary and pass in all the commands
as command line parameters so we can figure out what command is being called by the user.
Once this is done simply cd back to your repository directory.
init() function
The first command the user enters is imperium revert init to initialize the folder as an imperium repositiory. The function to handle this process is fairly simple.
Lets see what we need to do step-by-step:
1. First check if the folder is already initialized as a repo. To do this we simply check if the .imperium folder exists already or not. This can be done by using the
stat function cpp
The <sys/stat.h> header defines the structure of the data returned by the functions fstat(), lstat(), and stat(). We use it to check directories and files as it is the fastest
way to do this.
Resource
If it does we just print a message and return to main() function. 2. If the previous check returns false then we move on to create a new file and folder, namely:
1. .imperium folder
2. .imperiumIgnore file (optional )
3. A conflict file in the .imperium folder just made, initialised with one line in it containing "false". (Do this step only if you want to implement revert )
The .imperium has all the data and logs required for our VCS to function and the .imperiumIgnore file acts like a .gitignore file, it tells our program
which files to not track.
3. Next we populate the .imperiumIgnore file with some common files and folders:
4. /.imperium
5. /.imperiumIgnore (optional, but highly recommended )
6. /node_modules
7. /.env
Keep in mind that /.imperium has to added to the file for it to work correctly.
4. Now we move on to the .imperium folder and start populating it.
5. Create two files called .commit.log and add.log . These will be used to keep track of the commits and the add cache.
Finally print a success statement.
