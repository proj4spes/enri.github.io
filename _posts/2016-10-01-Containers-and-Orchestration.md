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

## Cosa Sono i Container

La prima  cosa da chiarirsi è che cosa sono i Container,  perchè sono un argomento di fortissimo interesse nel mondo IT odierno , quali sono i loro punti di forza e quali le loro debolezze.

>I container sono degli "ambienti" che tramite l'utilizzo di alcuni  servizi runtime del sistema operativo Linux/Unix permettono alle applicazioni, configurate al loro interno di godere di un certo grado di **isolamento**,  semplificando così la fase della loro messa in opera (deployment). Questa semplificazione, unitamente all'introduzione di architetture a **microservizi** delle applicazioni stesse porta ad una accelerazione del ciclo di rilascio del servizio in produzione,con evidente vantaggio in termini di flessibilità e reattività del prodotto/servizio rispetto al mercato.

Volendo essere più chiari, il kernel linux/unix mette a disposizione dei file system , degli alberi dei processi , della memoria , dei device che "appaiono" all'applicazione essere quelli del sistema operativo  Host e gli permette di utilizzarli in modo esclusivo  ma in realtà le risorse sono condivise e gestite con le stesse strutture dati dello stesso kernel.

Quella precedente è la definizione diciamo dei puristi , in effetti un altro motivo di rilevanza dei container, soprattutto in una loro implementazione ovvero Docker, è la speranza che essi vadano ad occupare fette di mercato  (almeno una parte) delle Virtual Machine tradizionali , ovviando ad alcuni "problemi" quali uso inefficiente delle risorse disco/memoria e lentezza nella fase di boot, producendo in ultima analisi una riduzione dei costi.

Dalla contrapposizione dei container alle Virtual Machine, emergono  i punti di debolezza dei primi rispetto alle seconde : il livello di isolamento dal mondo esterno e quindi il livello di sicurezza introdotto (soprattutto nei confronti del mondo esterno). Infatti mentre le VM all'interno del singolo host condividevano solamente la CPU e alcuni driver in caso  di para-virtualizzazione, i container condividono l'intero sistema operativo, shared library e driver.

In effetti bisogna dire che le maggior debolezza dei container in termini di sicurezza si presenta nei sistemi Linux in quanto altri sistemi operativi più vicini al mondo mainframe già prevedevono forme di container native (system Z Ibm) o aggiunte  (Solais Zone) i cui in effetti esisteva una segmentazione (duplicazione) delle risorse all'interno del Kernel. 

 ***Quindi gli sforzi per mantenere un livello di sicurezza a livello Enterprise si muove verso 3 direttrici diverse :***

- eseguire i container su sistemi operativi diversi da linux (joyent Triton cloud provider operante su SmartOS riprendente concetti di Zone Solaris)

- eseguire i workload  su  hypervisor light specializzati per container (LXD di Canonical) in questo caso i container eseguono un proprio sistema operativo al di sotto delle applicazioni (system container)

