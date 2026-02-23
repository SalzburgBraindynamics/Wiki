# MEG : Introduction to Python and what is different / similar to Matlab

If you already have some experience in a programming language like Matlab, using Python will not be a big problem for you. Concepts like variables, loops, functions and how to structure a script or program is the actual hard part of programming. All of the concepts you already know exist in Python as well. They might be a little different but they still follow the same principles.

This is not going to be another Python tutorial, as lots of those already exist on the internet. I especially recommend this one: [https://www.datacamp.com/courses/intro-to-python-for-data-science](https://www.datacamp.com/courses/intro-to-python-for-data-science)

Here is a list of common things to do in in Matlab and how they are done in Python:

### The semicolon

The semicolon is important in Matlab as otherwise you command window gets littered with the result of the function:

```java
>> a = 1 + 2

a =

     3

>> a = 1 + 2;
>> 
```

In python, it is not a problem if you use the semicolon, but it is not necessary:

```py
a = 1 + 2

a = 1 + 2;
```

### Comments

Matlab:

```java
a = 1; % I am a comment
```

Python:

```py
a = 1 # I am a comment
```

If you use spyder, you can use a cell mode similar to Matlab by writing a line comment starting with _#%%_:

```py
#%% First cell
a = 1 # I am a comment

#%% Second cell
b = 2
```

### The Path, Packages, imports

If you want to add a toolbox in Matlab, you copy its content to some folder on your harddrive (or on the storage). You the add it to the "Matlab Path" in order to use it. The functions are then directly available to you.

This works very different in Python. Packages and toolboxes are installed using either _pip_ or _conda_, one of the two available package managers. The package managers install the packages to a central location that is know to the Python interpreter. In order to use the package, you need to import it into Python like this:

```py
import os # Import the os package. This is a package that is included in python
import mne # Import the MNE Python package
```

The other difference compared to Matlab is that when you import a package or a toolbox, its functions are not put into the global namespace.

Here is an example. In Matlab, you would do this:

```java
addpath('path/to/toolbox'); % Tell Matlab where to find the toolbox m files.

fun_from_toolbox('Hello!'); % Call a function from the toolbox
```

In python however, you need to explicitly write from which package the function is from:

```py
import mytoolbox # Import mytoolbox

mytoolbox.fun_from_toolbox('Hello!') # Call a function from the toolbox
```

This looks a bit strange at first but it is a great way to keep all the functions organized

#### How to install your own packages on the bomber

As long as you work on your own computer, you can install packages yourself. On the bomber, the Python environment is shared between all users. This means that individual users cannot install packages.

The solution is easy: create your own environment by using the conda command like this:

```java
create_obob_mne_env /mnt/obob/staff/thartmann/envs/my_private_environment/
```

In order to use it, write this in the command window:

```java
conda activate /mnt/obob/staff/thartmann/envs/my_private_environment
```

Please note that the Spyder icon on the desktop always starts the Spyder in the shared environment. If you want to work with Spyder in your own one, run it from the command-line after activating your environment:

```java
conda activate /mnt/obob/staff/thartmann/envs/my_private_environment
spyder
```

### Defining functions

Matlab:

```java
function return_value = my_function(val1, val2)
if nargin < 2
  val2 = 3
end %if
disp(val1 + val2)
return_value = val1 * val2
```

Python

```py
def my_function(val1, val2=2):
	print(val1 + val2)
	return val1 * val2
```

Here are the differences:

1. Functions are declared with _def_ instead of _function_.
1. Python does **not** state the return variable in the function declaration.
1. Python uses the _return_ statement to return values. In Matlab you need to assign the value you want to return to the variable that you have declared in the function declaration.
1. The function declaration **must** be followed by a colon is Python.
1. The function body (i.e. what the function is doing) **must** me indented.
1. Function arguments can have default values in Python.

### Indentation is mandatory

If you, for example, have an _if_ statement in your code, followed by a block of code that should only be run, if the statement was true, Python or Matlab needs to know, when that block is over. In Matlab, this code block goes from the _if/for/function/while_ statement until the _end_ statement:

```java
b = 1;
if a == 2
  disp('A is two!');
  b = 3;
else
  disp('A is not two :-(');
  b = 1;
end
```

In Python, code blocks are defined by how many spaces they are indented:

```py
b = 1
if a == 2:
	print('A is two!')
	b = 3
else:
	print('A is not two :-(')
	b = 1
```

Additionally:Every statement that is followed by its code-block (i.e. def, if, while, for etc.) ends with a ':'!

### For loops

This is how for loops work in Python:

```py
my_list = ['hello', 1, 20.11]

for cur_item in my_list:
	print(cur_item)
```

So, no index variables anymore. But if you should need one:

```py
my_list = ['hello', 1, 20.11]

for idx, cur_item in enumerate(my_list):
	print(idx + ':' + cur_item)
```
