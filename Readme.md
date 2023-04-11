Doc : https://www.techiediaries.com/python-unit-tests-github-actions/

# Création de test unitaires en python et lancement des test avec github action

## 1. Installer l'outil de test python

Je commence par créer un environnement pytohn, il existe plusieurs outils pour ça, en ce qui me concerne j'utilise `pipenv` :
```bash
$ pipenv shell
```
A noter : l'envvironnement python n'est pas forcément nécéssaire en production

J'installe maintenant `pytest` dans mon environnement python. `pytest` est l'outil qui nous permettra d'effectuer nos tests unitaires
```bash
$ pipenv install pytest
```
Si vous n'utiliser pas `pipenv`, la commande `pip install pytest` sera plus adéquat

## 2. Écriture du code à tester

Maintenant nous allons créer un fichier s'appelant 'test_capitalize.py', puis écrire le code suivant dedans :
```python
	def capitalize_string(s):
	if not isinstance(s, str):
		raise TypeError('Please provide a string')
	return s.capitalize()

	# Cette fonction sert de test
	# Elle test la fonction de dessus en comparant la valeur voulue et la valeur retournée 
	def test_capitalize_string():
		assert capitalize_string('test') == 'Test'
```

Pour lancer le test unitaire :
```bash
$ pytest
```

Voici à quoi ressemble le "output" lorsque les test c'est bien déroulés:
```bash
====================================================================== test session starts ================================================================================================
platform linux -- Python 3.10.6, pytest-7.3.0, pluggy-1.0.0
rootdir: /home/marcus/Documents/exo/ci_exercise/ci_github
collected 1 item                                                                                                                                                                                                    

test_capitalize.py .                                                                                                                                                                  [100%]

======================================================================== 1 passed in 0.00s =================================================================================================

``` 
Voici à quoi ressemble le "output" lorsque les test ne c'est pas bien déroulés:
```bash
======================================================================== test session starts ================================================================================================
platform linux -- Python 3.10.6, pytest-7.3.0, pluggy-1.0.0
rootdir: /home/marcus/Documents/exo/ci_exercise/ci_github
collected 1 item                                                                                                                                                                                                    

test_capitalize.py F                                                                                                                                                                   [100%]

============================================================================ FAILURES ======================================================================================================
_____________________________________________________________________ test_capitalize_string _______________________________________________________________________________________________

    def test_capitalize_string():
>       assert capitalize_string('test') == 'test'
E       AssertionError: assert 'Test' == 'test'
E         - test
E         ? ^
E         + Test
E         ? ^

test_capitalize.py:8: AssertionError
===================================================================== short test summary info ==============================================================================================
FAILED test_capitalize.py::test_capitalize_string - AssertionError: assert 'Test' == 'test'
======================================================================== 1 failed in 0.01s =================================================================================================

``` 
Maintenant que les tests unitaire sont mis en place, il faut créer un `requirement.txt`, pour pouvoir installer toutes les dépendances nécéssaires au projet

```bash
$ pip freeze > requirements.txt
```

## 3. Configuration du worflow github
Pour créer un workflow, il faut créer un fichier YAML dans `github/workflows/ci.yml`.

Ouvrez le fichier `ci.yml`, puis ajoutez y le code suivant :
```yaml
name: Run Python Tests
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install Python 3
        uses: actions/setup-python@v1
        with:
          python-version: 3.7
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      - name: Run tests with pytest
        run: pytest 
```

Ici le workflow est nommé `Run Python Tests`, il démarre lorqu'on effectue un push ou un pull sur la branch `main`. Il contient un seul job nommé `build`, celui-ci est composé de quatres étapes qui seront excutées dans un container Ubuntu.
La première étape consiste à donner l'accès au workflow, afin qu'il puisse accéder au code de notre dépot git.
Nous ajoutons ensuite la deuxième étape qui consiste tout simplement à installer `python` (dans notre cas la version `3.7`)
Dans la troisième étape nous installons les dépendances indiquées dans le fichier `requirements.txt`.
Puis finalement, il ne reste plus qu'à lancer nos tests unitaires.

## 4. Lancement du workflow
Pour lancer le worflow, il suffit d'effectuer un commit et un push
```bash 
$ git add -A
$ git commit -m "First commit"
$ git push origin master 
``` 