- aumentare la sicurezza intrinseca all'interno dei container applicando i metodi quali Apparmor/SELinux o configurando oppurtunamente le capabilities dei processi all'interno dei container

 ***Ma perchè pur essendo le tecnologie che permettola realizzazione dei container presenti da decadi, solamente ora si è osservato un grande interesse ?*** Perchè  finora tutte le implementazioni non prevedevano un controller ed un set di tool che facilita notevolmente (enormemente) la messa in opera di un container. Basti pensare che il prodotto più diffuso OpenVZ (e forse anche il più simile a docker) presenta un set di Template da applicare/modificare a secondo dei tipi dei workload previsti. In effetti la grande innovazione di docker è la presenza del controller (processo docker spazio user dell'Host) che permette l'attivazione del container a partire da immagini (di tipo docker) precostituite e presenti su repository pubblici. 
In effetti il valore di Docker è  soprattutto nei registry pubblici dove ormai sono  presenti migliaia di immagini di applicazioni diverse costruite in modo incrementale da Template (immagini certificate ..) tanto che altre soluzioni di container mantengono lo stesso formato o lo ampliano (vedi RKT di Coreos) ma mantenendo comunque la compatibilità.

Vediamo ora come i container Docker vengono gestiti e che ecosistema di tool, management software, scheduler, hanno generato anche indirettamente , ovvero presso i possibili competitors..  

Diciamo subito che gli attori che hanno abbracciato il credo Docker appartengona a svariate categorie che vanno dalle Startup (prima tra tutte quelle che si occupano di infrastrutture  per il Clou ovviamente), ai grandi Cloud Provider nella declinazione Iaas, Paas a Saas, ai grandi vendor di OS e ambienti virtualizzati, ai fornitori di servizi applicativi .. e tanti "piccoli" sviluppatori che usano Docker per semplificare il loro lavoro.

Quindi alcuni tool (o architetture ) saranno più adatti, ad esempio, al mondo degli sviluppatori mentre altri saranno interessanti per gli amministratori di DataCenter ad esempio.

Poviamo ad elencarli per categorie funzionali ovvero in base alle esigenze a cui rispondono.

## Dove viene eseguito Docker ?

- su bare metal ovvero pc di casa ... nienta da dire   Build,Ship and Run e divertitevi

- su bare metal all'interno di container "nativi" (vedi sopracitato Joyent) : procuratevi il Docker patchato dal fornitore del container esterno ( docker non prevede di operare con namespce confinati , i cgroup device (virtual) non possono essere su AUFS , il processo deve avere accesso privilegiato ad alcune strutture linux..)

- su virtual machine : in questo caso le accortezze sono di scegliere OS guest leggeri(in ram) e veloci a "salire" e  che comunque supportino docker. L'estremizzazione di ciò è rapppresentata dalla tecnologia (sperimentale a livello di prodotto generale) degli unikernel.  A tal proposito và detto che Docker Inc. ha acquisito all'inizio dell'anno una delle società più prometttenti in questo settore (Unikernel Systems) e grazie a questa acquisizione ha rilasciato in Beta (privata) "Docker per Windows" e Docker per OSX, infatti in queste soluzioni Docker opera in VM speciali il cui OS guest è un Unikernel (nota: l'hypervisor per OSX è un performante Open Source che opera in UserSpace . Un'altro approccio meno spinto  ma più prodotto può essere considerato il Photon  OS di VMware (viene offerta un' immagine linux minimale ma con un set di librerie ottimizzate a gestire Virtual Hardware ESXi) . Photon Os è naturalmnte integrato in vSphere.

- più genericamente in Cloud : se intendete usare docker(standalone)  nel vostro public Cloud di riferimento ed intendete integrarlo con altre funzionalità tiptiche del cloud (ad esempio integrandolo con un autoscaler ivi presente) e quindi non operando solo a livello Paas, è importante notare che tutti i maggiori attoti AWS, GCE, Azure ..) non presentano tool di "configurazione" compatibili con  Compose Docker. Quindi il progetto inteso come package dovrà tener conto di diversi file di "configurazione" per il deployment in produzione presso il Cloud .


## Come viene generato un singolo container di tipo Docker? Quali sono le sue caratteristiche?

 Volendo semplificare un container di tipo Docker è un processo che esegue delle chiamate ai servizi di Kernel cgroups (per ottenere un certo grado di separazione e limiti  sull'utilizzo delle risorse dell'Host quali cpu, memoria ,  device ..per ogni processo/gruppo di processi)  e namespaces ( per ottenere  l'univocità dei nomi delle risorse all'interno dei container); quindi monta un file system root  contenente tutto quello che serve all'applicazione (di tipo Copy on write o a cipolla per essere più chiari) e delle librerie per fornire i servizi di netwoking ed accesso ai volumi ed infine lancia l'applicazione desiderata.
