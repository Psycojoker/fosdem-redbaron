# RedBaron<br>une approche bottom-up au refactoring en Python

---

# Moi

* Belgique
* Code beaucoup trop (python (beaucoup), django, oueb, haskell, ...)
* Tourisme des langages de programmation
* Neutrinet (FFDN, Gitoyen)
* La Quadrature du Net/Nurpa
* UrLab (hackerspace à l'ULB/Bruxelles)/HSBxl (hackerspace à Bruxelles)

En beaucoup trop détailler:
[http://worlddomination.be/about/about.html](http://worlddomination.be/about/about.html)

---

# Avant de commencer: revisions

---

# Refactoring (eclipse)

![refactoring.png](refactoring.png)

---

# Abstract Syntaxe Tree (AST)

![ast.png](ast.png)

---

# Plan

* Pourquoi ?
* Solution au premier problème (baron)
* Solution au deuxième problème (RedBaron)
* Conclusion

---

# Pourquoi ?

---

# Refactoring custom

* J'ai toujours voulu écrire du code pour modifier mon code
* Très difficile: string énorme sans sens, analyse, déplacement, trop de possibilités, syntaxe
* Frustrant, plein de cas où "ah, si seulement je pouvais scripter cette modification !"
* Comme une blessure à la lèvre
* Générer du code aussi

---

# Ast.py

![ast.py.png](ast.py.png)

---

# Ast.py

Pas lossless !

    ast_to_code(code_to_ast(code_source)) != source_code

(Commentaires, formatting)

(Et ast\_to\_code n'existe même pas de manière standard).

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

Super chiant, impossible à utiliser dans IPython efficacement.

---

# Generation de code

Django (memopol et co):

    stuff.json -> models.py import.py

Autre project:

    Générer du boiler plate en lisant des models de db

---

# pythonfmt

Auto formater du code python

---

# BeurpMiner

![beurpminer.png](beurpminer.png)

---

# Refactoring en python

![refactoring.png](refactoring.png)

* BycleRepairMan
* Rope (ast.py + regexs)
* PyCharm (?)
* Hyper dure: je suis en (x,y) dans un fichier, y a quoi autour de moi ?

---

# Refactoring: top to bottom

![refactoring1.png](refactoring1.png)

---

# Conclusion: 2 problèmes

* Il manque la bonne abstraction
* Il manque la bonne interface

---

# Solution 1: l'abstraction -> Baron

---

# Baron

* ast lossless ! (FST == Full Syntaxe Tree)
* source == ast\_to\_code(code\_to\_ast(source))
* transforme un problème d'analyse de code en parcours/modification d'un graph
* output du json pour compatibilité maximum (+ structure de donnée simple)

---

# Exemple

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

# État du projet

*1 an de boulot (j'ai du apprendre)*

* +1000 tests (TDD)
* marche sur le top 100 de pypi
* utilities: position\_to\_path, position\_to\_node, boundinx\_box, walker etc...
* (encore quelques bugs ultra rare)
* entièrement documenté

---

# Parenthèse: pyfmt

---

# Solution 2: l'interface -> RedBaron

---

# RedBaron

* Api au dessus du FST de Baron
* Comme BeautifulSoup/Jquery: mapping structure de donnée -> objects
* Pour l'humain, ~user friendly~ (pour certaines définition de user)
* Pensé, entre autre, pour être utilisé dans IPython (ou bpython)

---

# Intuitif (autant que possible)

.help()

shell __repr__

---

# Exploration

.find

.find_all

Raccourcies:

* .query
* (query)

---

# Modification

"Normalement on devrait faire ça" -> pourri

Magie de __setattr__ -> strings

-> pas besoin de ré-apprendre un nouveau truc, vous savez déjà coder du python

![refactoring3.png](refactoring3.png)

---

# Modifications avancés:

funcdef.value.dumps() -> "chiant"

Solution: magie (exemples)

Pareil pour les: .else, .exceptions, .finally etc ...

---

# Listes de choses

Problème: [1, 2, 3] -> 5 éléments en fait car 2 ","

Solution: des "proxy" de listes qui donnent la même API que les listes python et gèrent le formatting pour vous

Marche pour les:
* "," (avec et sans indentation)
* les ".", eg: a.b.c()
* les lignes séparés par des retours à la ligne

---

# Démo ?

---

# Etat

* +1200 tests
* entièrement documenté (plein d'exemples)

---

# Conclusion

---

![kent.png](kent.png)

---

# « Mec, t'es en train de coder le nouvel 'ed' du 21 ème siècle avec 4 niveaux d'abstractions en plus »<br>Hastake - Fin bourré

---

# Infos

* [https://github.com/psycojoker/baron](https://github.com/psycojoker/baron)
* [https://github.com/psycojoker/redbaron](https://github.com/psycojoker/redbaron)
* [https://baron.readthedocs.org](https://baron.readthedocs.org)
* [https://redbaron.readthedocs.org](https://redbaron.readthedocs.org)
* Moi: cortex@worlddomination.be
* Irc: irc.freenode.net#baron
