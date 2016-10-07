---
layout: post
title: Containers and its Orchestration
tags:
- Docker
- Orchestration
categories:
- Cloud
---


<div class="message">
  Hy! In questo post voglio  classificare diversi software che gestiscono/creano/usufruiscono di una tecnologia che è diventatata "hot" nel corso degli ultumi 2/3 anni.
</div>
"<!-- more -->"
#Cosa Sono i Container
La prima  cosa da chiarirsi è che cosa sono i Container,  perchè sono un argomento di fortissimo interesse nel mondo IT odierno , quali sono i loro punti di forza e quali le loro debolezze.

>I container sono degli "ambienti" che tramite l'utilizzo di alcuni  servizi runtime del sistema operativo Linux/Unix permettono alle applicazioni, configurate al loro interno di godere di un certo grado di **isolamento**,  semplificando così la fase della loro messa in opera (deployment). Questa semplificazione, unitamente all'introduzione di architetture a **microservizi** delle applicazioni stesse porta ad una accelerazione del ciclo di rilascio del servizio in produzione,con evidente vantaggio in termini di flessibilità e reattività del prodotto/servizio rispetto al mercato.

Volendo essere più chiari, il kernel linux/unix mette a disposizione dei file system , degli alberi dei processi , della memoria , dei device che "appaiono" all'applicazione essere quelli del sistema operativo  Host e gli permette di utilizzarli in modo esclusivo  ma in realtà le risorse sono condivise e gestite con le stesse strutture dati dello stesso kernel.

Quella precedente è la definizione diciamo dei puristi , in effetti un altro motivo di rilevanza dei container, soprattutto in una loro implementazione ovvero Docker, è la speranza che essi vadano ad occupare fette di mercato  (almeno una parte) delle Virtual Machine tradizionali , ovviando ad alcuni "problemi" quali uso inefficiente delle risorse disco/memoria e lentezza nella fase di boot, producendo in ultima analisi una riduzione dei costi.

Dalla contrapposizione dei container alle Virtual Machine, emorgono  i punti di debolezza dei primi rispetto alle seconde : il livello di isolamento dal mondo esterno e quindi il livello di sicurezza introdotto (soprattutto nei confronti del mondo esterno). Infatti mentre le VM all'interno del singolo host condividevano solamente la CPU e alcuni driver in caso  di para-virtualizzazione, i container condividono l'intero sistema operativo, shared library e driver.

In effetti bisogna dire che le maggior debolezza dei container in termini di sicurezza si presenta nei sistemi Linux in quanto altri sistemi operativi più vicini al mondo mainframe già prevedevono forme di container native (system Z Ibm) o aggiunte  (Solais Zone) i cui in effetti esisteva una segmentazione (duplicazione) delle risorse all'interno del Kernel. 

Quindi gli sforzi per mantenere n livello di sicure a livello Enterprise si muve verso 3 direttrici diverse : 

- eseguire i container su sistemi operativi diversi da linux (joyent clud provider operante su SmartOS riprendente concetti di Zone Solaris)

- eseguire i workload  su  hypervisor light specializzati per container (LXD di Canonical) in questo caso i container eseguono un proprio sistema operativo al di sotto delle applicazioni (system container)

- aumentare la sicurezza intrinseca all'interno dei container applicando i metodi quali Apparmor/SELinux o configurando oppurtunamente le capabilities dei processi all'interno dei container

Ma perchè pur essendo le tecnologie che permettola realizzazione dei container presenti da decadi, solamente ora si è osservato un grande interesse ? Perchè  finora tutte le implementazioni non prevedevano un controller ed un set di tool che facilita notevolmente (enormemente) la messa in opera di un comtainer. Basti pensare che il prodotto più diffuso (e forse anche il più simile a docker) presenta un set di Template da applicare/modificare a secondo dei tipi dei workload previsti. In effetti la grande innovazione di docker è la presenza del controller (processo docker spazio user dell'Host) che permette l'attivazione del container a partire da immagini (di tipo docker) precostituite e presenti su repository pubblici. 
in effetti il valore di Docker è  soprattutto nei registry pubblici dove ormai sono  presenti migliaia di immagini di applicazioni diverse costruite in modo incrementale da Template (immagini certificate ..) tanto che altre soluzioni di container mantengono lo stesso formato o lo ampliano (vedi RKT di Coreos) ma mantenedo comunque la compatibilità.

Vediamo ora come i container Docker vengono gestiti e che ecosistema di tool,management software , scheduler , hanno generato anche indirettamente , ovvero presso i possibili competitors..  

-----

