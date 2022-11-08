# TP CI/CD

# Prérequis 

Pour faire ce TP il vous faut un compte sur https://gitlab.com, si ce n'est pas le cas allez en créer un c'est gratuit.
Une fois sur Gitlab il vous faudra créer un projet. Pour cela il suffit de cliquer sur "New projet" en haut à droite de la page d'accueil une fois connecté. 
Vous allez ensuite créer un projet vide "Create blank project". 
Pour pouvoir utiliser la partie CI/CD de Gitlab vous allez devoir register votre propre runner.
Pour cela vous devez posséder Docker et vous allez exécuter la commande suivante : 

```bash
docker run -d --name gitlab-runner --restart always -v /root/runner-config:/etc/gitlab-runner -v /var/run/docker.sock:/var/run/docker.sock gitlab/gitlab-runner:latest
```
"/root/runner-config" étant la destionnation de votre choix, vous pouvez donc la remplacer par ce que vous voulez. 
Elle va avoir pour but d'exécuter un runner gitlab dans un container Docker sur votre machine. 
Ensuite vous allez devoir register le runner gitlab dans votre projet gitlab, pour ca vous devez récuper un token que vous trouverai sur le projet gitlab que vous venez de créer. 
Allez dans votre projet "Settings" > "CI/CD", dans la section "Runners", cliquez sur "Expand". Dans la partie "Specific runners" récupérer votre token, et dans la partie "Shared runners", désactiver l'option "Enable shared runners for this project".
Cela permettra à votre projet d'utiliser uniquement les runners que vous mettez à disposition et non ceux de gitlab.
Vous allez ensuite vous connecter en ligne de commande dans le container runner-gitlab pour le register sur votre projet. 
Pour cela vous allez utiliser la commande suivante : 

```bash
docker exec -it gitlab-runner bash
```

Et maintenant vous allez rentrer la ligne de commande suivante : 

```bash
gitlab-runner register --url https://gitlab.com/ --registration-token TOKEN
```

"TOKEN" sera remplacé par le token qui vous avez récupérer dans gitlab dans l'étape d'avant. 
Maintenant que vous avez, un compte, un projet et un runner, nous pouvons commencer le TP. 


## Exercice 1 

Pour commencer vous allez ajouter un fichier .gitlab-ci.yml à la racine de votre projet, avec le contenue suivant : 

```yml
stages:
  - build

build-job:
  stage: build
  script:
    - echo "hello world"
```

Une fois le ficher push dans votre projet, le pipeline de CI/CD va s'exécuter, pour voir les détails de l'xécution, vous pouvez aller voir dans "CI/CD" > "Pipeline", vous pourrez ainsi voir les différents pipelines du projet. 
Pour consulter un pipeline il suffit de cliquer sur son status qui peut être "running", "passed", "cancelled" ou "failed". 

![CI PAGE](images/page-ci.png)

Pour avoir plus de détail sur ce qu'il s'est passé pendant un job, il vous suffit de cliquer dessus, ce qui va vous donner toutes les informations, depuis le runner sur lequel il est exécuté, aux résultats des commandes effectuées dans le script. 

![CI JOB](images/job-ci.png)

![CI EXECUTION](images/ci-running.png)

Quelle est l'image utilisé par la CI ? Quel est le nom du runner sur lequel le job est effectué ? 

Comme nous allons utiliser la CI pour faire du python il nous faut vérifier la version de python (avec la commandez : python --version)

Rajoutez une étape dans la CI pour nous permettre d'afficher le numéro de version du python. 

Pourquoi est-ce que la commande ne marche-t-elle pas ? 

Faites en sorte de changer l'image utilisée par le job par une image python et retestez.

## Exercice 2  

Pour mettre en place de la CI/CD sur un projet il vous un projet avec des tests unitaires, vous pouvez utiliser celui que vous voulez, mais si vous n'en avez pas nous allons utiliser un projet python qui a pour but de nous retouner si un nombre est premier ou pas. 

Vous allez créer un fichier nommé _project.py_ avec le contenu suivant :

```py
import math


def is_prime(n):
    if n < 0: 
        return 'Negative numbers are not allowed'

    if n <= 1:
        return False

    if n == 2:
        return True 

    if n % 2 == 0:
        return False

    for i in range(2, int(math.sqrt(n)) + 1):
        if n % i == 0:
            return False 
    return True


def cubic(a): 
    return a * a * a


def say_hello(name):
    return "Hello, " + name

say_hello("world !")
```

Et nous allons créer un fichier de test unitaire qui aura pour but de tester mes fonction de notre application.

Vous allez créer un fichier nommé _test.py_ avec le contenu suivant :

```py
import unittest

import project


class TestProject(unittest.TestCase):
    def test_is_prime(self):
        self.assertFalse(project.is_prime(5))
        self.assertTrue(project.is_prime(2))
        self.assertTrue(project.is_prime(3))
        self.assertFalse(project.is_prime(8))
        self.assertFalse(project.is_prime(10))
        self.assertTrue(project.is_prime(7))
        self.assertEqual(project.is_prime(-3),"Negative numbers are not allowed")


if __name__ == '__main__':
    unittest.main()

```

