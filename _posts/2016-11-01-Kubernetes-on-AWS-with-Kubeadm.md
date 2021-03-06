---
layout: post
title: Come installare Kubernetes su AWS
tags:
- Kubernetes
- DevOps
categories:
- Cloud
---


<div class="message">
  In questo Blog voglio esaminare  brevemente quali sono le attuale modalità di installazione per il sistema di orchestrazione di Kubernetes  e poi soffermarmi 
su l'ultimo (cronologicamente) tool di installazione Kubeadm per sottolineare quale siano i suoi vantaggi ed gli attuali limiti.
</div>
"<!-- more -->"
 Iniziamo con il dire che la parte di installazione di kubernetes ha una particolare imporanza per lo sviluppo (in termini di percentuali di mercato) per il prodotto in questione; questo per 2 motivi :
-  il competitor attuale principale ovvero Docker essenzialmente viene installato, indipendentemente dal fatto che l'Host sia fisico  o in virtuale in Coud)   in 2 soli modi : a mano  (tramite il package manager di riferimento del OS Host ) o tramite Docker-Machine. Questo è possibile in quanto il cluster di  Docker (swarm) è "auto-contenente" (un esempio ne è  il load balancing interno) mentre la componente Docker engine è  lo standard DeFacto per i container ed è molto facile sia da installare sia da integrare in altri prodotti (semistrutturati come DCOS/Mesos ad esempio).

 Questo in effetti  si traduce anche in  una strategia di sviluppo dei 2 prodotti  : ad applicazioni interagenti  (loosly coupled) per Docker od applicazione  a moduli strettamente connessi/integrati per kubernetes.

- Kubernetes si presenta e si presenterà sempre più come piattaforma (di sviluppo/integrazione) di Container per altri sistemi di più ampio respiro (Openstack ad esempio) e per questo motivo (oltre a quelli sopracitati di strategia di sviluppo)  le modalità di installazioni/migrazioni della singola componente devono essere ben definite . 

Nonostante quanto detto attulamente kubernetes si installa in un quantità di maniere differenti a seconda della piattaforma ospitante (anche se in effetti si riducono essenzialmente a 2/3 modalità e  utilizzanti i soliti 2/3 tool di deployment..salt, ansible, chief oltre ai sistemi di template quali CloudFormation per i cloud Provider. Questo perchè effettivamente Kubernetes è un progetto più giovane rispetto a Docker , sia perchè la community è molto più vasta e meno controllata rispetto a quella di Docker sia perchè infine il livello di integrazione di kubernetes con i sistemi che lo "ospitano" è sicuramente superiore a quello di Docker ( e varia anche a secondo della piattaforma dove possiamo dire che ad oggi il livello di integrazione maggiore ovviamente si ha con Google Container Engine). 

Infine bisogna dire che comunque questa fase di anarchia giovanile stà velocemente  decrescendo infatti volendo idendificare tre fasi per installare kubernetes :

1) Provisioning dei sistemi 

2) deployment delle componenti del sistema

3) integrazione / riconoscimento / discovery/autenticazione delle componenti stesse 

la prima fase ovviamante manterrà delle differenze (magari nascoste da tool multi-provider quali Terraform) mentre le altre 2 fasi saranno risolte in modo univoco utilizzando tools quali **KubeAdm** o **Kops** che sfruttano i Manifest (come i Configuration Files ma per le componenti si sistema), gli oggetti Deamonset(.yaml) e  l'engine dei pod (kubelet ) per "costruire" il sistema.
In questo Blog porterò l'esempio di un' installazione di un cluster kubernetes con il tool Kubeadm su Aws  illustrandone (secondo me ) i pregi ed i limiti.

