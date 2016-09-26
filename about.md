---
layout: page
title: About
---

<p class="message">
Hi, Con questa pagina mi presento e descrivo brevemente le attività che hanno portato alla realizzazione di questo sito. 
Sono un consulente in Tecnologie innovative, che parte da un passato puramente tecnico (progettazione software in ambito Telecomunicazioni/automazione industriale) fino ad arrivare ad avere esperienza ed attitudine commerciale . Aspetto comune a tutta la mia vita professionale è comunque stata la curiosità per le cose nuove. Questo è il motivo che mi spinge oggi a pubblicare queste miei pensieri...
</p>
Il sito ( di tipo statico) è stato generato utilizzando il sistema [Jekyll](http://jekyllrb.com) chepermette di definire siti multipagnine con stili diversi. Ovviamente il layout "principe"  è quello   del Blog, il che lo rende particolarmente adatto anche alle realizzazione di siti personali o attinenti ad un progetto.

Come funziona Jekyll ? Il sistema opera su una directory strutturata in modo predefinito ( fie pubblici , file di css , file di immagini , file index.html ...) contenente file di testo di vario formato : [Markdown](https://daringfireball.net/projects/markdown/) language o html e tramite l'interpretazione di [Liquid](https://github.com/Shopify/liquid/wiki) Tag  produce un  completo Web statico pronto per essere pubblicato .

La caratteristica di questo heneratore di  Web Statici è che fortemente integrato (e supportato) da GitHub  nel suo prodotto GitHub Pages , tanto che è possibile pubblicare il sito non  caricando il prodotto generato (file Html) sui repository (come faremmo per Amazon S3 per esempio ) ma tramite commit e push su master branch  del sorgente del sito. Tutto questo  in modo FREE fino ad una dimensione massima deel sito prodotto di 1GB e ad un massimo di 100000 richieste http al mese.


## Setup

Riassumendo :

* Costruito con [Jekyll](http://jekyllrb.com)
* Sviluppato e soggetto a controllo di Versione secondo Git [GitHub Pages](https://pages.github.com)
* Hosted in modo gratuito su GitHub Pages
* Sviluppato ed installato sia localmente per testing e su GitPages in meno di 2 giorni

P.S. Altra cosa che ho appreso  è che i siti statici  hanno delle limitazioni e delle best practises :* Non possono ricevere i dati da una Form ad esempio
* è bene che non contengano informazione sensibile ( come password o indirizzi e-mail)
Per ovviare a questi problemi vengono utilizzati servizi esterni : 

* [FormSpree](https://formspree.io/)
* [Mailhide](https://www.google.com/recaptcha/mailhide/d)

L'integrazione con questi servizi potrebbe essere oggetto del prossimo post.

Thanks for reading!