A ce moment du TP vous devez avoir dans votre répo git (Si vous avez choisi de ne pas prendre de projet personnel ou professionnel), 3 fichiers: 

- .gitlab-ci.yml
- project.py
- test.py

Maintenant que nous avons notre projet nous pouvons passer à la phase suivante, l'exécution du build et des tests dans la CI.

Pour cela nous allons modifier le fichier _.gitlab-ci.yml_ : 

```yml
stages:
  - build
  - test

build-job:
  stage: build
  image: python:alpine3.16
  script:
    - #Exécution du projet

test-job: 
  stage: test
  image: python:alpine3.16
  script:
    - #Exécution des tests

```

Implémentez les étapes pour que la CI fonctionne. 

Qu'est ce que vous constatez au lancement des tests unitaires ? 

Trouvez la raison de cela, puis une solution pour que la CI s'exécute correctement. 

## Exercice 3

Une fois votre CI réparée, nous allons rajouter une étape importante : __le lint__, aussi appelé qualité du code. 

La qualité du code à plusieurs objectifs, le premier est comme son nom l'indique d'assurer une qualité dans le code en terme de synthaxe, mais aussi de faire appliquer ce l'on appelle une "convention de style". Ces conventions ont pour but de codifier la façon dont les développeurs développent, avec par exemple la normalisation du nommage des fonctions, des variables, des globales, etc...

Tout cela ayant pour but de rendre le code le plus facilement lisible et compréhensible pour n'importe quelle personne qui devrait le lire. 

Chaque language possède sa ou ses conventions (elles peuvent être différentes en fonction du framework utilisé), PSR pour le PHP ou PEP8 pour python. 

Maintenant que nous savons mieux ce que le lint nous allons rajouter un job dans notre CI qui va effectuer cette étape de lint. 

Le lint peut être effectué en parallèle des tests unitaires, nous allons donc l'intégrer dans le stage de "test"

```yml
stages:
  - build
  - test

build-job:
  stage: build
  image: python:alpine3.16
  script:
    - #Exécution du projet

test-job: 
  stage: test
  image: python:alpine3.16
  script:
    - #Exécution des tests

lint-job: 
  stage: test
  image: python:alpine3.16
  script:
    - #Exécution du lint
```

Implémentez l'étape de lint. 

Pour le lint python nous allons utiliser un utilitaire appelé _pylint_. 

L'utiliraire pour installer les package en python est "pip", donc pour installer un package il faut utiliser la commande "pip install <nom_du_package>".

Que constatez vous lors du lancement de la CI ? Pourquoi ? Comment faire pour régler ça ? 

Quel est le problème avec le fait d'installer in package dans une CI ? Comment pourriez vous résoudre ce problème ? 

## Exercice 4

Nous allons maintenant utiliser la fonction d'artifact que nous offre Gitlab. Pour mettre cela en oeuvre nous allons améliorer l'étape précédente. 

Pour cela nous allons générer un rapport avec la liste des erreur détecter par _pylint_. 

Pylint possède une option pour créer un fichier de sortie avec l'option _--output <nom-du-fichier>_

```yml
stages:
  - build
  - test

build-job:
  stage: build
  image: python:alpine3.16
  script:
    - #Exécution du projet

test-job: 
  stage: test
  image: python:alpine3.16
  script:
    - #Exécution des tests

lint-job: 
  stage: test
  image: python:alpine3.16
  script:
    - #Exécution du lint
  artifacts:
    paths:
    - /path/to/the/output
```

Comment pouvez vous récupérer l'artéfact ? Combien de temps reste t-il disponible ? 

## Exercice 5

Nous allons maintenant voir comment passer un artéfact d'un job à l'autre, ce qui va nous être utile dans beaucoup de language compilé comme le dotnet, car une fois build, nous allons passer le résultat du build à chaque job au lieu de le rebuild à chaque étape de la CI. 

Comme nous utilisons du python nous allons passer le seul artéfact que nous avons créé actuellement, à savoir le rapport généré par l'étape de lint. 

Nous allons donc créer une étape qui va nous permettre d'afficher le contenu du rapport du qualité.

Par défaut, les artéfacts sont téléchargés par chaque job.

```yml
stages:
  - build
  - test

build-job:
  stage: build
  image: python:alpine3.16
  script:
    - #Exécution du projet

test-job: 
  stage: test
  image: python:alpine3.16
  script:
    - #Exécution des tests

lint-job: 
  stage: test
  image: python:alpine3.16
  script:
    - #Exécution du lint
  artifacts:
    paths:
    - /path/to/the/output

echo-lint-job:
  stage: test
  script:
    - cat <nom-artéfact>

```

Que constatez vous après l'exécution de la CI ? Quelle commande commande pouvez vous effectuer pour vérifier si le fichier existe ? 

En utilisant la documentation de Gitlab, proposez une solution pour régler ce problème et appliquez le sur votre CI.





Petit rappel, avant d'installer un paquet, ne pas oublier de faire un update. 

De plus, le gestionnaire de paquet de la distribution alpine est "apk", donc pour installer un package il faut utiliser la commande "apk add <nom_du_package>".