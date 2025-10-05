# *Development*

Die Konfiguration der Automation Platform wird über ein Ansible Playbook angepasst, du kannst das Playbook mit dem folgenden Kommando im Terminal/auf der Kommandozeile ausführen:

```console
ansible-navigator run playbook_controller_automation.yml
```

Bei den ersten Ausführungen wirst du mit einigen Fehlermeldung konfrontiert, es fehlen Dependencies, Credentials und auch ein Teil der Automatisierung an sich.

## 1 - Credentials exportieren

=== "Kurz und knapp"

    *Exportiere* (definiere) die folgenden **drei** Variablen:

    * `CONTROLLER_HOST`
    * `CONTROLLER_USERNAME`
    * `CONTROLLER_PASSWORD`

    **Exportiere alle drei Variablen**, für `CONTROLLER_USERNAME` beispielsweise die folgende Zeile **kopieren** und im *Terminal* **einfügen**:

    ```console
    export CONTROLLER_USERNAME=admin
    ```

    Nutze die **[Werte aus der Workshop-Übersichtsseite]({{ workshop_url | default('https://timgrt.github.io/IaC-Demo/') }}){ target=_blank }** aus **Abschnitt 3**.

=== "Ausführliche Erklärung"

    Um die AAP automatisieren zu können, muss sich das Playbook bei der Ausführung an der API anmelden, die notwendigen Credentials werden als *Umgebungsvariablen* übergeben. Die folgenden drei Variablen müssen definiert werden:

    * `CONTROLLER_HOST`
    * `CONTROLLER_USERNAME`
    * `CONTROLLER_PASSWORD`

    Die Umgebungsvariablen werden auf der Kommandozeile mit dem `export` Kommando übergeben, *Key* und *Value* werden durch ein Gleichzeichen (`=`) getrennt. Für `CONTROLLER_USERNAME` beispielsweise die folgende Zeile **kopieren** und im *Terminal* **einfügen**:


    ```console
    export CONTROLLER_USERNAME=admin
    ```

    **Exportiere alle drei Variablen.**  

    Nutze die **[Werte aus der Workshop-Übersichtsseite]({{ workshop_url | default('https://timgrt.github.io/IaC-Demo/') }}){ target=_blank }** aus **Abschnitt 3**.  

    Für die Variable `CONTROLLER_HOST` die URL des Red Hat Ansible Automation Controller, für die Variable `CONTROLLER_PASSWORD` das entsprechende Passwort.

??? question "Hilfe nötig? Klick mich..."

    Kopiere die folgenden Zeile und **ergänze sie mit den korrekten [Werten aus der Workshop-Übersichtsseite]({{ workshop_url | default('https://timgrt.github.io/IaC-Demo/') }}){ target=_blank }** aus **Abschnitt 3**:

    ```console title="Red Hat Ansible Automation Platform URL"
    export CONTROLLER_HOST=
    ```

    ```console title="Red Hat Ansible Automation Platform User name"
    export CONTROLLER_USERNAME=admin
    ```

    ```console title="Red Hat Ansible Automation Platform Password"
    export CONTROLLER_PASSWORD=
    ```

Mit dem folgenden Kommando kannst du prüfen ob die Variablen erfolgreich exportiert wurden:

```console
env | grep CONTROLLER
```

!!! success "Alles erledigt?"

    * [x] AAP-API Credentials als Umgebungsvariablen exportiert
        * [X] `CONTROLLER_HOST`
        * [X] `CONTROLLER_USERNAME`
        * [X] `CONTROLLER_PASSWORD`

## 2 - Credentials im (Ausführungs-) Container bekanntmachen

Du kannst jetzt das Playook mit dem folgenden Kommando ausführen:

```console
ansible-navigator run playbook_controller_automation.yml
```

!!! failure "*Erwartete (!)* Fehlermeldung"

    ```console
    TASK [Ensure required connection variables are defined] ************************
    fatal: [localhost]: FAILED! => {"assertion": "lookup('env', 'CONTROLLER_HOST') | length > 0", "changed": false, "evaluated_to": false, "msg": "Variables for accessing Automation controller are missing! Export the variables."}
    ```

    Es fehlen Variablen für den Login an der Ansible Automation Platform.

    **Aber die Variablen habe ich doch im vorherigen Schritt definiert/exportiert?**

