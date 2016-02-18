Standard Modules
Headers

# This is an <h1> tag
## This is an <h2> tag
###### This is an <h6> tag
Emphasis

*This text will be italic*
_This will also be italic_

**This text will be bold**
__This will also be bold__

_You **can** combine them_
Lists

Unordered

* Item 1
* Item 2
  * Item 2a
  * Item 2b
Ordered

1. Item 1
2. Item 2
3. Item 3
   * Item 3a
   * Item 3b
Images

![GitHub Logo](/images/logo.png)
Format: ![Alt Text](url)
Links

http://github.com - automatic!
[GitHub](http://github.com)
Blockquotes

As Kanye West said:

> We're living the future so
> the present is our past.
Inline code

I think you should use an
`<addr>` element here instead.


![image test](http://wp.patheos.com.s3.amazonaws.com/blogs/faithwalkers/files/2013/03/bigstock-Test-word-on-white-keyboard-27134336.jpg)


>>> import sys
>>> sys.ps1
'>>> '
>>> sys.ps2
'... '
>>> sys.ps1 = 'C> '
C> print('Yuck!')
Yuck!
C>
These two variables are only defined if the interpreter is in interactive mode.

The variable sys.path is a list of strings that determines the interpreter’s search path for modules. It is initialized to a default path taken from the environment variable PYTHONPATH, or from a built-in default if PYTHONPATH is not set. You can modify it using standard list operations:

>>> import sys
>>> sys.path.append('/ufs/guido/lib/python')
