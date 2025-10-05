# Einführung

*Infrastruture as Code* in *DevOps* Projekten mit **{{ target_audience | default('Computacenter') }}**.

<figure markdown>
  ![CC Logo](assets/images/computacenter.png#only-light){ width="300" }
  ![CC Logo](assets/images/computacenter-white.png#only-dark){ width="300" }
  <figcaption></figcaption>
</figure>

## Ziel

Als kleines Projekt sollst du *automatisiert* die Konfiguration der *Ansible Automation Platform* (kurz *AAP*) anpassen. Die AAP bietet unter anderem ein Web-UI für die einfache Ausführung von Ansible Automatisierung, Hochverfügbarkeit, Authentifizierung, Auditing, Logging und vieles mehr.  
Die AAP selbst kann aber auch über eine API automatisiert werden, der Code dafür ist als sogenanntes *Ansible Playbook* bereits erstellt, **muss aber angepasst werden**.  

!!! success
    **Automate the automation (platform)!**

## Tipps

Du wirst in den folgenden drei Teilen den typischen Entwickler-Workflow durchlaufen und mit vielen Tools und Prozessen in Berührung kommen, welche dir tagtäglich **in einem *DevOps*-Projekt** begegnen werden.

!!! info
    Die folgenden Seiten bieten **Tabs** zur Aufgabenbeschreibung in **kurz und knapp** oder mit **ausführlicher Erklärung**, du kannst zwischen den beiden Tabs wechseln.

    === "Kurz und knapp"

        Hier findest du nur das nötigste.

        ```yaml
        tip: Use the small copy button on the right!
        ```

    === "Ausführliche Erklärung"

        In diesem Tab wird alles etwas ausführlicher beschrieben, mit zusätzlichen Informationen zu den verwendeten Kommandos oder Hintergrund-Infos zu der zugrunde liegenden Technologie.  

        Der Inhalt aller Code-Blöcke kann über einen kleinen Copy-Button rechts im Code-Feld kopiert werden.

        !!! tip
            Mit der **rechten Maustauste** kann der kopierte Content im Terminal eingefügt werden.

## Start

!!! quote ""
    **:rocket: - Los gehts mit der [Vorbereitung der Demo-Umgebung!](part1.md)**
