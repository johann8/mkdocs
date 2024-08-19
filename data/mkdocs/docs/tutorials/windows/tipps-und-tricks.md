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

??? tip "Windows 11 - Altes Kontextmenü wiederherstellen"

    #### 

    - Suchen -> regedit eingeben und `Registrierungs-Editor` starten
    - `HKEY_USERS` ausklappen und auf User `SID` mit rechter Maustaste klicken und `umbenennen` wählen
    - Mit `Strg` + `C` den Wert kopieren z.B. `S-1-5-21-2986536721-483922400-4177506015-1001`
    - Editor öffnen und den Wert einfügen
    - Die `Befehlszeile` `unten` kopieren und in die neue Zeile in Editor einfügen

    ```
    reg.exe add "HKEY_USERS\S-1-5-21-3764594489-3024899086-447399387-1004_Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve
    ```

    Die User `SID` in der `Befehlszeile` austauschen wie unten im Beispiel
    ```
    reg.exe add "HKEY_USERS\S-1-5-21-2986536721-483922400-4177506015-1001_Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve    
    ```

    - `CMD` als User starten und die angepasste `Befehlszeile` einfügen und mit `Enter-Taste` ausführen
    - Nach dem Reboot steht das alte Kontextmenü bereit

[^1]: [Wikipedia - Chocolatey](https://de.wikipedia.org/wiki/Chocolatey){target=\_blank}