> Nella release 1.4 (settembre 2016) kubernetes introduce il comando Kubeadm, il quale riduce la fase di start del cluster a soli 2 comandi : kubeadm init (da eseguire sul nodo aster) e kubeadm join (da eseguire sui nodi worker) . Questi 2 comeandi corrispondono ale fasi 2) e 3) sopracitate, mentrec il provisioning dei nodi verrà effettuato con Terraform che creerà l'infrastruttura AWS-VPC, i nodi e lancerà (tramite "user data" Aws anche  i suddetti commandi .Tale semplificazione si è raggiunta anche tramite la creazione di package apt-get e yum per i principali componenti (kublet, kubeadm, kubectl, kubernetes-cni) e la creazione di Deamonset per molti add-ons ultimo tra i quali quello per la rete Weave nel nostro caso. 
Altra feature introdotta in alpha in rel.1.4 ed adottata da Kubeadm è il TLS boostrap API, ovvero una lib per facilitare l'attivazioni di TLS  verso la componente "API server" (sul master) da parte dei kubelet (sui nodi). Questa API richiede una firma di tipo Token Cryptato (costruito esternamente e inietttato sia nel master che nei nodi) per attivare le procedure di instaurazione dei TLS (csr-certificate signing request). Vediamo di seguito come installare il cluster.

## Come installare

### Prerequisiti

Bisogna avere installato Terraform , aws cli (ed avere un account AWS ) , python 

### Macro step  per l'installazione

- Generare il token :
```bash
 python -c 'import random; print "%0x.%0x" % (random.SystemRandom().getrandbits(3*8), random.SystemRandom().getrandbits(8*8))' 
```
Il token verrà iniettato tramite Terraform nelle user_data delle istanze aws per il master ed i worker (nodi)

- Generare le keys ssh 
``ssh-keygen -f k8s-kubeadm-test`` . le ssh-key verranno  installate su tutti i nodi

- Esegui Terraform plan :
``Terraform plan -var k8s-ssh-key="$(cat k8s-kubeadm-test.pub)" -var 'k8stoken=<token>'
``
- Esegui Terraform apply :
``terraform apply  -var k8s-ssh-key="$(cat k8s-kubeadm-test.pub)" -var 'k8stoken=<token>'``

- Collegati con ssh al master (vedi output di terraform) :``ssh ubuntu@$(terraform output ip o DNS ) -i k8s-kubeadm-test``


- esegui sul master :
 ``$ kubectl get nodes``
per verificare che il cluster e tutti i worker siano attivi. La risposta dovrebbe essere simile alla seguente:

{% highlight js %}
NAME             STATUS    AGE
ip-10-0-100-52   Ready     1d
ip-10-0-100-53   Ready     1d
ip-10-0-100-93   Ready     1d
{% endhighlight %}

**Complimenti se il risultato è quello qui sopra  hai appena installato un cluster di 2 nodi  kubernetes ed un master su altrettante istanze (VM os ubuntu 16.04) AWS .** Le istanze sono state attivate su 2 AZ differenti della region Irland (eu-west-1) in 2 subnet differenti . Inoltre, essendo la VPC non di default è stato attivato un internet gateway e relative tabelle di routing per le 2 subnet. 
**Ma...c'è sempre un ma, specialmente per i software rilasciati in alpha infatti tra i limiti più importanti  di kubeadm vi è l'impossibilità di creare loadBalancer (di tipo elb AWS gestito da kubernetes) e di supportare i PersistentVolume (integrabile con ebs AWS) oltre a non supportare un cluster di master (e di etcd) e un CA temporanea.**

 Quindi se volessi creare un servizio che opera su più nodi dovrei dovrei definirmi un ingress usando sistemi quali nginx o  traefik oppure attivare l'ELB di Aws previo descrizione del servizio in qualche file di configurazione (da capire come fanno gli altri tools d'installazione!!) . Opto per una soluzione molto più semplice (anche se molto meno elegante) : creo un ASG (AutoScalingGroup) AWS , i worker li metto sotto l'ASG  e per ogni servizio kubernetes che creo (con TYPO= NodePort) aggiungo un listener ed  un healty check sulla porta (per il primo servizio creo ache l'elb) degli hosts che mi viene restituita dal "describe svc".
