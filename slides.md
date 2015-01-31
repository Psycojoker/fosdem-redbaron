# RedBaron<br>a bottom up approach to refactoring in python

---

# Me

* Laurent Peuch
* Bram

Verbose version:
[http://worlddomination.be/about/about.html](http://worlddomination.be/about/about.html)

---

# Plan

* Why?
* Solution for the first problem (baron)
* Solution for the second problem (RedBaron)
* Conclusion

---

# Before starting: some reminders

---

# Refactoring (eclipse)

![refactoring.png](refactoring.png)

---

# Abstract Syntax Tree (AST)

![ast.png](ast.png)

---

# Why?

---

# Custom refactoring

* I always wanted to be able to write code to modify source code
* Extremely hard: working on a huge string without any sens, analysing it, moving stuff around, syntax, too many possibilities
* Frustrating, so many situation where I though "Ah! If I could do that!"
* Always bothering me
* Code generation too
* Only a bunch of people in the world actually do that
* Extremely hard: I'm at (x,y) in one file, what is the meaning of my surroundings?

---

# Ast.py

![ast.py.png](ast.py.png)

---

# Ast.py

Not lossless !

    ast_to_code(code_to_ast(code_source)) != source_code

(Comments, formatting)

(And ast\_to\_code doesn't exist officially).

---

# Ast.py

API Sax like

    !python
    class KeyAttributesFinder(ast.NodeVisitor):
        def visit_Assign(self, assign_node):
            # ...

        def visit_FunctionDef(self, function_node):
            # ...

        # visit_...

Terribly boring and unfunny to use (and unusable in a shell like IPython.)

---

# pythonfmt

Source code auto formater

---

# Code generation

Django (opendata projects like memopol):

    donnees.json -> models.py + import.py

---

# Refactoring: top to bottom

![refactoring1.png](refactoring1.png)

---

# Refactoring en python

* BycleRepairMan
* Rope (ast.py + regexs)
* PyCharm

---

# Conclusion: 2 problems

* We miss a good abstraction
* We miss a good API

---

# Solution 1: abstraction -> Baron

---

# Baron

* lossless ast!
* source == ast\_to\_code(code\_to\_ast(source))
* from an analysis problem to a graph modification problem
* output json for maximal interoperability (and data structure > objects in simplicity)

---

# Example

    !python

    from baron.helpers import show

    print show("1 + 2")

    [
        {
            "first_formatting": [
                {
                    "type": "space",
                    "value": " "
                }
            ],
            "value": "+",
            "second_formatting": [
                {
                    "type": "space",
                    "value": " "
                }
            ],
            "second": {
                "section": "number",
                "type": "int",
                "value": "2"
            },
            "type": "binary_operator",
            "first": {
                "section": "number",
                "type": "int",
                "value": "1"
            }
        }
    ]

---

![refactoring2.png](refactoring2.png)

---

# State of the project

*~1 year of work (needed to learn)*

* +1000 unit tests (TDD)
* works on the top 100 of pypi
* utilities: position\_to\_path, position\_to\_node, bounding\_box, walker etc...
* fully documented

---

# Solution 2: API -> RedBaron

---

# Plan

* principle
* exploration (query)
* graph modification
* lists modification

---

# RedBaron

* API on top of Baron
* Like BeautifulSoup/JQuery: mapping from a datastructure to objects
* For the human: user friendly as much as possible
* try to do all the boring low level stuff for you
* Designed to be used in IPython (but not only)

---

# RedBaron

Very simple API:

    !python
    from redbaron import RedBaron

    red = RedBaron("some source code as a string")
    # ...
    red.dumps()  # source code

---

# As intuitive as possible

Overloading of \_\_repr\_\_:

BeautifulSoup:

![soup.png](soup.png)

---

# As intuitive as possible

Overloading of \_\_repr\_\_:

BeautifulSoup:

![soup.png](soup.png)

RedBaron:

![repr.png](repr.png)

---

# Self descriptive

RedBaron:

![at_0.png](at_0.png)

---

# Self descriptive

RedBaron:

![at_0.png](at_0.png)

".help()"

![help.png](help.png)

---

# Exploration

Like BeautifulSoup:

    !python
    red = RedBaron("a = 42\ndef test_chocolate(): pass")
    red.find("name")
    red.find("int", value=42)
    red.find("def", name="g:test_*")
    red.find("def", name="re:test_*")
    red.find("assignment", lambda x: x.target.dumps() == "INSTALLED_APPS")

    red.find_all("name")
    red.find_all(("name", "int"))
    red.find_all("def", arguments=lambda x: len(x) == 3)
    red.find_all("def", recursive=False)

---

# Exploration

Like BeautifulSoup:

    !python
    red = RedBaron("a = 42\ndef test_chocolate(): pass")
    red.find("name")
    red.find("int", value=42)
    red.find("def", name="g:test_*")
    red.find("def", name="re:test_*")
    red.find("assignment", lambda x: x.target.dumps() == "INSTALLED_APPS")

    red.find_all("name")
    red.find_all(("name", "int"))
    red.find_all("def", arguments=lambda x: len(x) == 3)
    red.find_all("def", recursive=False)

Shortcuts (like BeautifulSoup):

    !python
    red = RedBaron("a = 42\ndef test_chocolate(): pass")
    red.name
    red.int
    red.else_

    red("name")
    red(("name", "int"))
    red("def", arguments=lambda x: len(x) == 3)

---

# Modification

How to modify a node?

    !python
    from redbaron import RedBaron, BinaryOperatorNode

    red = RedBaron("a = 'plop'")
    red[O].value  # 'plop'
    red[0].value = BinaryOperatorNode({'first_formatting':[{'type': 'space',
    'value': ' '}], 'value': '+', 'second_formatting': [{'type': 'space',
    'value': ' '}], 'second': {'section': 'number', 'type': 'int', 'value':
    '1'}, 'type': 'binary_operator', 'first': {'section': 'number', 'type':
    'int', 'value': '1'}})

Pretty much horrible.

---

# Magic with \_\_setattr\_\_

    !python
    from redbaron import RedBaron, BinaryOperatorNode

    red = RedBaron("a = 'plop'")
    red[0].value = "1 + 1"

    # works also with: redbaron nodes and json ast

Works for __every__ nodes.

---

# Advanced modifications:

Another problem: what is the body of the bar function?

    !python
    class Foo():
        def bar(self):
            pass

        def baz(self):
            pass

---

# Advanced modifications:

Another problem: what is the body of the bar function?

    !python
    class Foo():
        def bar(self):
            pass

        def baz(self):
            pass

Expected:

![expected.png](expected.png)

---

# Advanced modifications:

Another problem: what is the body of the bar function?

    !python
    class Foo():
        def bar(self):
            pass

        def baz(self):
            pass

Expected:

![expected.png](expected.png)

Reality:

![reality.png](reality.png)

---

# Solution: magic!

![magic.gif](magic.gif)

    !python

    red.find("def", name="bar").value = "pass"
    red.find("def", name="bar").value = "pass\n"
    red.find("def", name="bar").value = "    pass\n"
    red.find("def", name="bar").value = "    pass\n    "
    red.find("def", name="bar").value = "        pass\n        "
    red.find("def", name="bar").value = "\n    pass\n    "
    # etc ..

Works for every node with a body: *else*, *exceptions*, *finally*, *elif* etc...

---

# Lists

Problem: how many elements are in this list? <code>['a', 'b', 'c']</code>

---

# Lists


Problem: how many elements are in this list?

**Expected**:

![list_expected.png](list_expected.png)

---

# Lists


Problem: how many elements are in this list?

**Expected**:

![list_expected.png](list_expected.png)

**Reality**:

![list_reality.png](list_reality.png)

---

# Lists: solution

Solution: "proxy lists", gives you the same API than python list and handle formatting for you.

**Reality again**:

![list_expected.png](list_expected.png)

Works for:

* "," (with and without indentations)
* ".", for example: <code>a.b.c().pouet[stuff]</code>
* endl separated lines (function body, "blocks")

---

# Helpers

* <code>.map</code>
* <code>.apply</code>
* <code>.filter</code>
* <code>.next</code>, <code>.previous</code>, <code>.parent</code>
* <code>.replace</code>

---

# Some examples

    !python

    # rename a 'name' (warning: won't rename everything)
    for i in red('name', value='pouet'): i.value = 'plop'

---

# Some examples

    !python

    # install a django app
    red.find("assign", target=lambda x: x.dumps() == 'INSTALLED_APPS').\
        value.append("'debug_toolbar.apps.DebugToolbarConfig'")

---

# Some examples

    !python

    # lines_profiler
    red('def', recursive=False).\
        map(lambda x: x.decorators.insert(0, '@profile'))

---

# Some examples

    !python

    # lines_profiler
    red('def', recursive=False).\
        map(lambda x: x.decorators.insert(0, '@profile'))

    # remove them
    red("decorator", lambda x: x.dumps() == "@decorator").\
        map(lambda x: x.parent.parent.decorators.remove(x))

---

# Some examples

    !python

    # print a -> logger.debug(a)
    red('print', value=lambda x: len(x) == 1).\
        map(lambda x: x.replace('logger.debug(%s)' % x.value.dumps())

    # print a, b, c -> logger.debug("%s %s %s" % (a, b, c))
    red('print', value=lambda x: len(x) == 1).\
        map(lambda x: x.replace('logger.debug("%s" % (%s))' %
                                    (" ".join('%s' * len(x.value)))

---

![refactoring3.png](refactoring3.png)

---

# Project state

* +1200 tests
* fully documented (example for everything
* tutorial
* reference lib to use works with baron
* still in alpha, still some a bit annoying bugs
* should be working for 80% of the cases
* **No** static analysis (later with astroid/jedi?)

---

# Documentation

Every example are run at documentation compile time:

![documentation.png](documentation.png)

---

# Conclusion

---

![kent.png](kent.png)

---

# "Dude, you are coding the new 'ed' of the 21th century with 4 more level of abstractions!"<br>a friend, drunk

---

# Infos

RedBaron:

* [https://github.com/psycojoker/redbaron](https://github.com/psycojoker/redbaron)
* [https://baron.readthedocs.org](https://baron.readthedocs.org)
* <code>pip install redbaron</code>

Baron:

* [https://github.com/psycojoker/baron](https://github.com/psycojoker/baron)
* [https://redbaron.readthedocs.org](https://redbaron.readthedocs.org)
* <code>pip install baron</code>

Contacts:

* Me: cortex@worlddomination.be
* Irc: irc.freenode.net#baron

---

# Projets voisins: PyFmt

Usage:

    pyfmt file.py  # output to standard output
    pyfmt -i file.py  # replace the content of the file, like -i of sed

Depuis python:

    !python
    from pyfmt import format_code

    format_code(source_code)

---

# Projets voisins: RedFlyingBaron

    !bash
    red *.py  # dans un shell bash/zsh/autre

Qui lance un shell:

    !python
    red
    red[0]
    red["./test_redflyingbaron.py"]
    red["test_redflyingbaron.py"]
    red["test_redflyingbaron"]
    red[1:]

    red.display()

    red[0].save()
    red.save()
    red[0].reload()
    red.reload()

    red["f:redflyingbaron"]
    red[re.compile(r'[^_]+')]
    red["re:[^_]+"]
    red[lambda key, value: "red" in key]

    red.find("stuff")
    red.find_all("stuff")

    red.add("/path/to/file", "/path/to/another/file", "again.py")
