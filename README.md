# Thonny-Debugging-Using-Remote-SSH
Thonny supports developing Python scripts on remote computers using SSH but the debugger is not available. This is a work around on to perform remote debugging.

## Background
I was use to using Thonny's remote connection to Raspberry PI PICO's developing MicroPython applications with no debugger support given the small footprint of MicroPython. However, when I went to use Thonny on my Raspberry PI P400 to do code development on my Raspberry PI ZERO, I discovered that even though I could set break points, running the debugger did not break on them. It appears that this is a known limitation of Thonny with Remote SSH interpreters. 

I investigated options and the two I found seemed overkill when I was use to Thonny. 
- Virtual Studio provides remote Python debugging. 
- PyCharm also does but you need to purchase their professional edition.

This lead me to the following solution.

## pdb — The Python Debugger
[pdb — The Python Debugger](https://docs.python.org/3/library/pdb.html) comes with Python installers. This means it's ready to be used with any Python Script. The big difference with Thonny debugging is that you need either programmatically set a breakpoint in the code or explicitly set the breakpoint while in the (PDB) prompt.

Here are some references in learning how to use PDB

- [The Python Debugger Documentation](https://docs.python.org/3/library/pdb.html)
- [PDB Cheat Sheet](https://kapeli.com/cheat_sheets/Python_Debugger.docset/Contents/Resources/Documents/index)
- [How to - Python Debugging With Pdb](https://realpython.com/python-debugging-pdb/)
- [How to - Debugging Python Apps](https://sunscrapers.com/blog/python-debugging-guide-pdb/)

## Configuring Thonny for Remote SSH Access
I have a repo showing how to do this in [Configuring-Thonny-Remote-Interpreter](https://github.com/DS256/Configuring-Thonny-Remote-Interpreter)

## Example Setup of Thonny Remote with PDB Debugging
There are two programs that are used for this:
- Thonny accessing the remote computer where the Python scripts are stored.
- SSH Terminal session where Python script is debugged using PDB

On Thonny, I find it best to have Files, Shell and Outline enabled under view.

### Setting Up SSH Terminal Window
Here is the way I've configured the SSH terminl into the PI ZERO from my P400
- Change the prompt. I used `PS1=">>$ "` to disquish it from other SSH sessions I may have open
- Right click the banner of the SSH terminal window, select `Layer` and choose `Always on top`. This way you don't have to hunt for it.
- Size the window the same as Thonny Shell and place the SSH window over the Shell. This is a personal preference. Obviously you could have a large SSH window open beside Thonny. It does provide more of the look and feel of Thonny.

## Example
Here is the example code we will use
```
#!/usr/bin/env python3

import os

def get_path(filename):
    """Return file's path or empty string if no path."""
    head, tail = os.path.split(filename)
    return head

filename = __file__
filename_path = get_path(filename)
print(f'path = {filename_path}')
print("Done")
```
When put this into Thonny, and run it, here is what shows up in the Thonny shell.
```
>>> %Run test.py
  path = /home/paul
  Done
>>> 
```
![Screenshot from 2024-03-13 19-11-16](https://github.com/DS256/Thonny-Debugging-Using-Remote-SSH/assets/32932990/a6f421ce-0b33-4fee-9842-6f24d500c161)

This is the normally way we do things except when we are debugging remotely.

Bring your SSH window over the shell.

You run the program, not from Thonny, but from the SSH terminal window `python test.py`

![Screenshot from 2024-03-13 19-25-54](https://github.com/DS256/Thonny-Debugging-Using-Remote-SSH/assets/32932990/39cc7c35-2387-4783-abc6-7a70a897d83a)

We'll now add a breakpoint between lines 7 and 8 in get_path()

![Screenshot from 2024-03-13 19-16-40](https://github.com/DS256/Thonny-Debugging-Using-Remote-SSH/assets/32932990/265bc1fe-f688-4364-bf7c-9ac1b2bb0248)

Note that the breakpoint function is recoginized as a Python function. `breakpoint()` also the performs the `import pdb` as well. Make sure you save the file.

> If you want, you can try running the code again with breakpoint() in place. You will not see anything and if you press enter in the shell you will get a PDB error. PDB and shell seem to have incompatible line input routines

Now from the SSH Window run the program.
```
>>$ python test.py
> /home/paul/test.py(9)get_path()
-> return head
(Pdb) 
```
![Screenshot from 2024-03-13 19-20-58](https://github.com/DS256/Thonny-Debugging-Using-Remote-SSH/assets/32932990/6378e614-7a23-43d0-a891-692b874e28e2)

This tells us we are about to execute line (9) in the function `get_path()` and that this statment is `return h`.

We can now explore the Python script through PDB.
```
(Pdb) p head, tail
('/home/paul', 'test.py')
(Pdb) w
  /home/paul/test.py(12)<module>()
-> filename_path = get_path(filename)
> /home/paul/test.py(9)get_path()
-> return head
(Pdb) a
filename = '/home/paul/test.py'
(Pdb) c
path = /home/paul
Done
>>$ 
```
In this sequence we:
- Printed the values for head and tail (p head,tail)
- Showed the current position and stack trace (w)
- Show the arguements for the current function (a)
- Continued to the next breakpoint or the program ends (c)

So, you can see, there is a fair amount you can do in PDB.

## Conclusion

This is a workaround in order to perform debugging when you need to work remotely from a different platformm. However, PDB is quite versatile and provides features you can't find with the non-interactive debugger which Thonny uses. 