> Intanto notiamo subito un particolare : se cambio il numero di istanze nell'ASG (aumentandolo di 1 ad esempio) usando terraform ( che mantiene lo stato ed esegue l'incremento) la nuova istanza esegue il "Kubeadm join" in user_data , quindi attiva il Daemonset di rete Weave ed a tutti gli effetti in cluster in rete con tutti gli altri nodi. Quindi Terraform (pur con qualche problema a mantenere lo stato a fronte di alcuni destroy) , in questo caso continua a mantenere lo stato -quindi continua ad essere documento di riferimento del cluster e allo stesso tempo è strumento di ampliamento (in termini di deployment di macchine fisiche) del cluster kubernetes (**Questa caratteristica è unica tra le modalità di installazione di kubernetes al di fuori di Google GCA , sia per features di Kubeadm sia per feature di terraform**)

Vediamo ora il come viene effettuato il deployment AWS in terrraform ed il provisioning tramite user_data.
{% highlight js %}

/*
test  kubeadm
*/

provider "aws" {
  access_key = "${var.access_key}"
  secret_key = "${var.secret_key}"
  region     = "${var.region}"
}

# Key pair for the instances
resource "aws_key_pair" "ssh-key" {
  key_name = "k8s"
  public_key = "${var.k8s-ssh-key}"

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_vpc" "main" {
    cidr_block = "10.0.0.0/16"
    enable_dns_hostnames = true

    tags {
        Name = "TF_Kube"
    }
}

resource "aws_internet_gateway" "gw" {
    vpc_id = "${aws_vpc.main.id}"

    tags {
        Name = "TF_Kube"
    }
}

resource "aws_route_table" "r" {
    vpc_id = "${aws_vpc.main.id}"
    route {
        cidr_block = "0.0.0.0/0"
        gateway_id = "${aws_internet_gateway.gw.id}"
    }

    depends_on = ["aws_internet_gateway.gw"]

    tags {
        Name = "TF_Kube"
    }
}

resource "aws_route_table_association" "publicA" {
    subnet_id = "${aws_subnet.publicA.id}"
    route_table_id = "${aws_route_table.r.id}"
}

resource "aws_route_table_association" "publicB" {
    subnet_id = "${aws_subnet.publicB.id}"
    route_table_id = "${aws_route_table.r.id}"
}


resource "aws_subnet" "publicA" {
    vpc_id = "${aws_vpc.main.id}"
    cidr_block = "10.0.100.0/24"
    availability_zone = "eu-west-1a"
  

    tags {
        Name = "TF_kube"
    }
}

resource "aws_subnet" "publicB" {
    vpc_id = "${aws_vpc.main.id}"
    cidr_block = "10.0.101.0/24"
    availability_zone = "eu-west-1b"
  

    tags {
        Name = "TF_kube"
    }
}


resource "aws_security_group" "kubernetes" {
  name = "kubernetes"
  description = "Allow inbound ssh traffic"
  vpc_id = "${aws_vpc.main.id}"

  ingress {
      from_port = 22
      to_port = 22
      protocol = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
      from_port = 0
      to_port = 0
      protocol = "-1"
      cidr_blocks = ["10.0.100.0/24"]
  }

  ingress {
      from_port = 0
      to_port = 0
      protocol = "-1"
      cidr_blocks = ["10.0.101.0/24"]
  }

  egress {
      from_port = 0
      to_port = 0
      protocol = "-1"
      cidr_blocks = ["0.0.0.0/0"]
  }

  tags {
    Name = "kubernetes"
  }
}

data "template_file" "master-userdata" {
    template = "${file("${var.master-userdata}")}"

    vars {
        k8stoken = "${var.k8stoken}"   			/* var d'ambiente per quando viene eseguita in init la user_data i.e. var rendered*/
    }
}

