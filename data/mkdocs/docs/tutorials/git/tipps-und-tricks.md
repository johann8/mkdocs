---
Title: Git Tipps und Tricks
Summary: Git Tipps und Tricks
Authors: Johann Hahn
Date:    August 07, 2024
tags: [git, repo]
---

# :material-github: `Git` Tipps und Tricks

!!! question "What is Git?"

    :material-git: [`Git`][Git][^1] ist eine freie Software zur verteilten Versionsverwaltung von Dateien.

    [Git]: https://github.com/

##### `Git` - Datei | Ordner zu Repository hinzufügen


!!! tip ""

    :material-lightbulb-on: **TIPP**

    ```bash
    git add README.md (folder/*)
    git diff --staged
    git commit -m "first commit"
    git push -u origin master
    ```

##### `Git` - Datei | Ordner von  Repository entfernen

!!! tip ""

    :material-lightbulb-on: **TIPP**

    ```bash
    git rm filename
    git rm -r  data/folder
    git commit -m "Deleted the file from the git repository"
    git push -u origin master
    ```

##### Git - Datei | Ordner innerhalb Repository verschieben

!!! tip "**Tipp** &nbsp; &nbsp; :material-lightbulb-on:"

    ```bash
    git mv folder/README.md ./
    git mv data/folder1/folder2 data/
    git commit -m 'moved folder "linux"'
    ```

##### Git - Repo clonen, Änderungen herunterladen

??? tip "&nbsp; &nbsp;**Tipp:** :material-lightbulb-on:"

    ```bash
    git clone https://github.com/johann8/mkdocs.git
    git remote -v
    git fetch origin
    git pull origin
    git status
    ```

#### Git - 

??? tip ""

    :material-lightbulb-on: **Tipp:** Git -

    ```bash

    ```

#### Git - 

??? tip ""

    :material-lightbulb-on: **Tipp:** Git -

    ```bash

    ```

[^1]: :material-wikipedia: [Wikipedia - Git](https://de.wikipedia.org/wiki/Git){target=\_blank}
