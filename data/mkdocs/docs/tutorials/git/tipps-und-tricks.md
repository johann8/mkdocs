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

##### `Git` - Neue Repository erstellen

!!! tip ""

    :material-lightbulb-on: **TIPP**

    ```bash
    echo "# Tools" >> README.md
    git init
    git add README.md
    git diff --staged
    git commit -m "first commit"
    git branch -M master
    git push -u origin master
    git config credential.helper store
    ```

##### `Git` - Datei | Ordner zu Repository hinzufügen

!!! tip ""

    :material-lightbulb-on: **TIPP**

    ```bash
    git add README.md (folder/*)
    git diff --staged
    git commit -m "first commit"
    git push -u origin master

    # Mit dem folgenden Befehl werden alle Änderungen in der Dateien in `commit` übernommen
    git commit -s -a -m
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

##### `Git` - Datei | Ordner innerhalb Repository verschieben

!!! tip ""

    :material-lightbulb-on: **TIPP**

    ```bash
    git mv folder/README.md ./
    git mv data/folder1/folder2 data/
    git commit -m 'moved folder "linux"'
    ```

##### `Git` - Repo clonen, Änderungen herunterladen

!!! tip ""

    :material-lightbulb-on: **TIPP**

    ```bash
    git clone https://github.com/johann8/mkdocs.git
    git remote -v
    git fetch origin
    git pull origin
    git status
    ```

#####  `Git` - Ändern Remote Repository

!!! tip ""

    :material-lightbulb-on: **TIPP**

    ```bash
    git remote -v
    git remote set-url origin https://github.com/johann8/tools1.git
    git remote -v
    git push -u origin master
    ```
#####  `Git` - Lokales Repository mit Remote erneut abgleichen

!!! tip ""

    :material-lightbulb-on: **TIPP**

    ```bash
    # Wechseln zu Repo Ordner
    cd /opt/mkdocs

    # Alte Ordner löschen
    rm -rf data/mkdocs/.cache
    rm -rf data/mkdocs/*
    ls -la data/mkdocs/

    # Den Name von Branch anzeigen lassen
    git branch

    # Daten laden und speichern
    git fetch
    git reset --hard HEAD
    git merge origin/master
    ls -la data/mkdocs/
    ```

[^1]: :material-wikipedia: [Wikipedia - Git](https://de.wikipedia.org/wiki/Git){target=\_blank}