=== "Kurz und knapp"

    Du musst die bestehende `ansible-navigator.yml` Konfigurationsdatei anpassen.

=== "Ausführliche Erklärung"

    Das `ansible-navigator` Binary startet bei jedem Aufruf einen (Podman-)Container, dieser Container ist eine isolierte Ausführungsumgebung (fast wie eine eigene, kleine Linux-Instanz), in dieser sind die von dir zuvor exportierten Umgebungsvariablen **nicht** bekannt.  

    Du musst die Umgebungsvariablen an den Container *übergeben* (*pass into the container*), dafür musst du die bestehende `ansible-navigator.yml` Konfigurationsdatei anpassen.  
    Diese Datei legt einige Parameter für den *Ansible Navigator* fest (z.B. das verwendete Container Image, wo Log-Dateien gespeichert werden sollen,

Schau in der [Ansible Navigator Dokumentation](https://ansible.readthedocs.io/projects/navigator/settings/#pass-environment-variable){ target=_blank } nach, an welcher Stelle die folgende Konfiguration **hinzugefügt** werden muss:

```yaml
   environment-variables:
     pass:
        - CONTROLLER_HOST
        - CONTROLLER_USERNAME
        - CONTROLLER_PASSWORD
```

Kopiere den obigen Block und füge ihn an der **richtigen** Stelle in der `ansible-navigator.yml` hinzu.

??? question "Hilfe nötig? Klick mich..."

    Die Konfigurationsdatei muss folgendermaßen aussehen, du kannst sie mit einem kleinen *Copy*-Button im Textfeld kopieren:

    ```yaml title="ansible-navigator.yml"
    ---
    ansible-navigator:
      execution-environment:
        image: registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:latest
        enabled: true
        container-engine: podman
        pull:
          policy: missing
        volume-mounts:
          - src: "/etc/ansible/"
            dest: "/etc/ansible/"
        environment-variables:
          pass:
            - CONTROLLER_HOST
            - CONTROLLER_USERNAME
            - CONTROLLER_PASSWORD
      logging:
        level: warning
        file: logs/ansible-navigator.log
      mode: stdout
      playbook-artifact:
        enable: true
        save-as: "logs/{playbook_status}-{playbook_name}-{time_stamp}.json"

    ```

Führe das Playbook mit dem folgenden Kommando erneut aus:

```console
ansible-navigator run playbook_controller_automation.yml
```

Zumindest der Task `Ensure all hosts from web group are enabled` sollte jetzt erfolgreich durchlaufen (die Authentifizierung ist erfolgreich).

!!! success "Alles erledigt?"

    * [X] Die `ansible-navigator.yml` Konfiguration ist angepasst
    * [x] Die drei Umgebgungsvariablen (`CONTROLLER_HOST`, `CONTROLLER_USERNAME`, `CONTROLLER_PASSWORD`) werden in den Container übergeben und die Authentifizierung ist erfolgreich.

## 3 - Automatisierung anpassen/vervollständigen

Zumindest der *Tasks* zur Überprüfung der Variablen und ein weiterer läuft bereits durch, trotzdem schlägt die Ausführung noch fehl.  

=== "Kurz und knapp"

    Die Automatisierung ist unvollständig, du musst sogenannte *Project*-Objekte erstellen.

=== "Ausführliche Erklärung"

    Die Automatisierung ist unvollständig, wenn du das Playbook erneut ausführst, wird versucht ein Job-Template zu erstellen, welches auf ein nicht-existentes *Project* verweist.

!!! failure

    ```console
    Request to /api/v2/projects/?name=Git+Repository+with+Ops+Playbooks returned 0 items, expected 1
    ```

=== "Kurz und knapp"

=== "Ausführliche Erklärung"

    Ein *Project* in der AAP zeigt auf ein *Git-Repository*, es enthält allen *Code* für die Automatisierung. Ein *Job Template* legt die Parameter zur Ausführung dieses *Codes* fest, es muss daher wissen, aus welchem *Project* der Code kommen soll.  
    Die AAP zieht alles zur Ausführung aus dem *Git Repository* und sorgt (durch die Auswahl einer Option) dafür, dass vor jeder Ausführung der Automatisierung, der aktuellste Stand geladen wird.

Ein Ansible Playbook besteht aus einer Liste an *Tasks*, **du musst die fehlenden Tasks hinzufügen!**  

=== "Kurz und knapp"

    Klicke links im *File Explorer* auf die Datei `playbook_controller_automation.yml`.

=== "Ausführliche Erklärung"

    Klicke links im *File Explorer* auf die Datei `playbook_controller_automation.yml`, ab Zeile 24 sind die einzelnen *Tasks* (Schritte/Aufgaben) beschrieben, um den AAP Controller in den gewünschten Zielzustand zu versetzen.

    !!! example ""

        {% raw %}
        ```yaml title="playbook_controller_automation.yml"
        ---
        - name: Prepare Automation Controller # (1)!
          hosts: localhost # (2)!
          connection: local
          module_defaults: # (3)!
            group/ansible.controller.controller:
              controller_host: "{{ lookup('env', 'CONTROLLER_HOST') }}"
              controller_username: "{{ lookup('env', 'CONTROLLER_USERNAME') }}"
              controller_password: "{{ lookup('env', 'CONTROLLER_PASSWORD') }}"
          pre_tasks: # (4)!
            - name: Ensure required connection variables are defined
              ansible.builtin.assert:
                that:
                  - lookup('env', 'CONTROLLER_HOST') is defined
                  - lookup('env', 'CONTROLLER_HOST') | length > 0
                  - lookup('env', 'CONTROLLER_USERNAME') is defined
                  - lookup('env', 'CONTROLLER_USERNAME') | length > 0
                  - lookup('env', 'CONTROLLER_PASSWORD') is defined
                  - lookup('env', 'CONTROLLER_PASSWORD') | length > 0
                quiet: true
                fail_msg: "Variables for accessing Automation controller are missing! Export the variables."
          tasks: # (5)!
            - name: Ensure all hosts from web group are enabled
              ansible.controller.host:
                name: "{{ item }}"
                inventory: Workshop Inventory
                enabled: true
                validate_certs: true
              loop:
                - node1
                - node2
                - node3
        # ... most tasks are cut for readability...
        ```
        {% endraw %}

        1. Der Name des Playbooks, wird auf der CLI bei der Ausführung ausgegeben.
        2. Das Ziel der Automatisierung, standartmäßig kommuniziert Ansible per SSH. Da wir eine API ansprechen wollen, können wir gegen `localhost` mit der Verbindungsmethode `local` ausführen.
        3. Einige Standart-Parameter welche **alle** Module der `awx.awx` Collection bekommen sollen. In diesem Fall sind es die Authentifizierungs-Parameter, in jedem Task wird ein neuer API-Call zur AAP gemacht.
        4. Eine Liste an Tasks (hier nur ein einzelner), welche definitiv vor allen anderen laufen sollen. In diesem Fall wird eine Prüfung der Umgebungsvariablen gemacht, sollten sie fehlen, wird eine *sprechende* Fehlermeldung ausgeben.
        5. **Ab hier startet die eigentliche Automatisierung der AAP!** In dieser Liste (im YAML-Format beginnt jedes Listenelement mit `-`) müssen alle Tasks in der richtigen Reihenfolge aufgeführt werden.

Suche in der [Ansible Dokumentation](https://docs.ansible.com/ansible/latest/collections/awx/awx/index.html#modules){ target=_blank } nach dem passenden *Modul* zur Erstellung eines *Projects*.  

!!! tip
    Nutze die *Examples* (Beispiele) auf der Dokumentations-Seite des Moduls, du kannst ein Beispiel kopieren und einfügen, anschließend passt du die Parameter an und fügst fehlende Parameter hinzu.  
    Der *Modul*-Name **muss** `ansible.controller.` statt mit `awx.awx.` beginnen!

Du musst sowohl das Project mit dem Operations-Content, als auch das Project mit dem Development-Content hinzufügen (**insgesamt also zwei Tasks**).

!!! example "Task-(Name): `Add project with playbooks for Ops workloads`"
    | Key                    | Value                                              |
    | ---------------------- | -------------------------------------------------- |
    | `name`                 | `Git Repository with Ops Playbooks`                |
    | `default_environment`  | `Default execution environment`                    |
    | `scm_type`             | `git`                                              |
    | `scm_url`              | `https://github.com/ansible/workshop-examples.git` |
    | `scm_branch`           | `webops`                                           |
    | `scm_update_on_launch` | `true`                                             |
    | `scm_delete_on_update` | `true`                                             |
    | `scm_clean`            | `true`                                             |
    | `state`                | `present`                                          |

!!! example "Task-(Name): `Add project with playbooks for Dev workloads`"
    | Key                    | Value                                              |
    | ---------------------- | -------------------------------------------------- |
    | `name`                 | `Git Repository with Dev Playbooks`                |
    | `default_environment`  | `Default execution environment`                    |
    | `scm_type`             | `git`                                              |
    | `scm_url`              | `https://github.com/ansible/workshop-examples.git` |
    | `scm_branch`           | `webdev`                                           |
    | `scm_update_on_launch` | `true`                                             |
    | `scm_delete_on_update` | `true`                                             |
    | `scm_clean`            | `true`                                             |
    | `state`                | `present`                                          |

!!! warning

    === "Kurz und knapp"

        **Ein Ansible Playbook wird sequentiell abgearbeitet, die Reihenfolge ist entscheidend!**  
        Playbooks sind im YAML-Format geschrieben, hier kommt es auf die passende **Einrückung** an, **alle Tasks müssen auf der gleichen Ebene starten!**

    === "Ausführliche Erklärung"

        **Ein Ansible Playbook wird sequentiell abgearbeitet, die Reihenfolge ist entscheidend!**  
        Füge die Tasks hinter dem Task `Ensure all hosts from web group are enabled`, aber vor dem Task `Create job template for infrastructure setup as Ops workload` ein.  
        Playbooks sind im YAML-Format geschrieben, hier kommt es auf die passende **Einrückung** an, **alle Tasks müssen auf der gleichen Ebene starten!**

??? question "Hilfe nötig? Klick mich..."

    Die folgenden beiden Tasks fügen die notwendigen *Project*-Objekte hinzu:

    {% raw %}
    ```yaml
    - name: Add project with playbooks for Ops workloads
      ansible.controller.project:
        name: Git Repository with Ops Playbooks
        default_environment: Default execution environment
        scm_type: git
        scm_url: https://github.com/ansible/workshop-examples.git
        scm_branch: webops
        scm_update_on_launch: true
        scm_delete_on_update: true
        scm_clean: true
        state: present

    - name: Add project with playbooks for Dev workloads
      ansible.controller.project:
        name: Git Repository with Dev Playbooks
        default_environment: Default execution environment
        scm_type: git
        scm_url: https://github.com/ansible/workshop-examples.git
        scm_branch: webdev
        scm_update_on_launch: true
        scm_delete_on_update: true
        scm_clean: true
        state: present
    ```
    {% endraw %}

    ??? example "Gesamtes Playbook"
        {% raw %}
        ```yaml
        ---
        - name: Prepare Automation Controller
          hosts: localhost
          connection: local
          module_defaults:
            group/ansible.controller.controller:
              controller_host: "{{ lookup('env', 'CONTROLLER_HOST') }}"
              controller_username: "{{ lookup('env', 'CONTROLLER_USERNAME') }}"
              controller_password: "{{ lookup('env', 'CONTROLLER_PASSWORD') }}"
          pre_tasks:
            - name: Ensure required connection variables are defined
              ansible.builtin.assert:
                that:
                  - lookup('env', 'CONTROLLER_HOST') is defined
                  - lookup('env', 'CONTROLLER_HOST') | length > 0
                  - lookup('env', 'CONTROLLER_USERNAME') is defined
                  - lookup('env', 'CONTROLLER_USERNAME') | length > 0
                  - lookup('env', 'CONTROLLER_PASSWORD') is defined
                  - lookup('env', 'CONTROLLER_PASSWORD') | length > 0
                quiet: true
                fail_msg: "Variables for accessing Automation controller are missing! Export the variables."
          tasks:
            - name: Ensure all hosts from web group are enabled
              ansible.controller.host:
                name: "{{ item }}"
                inventory: Workshop Inventory
                enabled: true
                validate_certs: true
              loop:
                - node1
                - node2
                - node3

            - name: Add project with playbooks for Ops workloads
              ansible.controller.project:
                name: Git Repository with Ops Playbooks
                default_environment: Default execution environment
                scm_type: git
                scm_url: https://github.com/ansible/workshop-examples.git
                scm_branch: webops
                scm_update_on_launch: true
                scm_delete_on_update: true
                scm_clean: true
                state: present

            - name: Add project with playbooks for Dev workloads
              ansible.controller.project:
                name: Git Repository with Dev Playbooks
                default_environment: Default execution environment
                scm_type: git
                scm_url: https://github.com/ansible/workshop-examples.git
                scm_branch: webdev
                scm_update_on_launch: true
                scm_delete_on_update: true
                scm_clean: true
                state: present

            - name: Create job template for infrastructure setup as Ops workload
              ansible.controller.job_template:
                name: Web App Deploy
                job_type: run
                inventory: Workshop Inventory
                project: Git Repository with Ops Playbooks
                execution_environment: Default execution environment
                playbook: rhel/webops/web_infrastructure.yml
                credentials:
                  - Workshop Credentials
                limit: web
                become_enabled: true

            - name: Create job template for infrastructure setup as Dev workload
              ansible.controller.job_template:
                name: Node.js Deploy
                job_type: run
                inventory: Workshop Inventory
                project: Git Repository with Dev Playbooks
                execution_environment: Default execution environment
                playbook: rhel/webdev/install_node_app.yml
                credentials:
                  - Workshop Credentials
                limit: web
                become_enabled: true

            - name: Create DevOps workflow for infrastructure setup and Nginx deployment
              ansible.controller.workflow_job_template:
                name: Deploy Webapp Server
                destroy_current_nodes: true
                workflow_nodes:
                  - identifier: Approve Deployment
                    unified_job_template:
                      name: Approve Deployment
                      type: workflow_approval
                    related:
                      success_nodes:
                        - identifier: Web App Deploy
                  - identifier: Web App Deploy
                    unified_job_template:
                      name: Web App Deploy
                      type: job_template
                    related:
                      success_nodes:
                        - identifier: Node.js Deploy
                  - identifier: Node.js Deploy
                    unified_job_template:
                      name: Node.js Deploy
                      type: job_template

            - name: Launch workflow
              ansible.controller.workflow_launch:
                workflow_template: Deploy Webapp Server
                wait: false

        ```
        {% endraw %}

!!! success "Alles erledigt?"

    * [x] Passendes Modul aus der Dokumentation identifiziert
    * [x] Task für Ops-Content Project hinzugefügt
    * [x] Task für Dev-Content Project hinzugefügt

## 4 - Automatisierung ausführen

=== "Kurz und knapp"

    Im Terminal wieder das folgende Kommando ausführen:

    ```console
    ansible-navigator run playbook_controller_automation.yml
    ```

=== "Ausführliche Erklärung"

    Sobald alles vollständig ist, kannst du die Automatisierung ausführen. Im Terminal wieder das folgende Kommando ausführen:

    ```console
    ansible-navigator run playbook_controller_automation.yml
    ```

    Du wirst die einzelnen Task-Beschreibungen mit einem gelben *Changed*-Status sehen (wenn die Objekte angelegt/angepasst) werden, **wenn alle Tasks erfolgreich abgearbeitet sind endet der Playbook-Run mit einem `Play Recap`**.

!!! quote ""

    **:rocket: - Weiter gehts mit der [Operations Stage!](part3.md)**
