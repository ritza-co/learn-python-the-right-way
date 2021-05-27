# Learn Python the Right Way

This is the source of the book available on https://learnpythontherightway.com.

It is a work-in-progress translation of How to Think Like a Computer Scientist ([GitHub](https://github.com/csev/py4e), [Website](https://openbookproject.net/thinkcs/python/english3e/index.html).)

Our goals are to allow students to follow the entire book on Replit.com. This is a controversial stance as learning how to install and configure programming languages is an important part of learning to be a software engineer, but it is also a huge sticking point for beginners. Especially with remote learning, many get frustrated by virtual environments, conflicting interpreters, and mangled PATHs before they discover the joy of coding.

## Progress 

So far we have done a semi-automated conversion of the RST source files to markdown - although RST has many advantages over markdown, many readers are familiar with markdown and we think this is an important part of allowing more contributions and improvements, though it does mean we have less control over e.g. line numbers and other fancy layout techniques.

The conversion process left some bad errors:

* Many code samples (those with line numbers) are missing in our version and need to be manually copied across.
* In many places hard spaces have been added (`e.g. the source text           looks like this`). This is usually ok as markdown ignores consecutive spaces, but sometimes it happens inside strings (`"something like      this"`) and then it breaks the rendered versions too.

We also want to make some other changes:

* Replit doesn't yet have all the debugging features of the local IDE used in the original book, so we need to adapt some of the code showing how to set breakpoints, single step through code, etc.
* Many of the images are low resolution, so we are re-creating many diagrams using Excalidraw, and retaking screenshots where necessary.
* Some of the exercises need to be updated - e.g. in Chapter 12 the example with Calendar locales doesn't work on Replit (or on many local computers that do not have a Spanish locale installed).
* Replit by default always runs the `main.py` file, but the book tells students to create main files with different names. We need to change these to `main.py` or exaplain how to use the `.replit` file to configure which file the "Run" button will target.

## Help and contributions

Pull requests to this repository are welcome. If you would like to coordinate with others working on the book, please join our Community Slack Group which includes learners taking the course. You can find more details about the course [here](https://ritza.co/writing-for-coders-coding-for-writers-course.html).
