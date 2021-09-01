# Chapter 21: A Case Study: Indexing your files


We present a small case study that ties together modules, recursion, files, dictionaries and introduces simple serialization and deserialization.

In this chapter we're going to use a dictionary to help us find a file rapidly.

The case study has two components:

-   A crawler program that scans the disk (or folder) and constructs and saves the dictionary to your disk.
-   A query program that loads the dictionary and can rapidly answer user queries about where a file is
    located.

## 21.1. The Crawler

Near the end of the chapter on recursion we showed an example of how to recursively list all files under a given path of our filesystem.

We'll borrow (and change) that code somewhat to provide the skeleton of our crawler. It's function is to recursively traverse every file in a given path. (We'll figure out what to do with the file soon: for the moment we'll just print it's short name, and its full path.)

```python
# Crawler crawls the filesystem and builds a dictionary
import os

def crawl_files(path):
    """ Recursively visit all files in path """

    # Fetch all the entries in the current folder.
    dirlist = os.listdir(path)
    for f in dirlist:
        # Turn each name into full pathname
        fullname = os.path.join(path, f)  

        # If it is a directory, recurse.
        if os.path.isdir(fullname):       
            crawl_files(fullname)
        else:  # Do something useful with the file
            print("{0:30} {1}".format(f, fullname))

crawl_files("C:\\Python32")
```

We get output similar to this:

```
CherryPy-wininst.log           C:\Python32\CherryPy-wininst.log
bz2.pyd                        C:\Python32\DLLs\bz2.pyd
py.ico                         C:\Python32\DLLs\py.ico
pyc.ico                        C:\Python32\DLLs\pyc.ico
pyexpat.pyd                    C:\Python32\DLLs\pyexpat.pyd
python3.dll                    C:\Python32\DLLs\python3.dll
select.pyd                     C:\Python32\DLLs\select.pyd
sqlite3.dll                    C:\Python32\DLLs\sqlite3.dll
tcl85.dll                      C:\Python32\DLLs\tcl85.dll
tclpip85.dll                   C:\Python32\DLLs\tclpip85.dll
tk85.dll                       C:\Python32\DLLs\tk85.dll
...
```

We'll adapt this now to store the short name and the full path of the file as a key:value pair in a dictionary. But first, two observations:

-   We can have many instances of files with the same name (in different paths). For example, the name index.
    html is quite common. But dictionary keys must be unique. Our solution is to map each key in our    dictionary to a *list* of paths.
-   File names are not case sensitive. (Well, not for Windows users!) So a good technique is to *normalize* 
    the keys before storing them. Here we'll just ensure that all keys are converted to lowercase. Of
    course we'll do the same later when we write the query program.

We'll change the code above by setting up a global dictionary, initially empty: The statement `thedict = {}` inserted at line 3 will do this. Then instead of printing the information at line 17, we'll add the
filename and path to the dictionary. The code will need to check whether the key already exists:

```python
key = f.lower()  # Normalize the filename
if key in thedict:
   thedict[key].append(fullname)
else:   # insert the key and a list of one pathname
   thedict[key] = [fullname]
```

After running for a while the program terminates. We can interactively confirm that the dictionary seems to have been built correctly:

```python
>>> len(thedict)
14861
>>> thedict["python.exe"]
['C:\\Python32\\python.exe']
>>> thedict["logo.png"]
['C:\\Python32\\Lib\\site-packages\\PyQt4\\doc\\html\\_static\\logo.png',
 'C:\\Python32\\Lib\\site-packages\\PyQt4\\doc\\sphinx\\static\\logo.png',
 'C:\\Python32\\Lib\\site-packages\\PyQt4\\examples\\demos\\textedit\\images\\logo.png',
 'C:\\Python32\\Lib\\site-packages\\sphinx-1.1.3-py3.2.egg\\sphinx\\themes\\scrolls\\static\\logo.png']
>>> 
```

It would be nice to add a progress bar while the crawler is running: a typical technique is to print dots to show progress. We'll introduce a count of how many files have been indexed (this can be a global variable), and after we've handled the current file, we'll add this code:

```python
filecount += 1
if filecount % 100 == 0:
    print(".", end="")
    if filecount % 5000 == 0:
        print()
```

As we complete each 100 files we print a dot. After every 50 dots we start a new line. You'll need to also create the global variable, initialize it to zero, and remember to declare the variable as global in the crawler.

The main calling code can now print some statistics for us. It becomes

```python
crawl_files("C:\\Python32")
print()  # End the last line of dots ...
print("Indexed {0} files, {1} entries in the dictionary.".
                    format(filecount, len(thedict)))
```

We'll now get something like

```python
>>> 
..................................................
..................................................
..................................................
....................................
Indexed 18635 files, 14861 entries in the dictionary.
>>>         
```

It is reassuring to look at the properties of the folder in our operating system, and note that it counts exactly the same number of files as our program does!

## 21.2. Saving the dictionary to disk

The dictionary we've built is an object. To save it we're going to turn it into a string, and write the string to a file on our disk. The string must be in a format that allows another program to unambiguously reconstruct another dictionary with the same key-value entries. The process of turning an object into a string representation is called **serialization**, and the inverse operation --- reconstructing a new object from a string ---is called **deserialization**.

There are a few ways to do this: some use binary formats, some use text formats, and the way different types of data are encoded differs. A popular, lightweight technique used extensively in web servers and web pages is to use JSON (JavaScript Object Notation) encoding.

Amazingly, we need just four new lines of code to save our dictionary to our disk:

```python
import json

f = open("C:\\temp\\mydict.txt", "w")
json.dump(thedict, f)
f.close()    
```

You can find the file on your disk and open it with a text editor to see what the JSON encoding looks like.

## 21.3. The Query Program

This needs to reconstruct the dictionary from the disk file, and then provide a lookup function:

```python
import json

f = open("C:\\temp\\mydict.txt", "r")
dict = json.load(f)
f.close()
print("Loaded {0} filenames for querying.".format(len(dict)))

def query(filename):
    f = filename.lower()
    if f not in dict:
        print("No hits for {0}".format(filename))
    else:
        print("{0} is at ".format(filename))
        for p in dict[f]:
            print("...", p)   
```

And here is a sample run:

```python
>>> 
Loaded 14861 filenames for querying.
>>> query('python.exe')
python.exe is at 
... C:\Python32\python.exe
>>> query('java.exe')
No hits for java.exe
>>> query('INDEX.HtMl')
INDEX.HtMl is at 
... C:\Python32\Lib\site-packages\cherrypy\test\static\index.html
... C:\Python32\Lib\site-packages\eric5\Documentation\Source\index.html
... C:\Python32\Lib\site-packages\IPython\frontend\html\notebook\static\codemirror\mode\css\index.html
... C:\Python32\Lib\site-packages\IPython\frontend\html\notebook\static\codemirror\mode\htmlmixed\index.html
... C:\Python32\Lib\site-packages\IPython\frontend\html\notebook\static\codemirror\mode\javascript\index.html
... C:\Python32\Lib\site-packages\IPython\frontend\html\notebook\static\codemirror\mode\markdown\index.html
... C:\Python32\Lib\site-packages\IPython\frontend\html\notebook\static\codemirror\mode\python\index.html
... C:\Python32\Lib\site-packages\IPython\frontend\html\notebook\static\codemirror\mode\rst\index.html
... C:\Python32\Lib\site-packages\IPython\frontend\html\notebook\static\codemirror\mode\xml\index.html
... C:\Python32\Lib\site-packages\pygame\docs\index.html
... C:\Python32\Lib\site-packages\pygame\docs\ref\index.html
... C:\Python32\Lib\site-packages\PyQt4\doc\html\index.html
>>> 
```

## 21.4. Compressing the serialized dictionary

The JSON file might get quite big. Gzip compression is available in Python, so let's take advantage of it...

When we saved the dictionary to disk we opened a text file for writing. We simply have to change that one line of the program (and import the correct modules), to create a gzip file instead of a normal text file. The replacement code is

```python
import json, gzip, io

## f = open("C:\\temp\\mydict.txt", "w")
f = io.TextIOWrapper(gzip.open("C:\\temp\\mydict.gz", mode="wb"))
json.dump(thedict, f)
f.close() 
```

Magically, we now get a zipped file that is about 7 times smaller than the text version. (Compressiion/decompression like this is often done by web servers and browsers for significantly faster downloads.)

Now, of course, our query program needs to uncompress the data:

```python
import json, gzip, io

## f = open("C:\\temp\\mydict.txt", "r")
f = io.TextIOWrapper(gzip.open("C:\\temp\\mydict.gz", mode="r"))
dict = json.load(f)
f.close()
print("Loaded {0} filenames for querying.".format(len(dict))) 
```


Composability is the key...

In the earliest chapters of the book we talked about composability: the ability to join together or *compose* different fragments of code and functionality to build more powerful constructs.

This case study has shown an excellent example of this. Our JSON serializer and deserializer can link with our file mechanisms. The gzip compressor / decompressor can also present itself to our program as as if it was just a specialized stream of data, as one might get from reading a file. The end result is a very elegant composition of powerful tools. Instead of requiring separate steps for serializing the dictionary to a string, compressing the string, writing the resulting bytes to a file, etc., the composability has let us do it all very easily!


## 21.5. Glossary
**deserialization**
Reconstruction an in-memory object from some external text
representation

**gzip**
A lossless compression technique that reduces the storage size of data.
(Lossless means you can recover the original data exactly.)

**JSON**
JavaScript Object Notation is a format for serializing and transporting
objects, often used between web servers and web browsers that run
JavasScript. Python contains a `json` module to provide this capability.

**serialization**
Turning an object into a string (or bytes) so that it can be sent over
the internet, or saved in a file. The recipient can reconstruct a new
object from the data.