data "template_file" "worker-userdata" {
    template = "${file("${var.worker-userdata}")}"

    vars {
        k8stoken = "${var.k8stoken}"
        masterIP = "${aws_instance.k8s-master.private_ip}" /* var d'ambiente per quando viene eseguita in init la user_data*/
    }
}

resource "aws_instance" "k8s-master" {
  ami           = "ami-0d77397e"
  instance_type = "t2.micro"
  subnet_id = "${aws_subnet.publicA.id}"
  user_data = "${data.template_file.master-userdata.rendered}"
  key_name = "${aws_key_pair.ssh-key.key_name}"
  associate_public_ip_address = true                  /*  only to test*/
  vpc_security_group_ids = ["${aws_security_group.kubernetes.id}"]

   depends_on = ["aws_internet_gateway.gw"]

  tags {
      Name = "[TF] k8s-master"
  }
}



/*resource "aws_elb" "web-elb" {
  name = "my-load-balancer"
   COMMENTATO : l'ELB lo CREAMO A MANO*/
#  The same availability zone as our instances
   #availability_zones = ["${split(",", var.availability_zones)}"]
    /*security_groups = ["${aws_security_group.kubernetes.id}"]
    subnets = ["${aws_subnet.publicA.id}","${aws_subnet.publicB.id}"]

   listener {
     instance_port     = 31409          in effetti l'elb si potrebbe creare a priori e
     instance_protocol = "http"         quando crei il servizio inponi di usare la porta
     lb_port           = 80             che hai usato qui -- a questo punto gestisci tu il 
					range di porte NodePort
     lb_protocol       = "http"
   }
 
   health_check {
     healthy_threshold   = 2
     unhealthy_threshold = 2
     timeout             = 3
     target              = "HTTP:31409/"
    interval            = 30
  }


} COMMENTATO : l'ELB lo CREAMO A MANO*/

resource "aws_autoscaling_group" "web-asg" {
#  availability_zones   = ["${split(",", var.availability_zones)}"]
  name                 = "terraform-example-asg"
  max_size             = "${var.asg_max}"
  min_size             = "${var.asg_min}"
  health_check_grace_period = 300
  health_check_type = "EC2"
  desired_capacity     = "${var.asg_desired}"
  force_delete         = true
  launch_configuration = "${aws_launch_configuration.web-lc.name}"
#   load_balancers       = ["${aws_elb.web-elb.name}"]

  vpc_zone_identifier = ["${aws_subnet.publicA.id}","${aws_subnet.publicB.id}"]
  depends_on = ["aws_internet_gateway.gw"]
  tag {
    key                 = "Name"
    value               = "web-asg"
    propagate_at_launch = "true"
  }
}

resource "aws_launch_configuration" "web-lc" {
  name          = "terraform-example-lc"
  image_id      = "ami-0d77397e"
  instance_type = "t2.micro"
  # Security group
  security_groups = ["${aws_security_group.kubernetes.id}"]
  user_data       = "${data.template_file.worker-userdata.rendered}"
  key_name        = "${aws_key_pair.ssh-key.key_name}"
  lifecycle {
      create_before_destroy = true
  }
}

{% endhighlight %}
Vediamo ora come sono i file che vengono eseguiti durante l'init delle VM Master (template_file.master-userdata) e le VM worker lanciate dall'ASG (template_file.worker-userdata) rispettivamente.
{% highlight js %}

#!/bin/bash -v

curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl kubernetes-cni
curl -sSL https://get.docker.com/ | sh
systemctl start docker

kubeadm init --token=${k8stoken}