Tutto questo viene effettuato secondo quanto scritto in un manifesto (metadati del container) associato all'immagine del container.
Quindi l'immagine così definita viene gestita nel su ciclo di vita da un deamon che a sua volta espone un'API utilizzata o da una CLI(command Line) o da remote agent (in effetti l'architettura è leggermente più complicata, cfr rel docker 1.12 Docker). Questo daemon è in effetti ciò che differenzia i container tipo docker dai container tipo LXC, Zones,Jail, ed in misura minore da OpenVZ ed LXD ed effetivmente è quello che ha decretatato il successo dei container di tipo Docker.
Perchè  continuo a citare "container di tipo Docker" ? Perchè in effetti ad oggi esistono 2 formati per le immagini ( e relativi daemon) per creare/gestire i container :  appc  e docker legacy (di gia?!) .
Il primo è stato implementato nel sistema RKT da CoreOs, in contrapposiziona al formato Docker iniziale (anzi definendo per la prima volta un formato delle immagini che voleva essere standard).
RKT ovviamente è in grado di eseguire **automaticamente** anche le migliaia di immagini Docker , oltre a quelle definite tramite lo standard "Appc Container Image" (ACI); Docker dal canto suo converte le immagini Aci in immagini Docker con un  tool  offline Aci2docker.
Stando così le cose si profilava un profonda spccatura all'interno dell'ecosistema docker :Google, RedHat , VMWare, Apcera-Ericsson, Mirantis  aderivano ad Appc non solo per motivi "politici" (effettivamente Appc permette di implementare diversi di tipi di servizi Container ed un maggior grado di sicurezza )
2 Notizie : una buona ed una meno -> a giugno 2015 viene definito dall'Open container Initiative  uno Image-specification che raccoglie i contributi sia di docker e (soprattutto) di Coreos e con la release 1.11 docker rilascia un modulo *runC* in grado convertire i formati docker in formato OCI (Open Container Image)  e di lanciarli e gestirli. 
La cattiva notizia è che  nella componente di runtime dello standard il networking previsto e sviluppato da docker non è compatibile con gli sviluppi di altri ( google e kubernetes in primis)

## Come viene automaticamente configurato un Host per eseguire Docker?
Il tool di riferimento fornito da Docker è **Docker Machine** che è un tool che fornendo driver (canali di comunicazione ) verso i pricipali Cloud provider (AWS,GCE,..) o verso i principali ambienti virtuali (hyper-Vm virtualbox...) crea un Docker Host (ovvero installa un Docker Deamon/engine) e poi invia commandi di tipo Docker client e quindi crea/lancia..docker container sugli host remoti/locali
Altri strumenti utilizzabili per raggiungere gli stessi risultati possono essere sia Ansible o Terraform di HashiCorporation ; avendo entrambi le funzionalità di provioning di Host tramite driver e di configurazione degli stessi.


## Come viene allocato un container Docker all'interno di un cluster di Host ?
Questa funzione è svolta dalla categoria dei sistemi chiamati  Container scheduler ed è  dove vengono realizzate le funzioni di allocazione dinamica,  di service recovery,  di scale-out, di service discovery, di load balancing sui servizi , di connessione ai servizi di networking e di volume management ; è ovvio quindi che l'attenzione del mondo del Cloud si concentri su queste architetture . ed è anche la categorie dove più si differenziano a mio parere,  i sistemi in funzione dei loro reali utenti finali.
Moltissimi sono i sistemi che possono rientrare in questa categoria , alcuni in esercizio da anni (scheduler di VM  come BOSH di Cloud Foundry) , altri appena nati ma già con interessati caratteristiche.. 
Io ne citerò solo quelli di maggior risonanza e più specificamente rivolti ai Docker Container. Primo ad essere nominato non può essere che docker Swarm : il  cluster è gestito da almeno 3 Master che sissinconizzano tramite un Consensus protocol interno o basato su RAFT di Consul HasshiCopr, sui nodi del cluster è presente il docker deamon/engine e sia le cli sia il tool di deployment dei Services (docker compose ) sono quello standard di Docker. Il Cluster è relativamente facile da attivare, il networking centralizzato su server Key-Value  interno (per service networking e service discover interno) o  basato su Consul altrettanto semplice.  

Il secondo Scheduler per risonanza è senz'altro **Kubernets** . In questo caso il cluster è più complicato da attivare perchè esistono processi specifici per specifiche funzionalitè o ruoli , caratteristiche del sistema che lo differenziano definitivamente del Docker Swarm (oltre ad un numero di funzionaità superiori)  sono: l'unitàminima di allocazione sono  i Pod (insiemi  di container  che condividono lo stessso network spacename (condividono lo stesso indirizzo  IP) , le label e i selettori che permettono di gestire in siemi di Pod in modo uniforme , 
il servisio di Networking è distribuito su ogni nodo del Cluster così come il servizio di dns resolver per i servizii, i container potranno essere di 2 tipi  : Docker (runC ad oggi) e RKT (appc) , una prima definizione di Pod statefull (Petset) anche se con limitazioni.

Se finora i sistemi descritti possono esere gestiti da una singola persona o da un piccolo team  i seguenti sistemi sono rivolti a grandi provider  per gestire cluster di grandi dimensioni. Tra essi possiamo elencare Mesos/Marathon e CoreOS. Mentre il primo è un sistema di scheduling (non solo per container) per framework (ovvero applicazioni distribuite) che insistono sullo stesso cluster ; ad esempio Marathon è il framework che alloca i container o in generale i workload eterogenei  mentre Chronos è una sorta di Cron operante su cluster mentre il secondo (CoreOs è un insieme di sistemi/tool loosed coupled che interagiscono per fornire un servizio di cluster management (operativo sul Cloud Tectonic).

 Da ultimo vorrei menzionare il sistema ***Nomad-Consul-Vault*** della HashiCorp. Il primo sistema  è uno scheduler molto veloce e facile da installare . Il second sistema (Consul ) è un sistema Key-value distribuito con protocollo RAFT integrato e servizi di Health-checking associato al service discoovery implentabile. Il terzo (Vault) è un sistma per la conservazione ei segreti  Il terzo (Vault) è un sistma per la conservazione dei segreti. I 3 sottosistemi sono ovviamente integrati molto bene  e offrono insieme funzioni tipiche di una piattaforma evoluta pur essendo questi prodotti molto "giovani". Bisogna dire Nomad non è solamente  uno scheduler di Container (al pari di Mesos) e quindi di interesse per gli eventuali integratori di rete  ibride. Unitamente al fatto che esiste un'altro prodotto ***"Terraform"*** di Hashicorp che è un sistema per deployment in fisico (Openstack ) e in cloud (molto buona l'integrazione con AWS ad esempio ) direi che HashiCorp è una società da tenere d'occhio.




## Come faccio a mantenere/gestire dei dati in modo persistente all'interno di un Container ?

 Volendo semplificare (semplificare molto) ci sono solo 3 modi : 

1. Bypasso il file file system UFS  e scrivo su disco locale  (volume driver) o monto una directory da Host 
2. Mi connetto ad un container che si comporta come al punto 1 
3. Utilizzo un cluster di storage/volume che fornisce un servizio (astraendo l'implementazione magari) di accesso ad un pool distribuito di volume/storage

Docker implementa 1 e 2 nativamente ed i 3 tramite plugins, Kubernet tutte e 3 le modalità e quando vado ad elencare i cluster supportati  sembra di leggere la documentazione di Openstack (livello di integrazione tipico delle IaaS).... Allo stesso tempo kubernetes inizia ad introdurre concetti di Volume Claim ( richiesta di  volume che nasconde le specifiche implementative -servizio tipico delle PaaS) . Tra tutti i tipi di cluster supportati da Kubernetes (Ceph, glusterFS , Nfs ...) ve ne è uno che si differenzia da gli altri :flocker. Quest'ultimo associa il volume (chiamato DataSet ) tipicamente al singolo container al momento della run  e nomina univocamente il container; quindi quando spostiamo (distruggiamo e ricreiamo il conatainer ) su un nuovo host riasssocia automaticamente il volume (che si può appoggiare su storage provider anche di tipo cloud :ebs di aws..) alla nuova istanza di container.. .  E' quello che fa anche l'implementazione dei PetSet di kubernetes (funzione rilasciata in alpha)...e stiamo iniziando  a parlare di FullState container o FullState application a micro-servizi .. ma questa è un'altra  storia...

## Network for Container
Affrontare in modo esaustivo tutte le caratteristiche dei vari sistemi per connettere i container distribuiti su più nodi o VM ( presso i vari Cloud provider) non l'obiettivo di questo blog .
Possiamo iniziare a classificare i sistemi di Networking per Cnatainer secondo :
1. la loro architettura di rete per connettere Conatainer su  Host diversi
2. la presenza di un storage cluster esterno per mantenere la configurazione e le route
3. I protocolli supportati
4. la presenza di DNS interno
5. l'integrazione con le IP tables
6. l'integrazione con soluzione di tipo SDN 
7. la modalità di integrazione come plugins o add-on nelle varie piattaforme di clustering
8. la possibilità di cryptare i flussi
9. etc. etc.

Diciamo comunque a parte Docker che ha sviluppato una sua soluzione "Docker Overlay Network" i oltre apoter usare altri plugins di tipo network , le altre soluzione (kubernetes , Mesos..) utilizzano essenzialmente i servizi di rete sviluppati terze parti.  Tra le più conosciute citiamo :

- ***Flannel*** fornisce un'Overlay network (Vxlan o UDP) che opera a livello 2 (MAC) , non presenta un DNS e si appggia ad un storage esterno   etcd 

-  ***Weave*** fornisce un'Overlay network (Vxlan o UDP) che opera a livello 2 (MAC) , presenta un DNS interno e non si appggia ad un storage esterno ma implementa un protocollo di tipo "Rumors" con cui scambia le route tra tutti gli agenti. Presenta enrcryption (non TLS) . Particolarità : si integra con Kubernetes secondo lo standard (scelto da Kubernetes per i Plugin ) CNI non supportato da Docker. Gli agenti sui nodi effettuano anche la funzione di transito permettendo così topologia sia di tipo mesh ma anche "Partially connected"

- ***Calico*** realizza una rete a livello 3 BGP

Infine bisogna dire che tutti i sopraccitati sistemi sono comunque in grado di sfrutttare i backend di rete dei principali Cloud Provider (AWS e GCE) al fine di sfruttare le funzioni agiuntive che quest'ultimi offrono come ELB .

## Operating System Docker Based
Tra le caratteristiche che hanno portato alla ribalta i container (di tipo Docker) sono la facilità di installazione , la struttura a cipolla (ovvero il deployment incrementale) ed l'atomcità del singolo container, ovvero il container è auto-contenente : si può concellare o aggiungere con un singolo comando , ma non si può cambiarne il contenuto (ovvero l'immagine). Queste caratteristiche in effetti sono interessanti anche per sistemi quali i packet manager ed in generale anche per la gestione dei sistemi operativi intesi come aggregati di pacchetti applicativi da gestire (update, upgrade, roll-back, controlli su dipendenze ...). In questa direzione si sono mossi Atomic (di RedHat ) e RancherOs . Quest'ultimo per esempio ha portato a l'estremo il concetto di Sistema Operativo a container , infatti RancherOs prevede il kernel linux ma sopra questo opera direttamente un processo Docker (detto System Docker) invece di processi quali init/systemd. Questo processo Docker crea alcuni container di sistema che forniscono servizi di sistema quali dhcp, udev, console, rsyslog e logind ed un container che ospita al suo interno un'istanza di Docker engine. Sopra quest'ultimo docker container (grazie a) vengono creati e gestiti tutti container di tipo applicativo (user level container).
Questo approccio,  anche se può sembrare eccessivo, ha portato ad avere un OS leggero, facilmente gestibile e con una fase di boot molto veloce ; ovviamente a discapito della tipologia dei workload ospitati (no IPC e principalmente Stateless). Bisogna anche dire che questo modello di deployment (autoreferenziale) è sempre più diffuso nelle applicazioni nel mondo dei container : ad esempio ultimamente è stata rilasciata in alpha una "procedura" (un comando in CLI) per semplificare l'attivazione di un cluster Kubernetes . Tale procedura prevede l'installazione (come processi linux con il package manager di riferimento dell'host) solamente di docker.io (engine di docker) , di  kubelet (engine di kubernetes) e del commando kubeadm su ogni nodo del cluster. Ogni altro componente del sistema (Api server , clusterControllerManager , etcd,  , kubeproxy , kube-discovery, network,  cAdvisor...) vengono lanciati come container (in Pod - di tipo Deamonset) e vengono preconfigurati con Manifest file come le applicazioni...
 
## Casi di utilizzo dei Container e dele relative piattaforme

Come detto all'inizio del Blog i container di tipo Docker hano un immediato successo presso gli sviluppatori Web , perchè facilitava loro le fasi testing e deployment ed infatti Docker ed il suo standard di container sono ormai lo standard DEFACTO per soluzioni tipo "Web multi tier application" o "Web services". Altro utilizzo di Docker Stand-alone sono i servizi tipo Container On Demand o OnTime schedule o On Event Container (come AWS Lamda)
 Altra cosa  è invece implementare HyperGrid o Hadoop,Spark cluster as Service ; in questo caso una piattaforma tipo Mesos è senz'altro più indicata sia per il controllo alivello di scheduling  che per la scalabilità offerti.
Kubernetes invece sia per i servizi realizzati e correntemente proposti, oltre a quelli promessi, ed anche per i suoi sponsor (Google, RedHat..) ed effettivamente per il modo strutturato di offrire servizinei container/pod (opinionated dicono gli anglosassoni) si presta ad essere considerato come la piattaforma per sistemi di livello superiorei (intesi ontop che aggiungono valore di tipo applicativo) come le Paas (Openshift, Deis) o le MBaaS (Mobile Backend as a Service) o più generalmente parlando  per tutte le applicazioni complesse basate a Micro-servizi.



-----

