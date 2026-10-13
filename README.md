# AWX Azure Lab

Progetto Ansible di laboratorio per simulare il flusso GitHub -> AWX -> VM Linux con Tomcat 9. Il repository contiene esclusivamente codice di test, senza configurazioni aziendali, credenziali o indirizzi delle VM.

## Struttura

```text
ansible.cfg
environment/
  common/prepare-ssh.yml
  lab/verify.yml
  lab/deploy.yml
```

`verify.yml` verifica SSH, Python, Tomcat 9 e HTTP. `deploy.yml` scarica il WAR sample pubblico Apache sulla VM, verifica SHA-256 e attende la servlet Hello World. Il deploy non installa Tomcat, non crea VM e non riavvia il servizio.

## Configurazione AWX

1. Creare un Project di tipo Git con URL di questo repository e branch `main`.
2. Abilitare Update Revision on Launch per sincronizzare il codice prima del job. Il repository pubblico non richiede una credenziale SCM.
3. Usare un Inventory con gruppo `tomcat_lab` e host di laboratorio.
4. Impostare sull'host `ansible_host`, `ansible_python_interpreter: /usr/bin/python3` e `lab_ssh_known_hosts`: riga known_hosts della VM verificata in precedenza tramite un canale attendibile.
5. Associare una credenziale Machine SSH, utente `ansible`, con sudo. La chiave privata va conservata in AWX, mai nel repository.
6. Creare Job Template con playbook `environment/lab/verify.yml` e `environment/lab/deploy.yml`; entrambi usano l'Execution Environment AWX 24.6.1 nel laboratorio iniziale.

Prima delle connessioni remote, i playbook preparano un file known_hosts temporaneo nell'Execution Environment con la chiave host fornita dall'inventario. SSH usa StrictHostKeyChecking=yes. Non viene acquisita automaticamente una chiave non verificata.

## Flusso di modifica

Modificare il codice in un branch di lavoro, revisionare le differenze e integrare in `main` con una pull request. AWX aggiorna il progetto e registra la revisione Git usata dal job. Prima del deploy eseguire il template di verifica. Per un rilascio riproducibile si può scegliere un tag o un commit specifico al posto di `main`.

Il WAR viene scaricato da https://tomcat.apache.org/tomcat-9.0-doc/appdev/sample/sample.war; la servlet è `/sample/hello`. Se Apache cambia l'artifact, il checksum fissato provoca un errore: verificare il nuovo WAR prima di aggiornare il checksum.

## Limiti del laboratorio

Questo progetto non integra Jenkins, Artifactory aziendale, inventario dinamico Azure, database applicativi o Liquibase. Non è una copia del repository aziendale environment-configuration. Simula la sincronizzazione Git del Project AWX e il rilascio su un singolo target Tomcat già predisposto.

Per un repository privato serve una credenziale Source Control con accesso in lettura; è distinta dalla credenziale Machine usata per SSH alla VM. La visibilità pubblica di questo esempio non modifica l'accesso alla VM.

Riferimento: [AWX Projects](https://docs.ansible.com/projects/awx/en/24.6.1/userguide/projects.html).