kubectl apply -f https://git.io/weave-kube
daemonset "weave-net" created
{%endhighlight %}
In questo file viene aggiornato il sistema APT  per poter scaricare l'ultima versione dei moduli kubernetes , quindi vengono installati tramite apt-get i moduli
 kubelet kubeadm kubectl kubernetes-cni , viene installato docker engine e viene  attivato , quindi viene eseguito il comando kubeadm init con token passato tramite la variabile di terraform . Si noti che in questo momento nessun processo kubernetes (siamo in fase di boot del VM) è attivo ..Kubeadm (tra le altre cose) crea dei manifest file (/etc/kubernetes/manifests sul master) che descrivono le risorse per creare i pod di "sistema" relativi alle altre risorse (controller, api-server..).Questi file vengono letti da kubectl al suo boot ( fatto tramite systemd e quindi da lui monitorato e d eventualmente ristartato) e vengono creati (come pod) tutti i componenti di kubernetes. Quindi kubectl può lanciare (tramite le Api dell'API-server che a questo punto è attivo e "connesso" in TLS -ricordate il Token?- a kubectl) il deamonset di rete Weave.

{% highlight js %}

#!/bin/bash -v

curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl kubernetes-cni
curl -sSL https://get.docker.com/ | sh
systemctl start docker

for i in {1..50}; do kubeadm join --token=${k8stoken} ${masterIP} && break || sleep 15; done
{%endhighlight %}
Con questo file (eseguito  sui nodi Worker) vengono installate le solite componenti di kubernetes (kubectl sui nodi non serve ma comunque..) e viene lanciato il comando kubeadm -join..

Dunque ora abbiamo il cluster funzionante e possiamo iniziare a creare deployments e servizi correlati. Eseguiamo i seguenti comandi :


```bash
$ kubectl run my-nginx --image=nginx --replicas=3 --port=80
```
Con questo comando abbiamo creato un deployment (astrazione di livello superiore del pod , contenente ad esempio il numer di replica dei pod ) contenente un solo container (contenente un'immagine di Nginx)

```bash
$ kubectl expose deployment my-nginx --port=80 --type=NodePort
```
Con questo comando "espongo" la porta 80 del pod su una porta dell'Host (all'interno di un range tra 30000-32767) ma equivale anche a creare  un "service" my-nginx.
```bash
$ kubectl describe service 
```
Infatti avendo lanciato il comando opra ottengo questa risposta:
Si noti il valore 30917 della proprietà NodePort.
{% highlight js %}
Name:                   my-nginx
Namespace:              default
Labels:                 run=my-nginx
Selector:               run=my-nginx
Type:                   NodePort
IP:                     100.67.89.151
Port:                   <unset> 80/TCP
NodePort:               <unset> 30917/TCP
Endpoints:              10.32.0.2:80,10.38.0.1:80,10.46.0.1:80
Session Affinity:       None
{%endhighlight %}
Quindi abbiamo 3 pod in listen agli indirizzi indicati dalla proprietà Endpoints(in effetti sui relativi nodi sono in listen sulle porte 30917). Ora non  ci resta che create un Elb AWS che è in in listen su internet sulla porta 80 e che fà loadbalancing verso le porte 30917  con i comandi :

La prima volta per creare l'elb e il listener sull'elb, e registrare le istanze attive nell'ASG (i worker di  kubernetes) all'elb.
{% highlight js %}
$ aws elb create-load-balancer --load-balancer-name my-load-balancer --listeners "Protocol=HTTP,LoadBalancerPort=80,InstanceProtocol=HTTP,InstancePort=30917" --subnets subnet-ae37fff5 subnet-3d847e01 --security-groups sg-8a6d78f0

$ aws elb register-instances-with-load-balancer --load-balancer-name my-load-balancer --instances  i-0ecc5edc78cb62c32 i-03b5a350bf596efea i-004a082ea6b8ce783

{% endhighlight  %} 
Le volte successive per aggiungere le porte dei servizi nuovi che hai aggiunto.
{% highlight js %}
$ aws elb create-load-balancer-listeners --load-balancer-name  my-load-balancer --listeners "Protocol=HTTP,LoadBalancerPort=81,InstanceProtocol=HTTP,InstancePort=xxxxx"
{% endhighlight  %}
**et nuv'là ...Les jeux sont faits** : esegui un http://<indirizzo DNS dell'elb> e vedrai la pagina di default di nginx. 



-----

