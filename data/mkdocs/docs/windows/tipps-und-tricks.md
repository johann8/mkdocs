---
Title: Windows Tipps und Tricks
Summary: Windows Tipps und Tricks
Authors: Johann Hahn
Date:    Juli 24, 2024
tags: [windows, tipps, tricks]
---

# :fontawesome-brands-windows: Windows Tipps und Tricks

### `Chocolatey` Repository installieren

[Chocolatey](https://chocolatey.org/){target=\_blank}[^1] is a machine-level, command-line package manager and installer for software on Microsoft Windows. It uses the NuGet packaging infrastructure and Windows PowerShell to simplify the process of downloading and installing software.

??? tip "`Chocolatey` Repository installieren"

    #### Chocolatey Repo installieren

    ```bash
    # CMD als Administrator starten und eingeben:
    @"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
    ```

    !!! tip "Terminal Programm `cmder` zu Indexierung einfügen."
        Starten Sie die Systemsteuerung und suchen Sie nach `Indizierungsoptionen`. Klicken Sie darauf und danach auf die Schaltfläche `Ändern`. Im Fenster navigieren Sie zum `C:\tools` und setzen Sie den Hacken ein. Danach klicken Sie auf `Schließen`. Nach etwas Zeit findet man `cmder` über Windows-Suche.    

    #### Windows Terminal `cmder` aus der Repository installieren

    ```bash
    # CMD als Administrator starten und eingeben:
    choco install cmder -y

    ```
    #### Softaware aus der Repository installieren

    ```bash
    # cmder als Administrator starten und eingeben:
    choco install 7zip cmder notepadplusplus firefox pdf24 googlechrome sudo adobereader -y

    ```
    #### Einige Beispiele

    ```bash
    # Alle installierte Software anzeigen lassen
    choco list
   
    # Software deinstallieren
    choco uninstall anydesk anydesk.portable -y

    # Software nur aus lokaler Repo entfernen, nicht deinstallieren 
    choco uninstall anydesk -n --skipautouninstaller -y

    # Suchen nach Software in der Repo
    choco search totalcomm

    # Update alle installierten Software
    choco upgrade all -y
    
    ```

??? tip "Den Fehler `Windows-RE-Image wurde nicht gefunden` beheben"

    #### Fehler `Windows-RE-Image wurde nicht gefunden` beheben

    ```
    # CMD | cmder als Administrator starten und führen Sie aus:
    
    # Suchen Sie die Datei wine.wim auf Ihrem Computer
    dir /a /s c:\winre.wim

    # Ändern Sie die Atrubute der Datei winre.wim
    attrib -h -s c:\$WinREAgent\Backup\winre.wim
    
    # Kopieren Sie die Datei winre.wim
    xcopy /h c:\$WinREAgent\Backup\winre.wim c:\Windows\System32\Recovery

    # Setzen Sie den neuen Pfad
    reagentc /setreimage /path C:\windows\system32\recovery

    # Aktivieren Sie die Wiederherstellungsumgebung
    reagentc /enable

    # überprüfen Sie den Status
    reagentc /info
    ```
[^1]: [Wikipedia - Chocolatey](https://de.wikipedia.org/wiki/Chocolatey){target=\_blank}
