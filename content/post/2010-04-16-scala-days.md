
---
comments: true
title: "Scala Days 2010"
date: 2010-04-16
categories:
 - Scala
 - Programmation 
 - DSL
draft: false
aliases:
    - /2010/04/16/scala-days-2010.html
---
_Les 15 et 16 avril 2010 j'ai eu la chance d'assister aux Scala Days à Lausanne (Suisse). Il s'agissait d'une conférence autour du langage de programmation multi-paradigme Scala, ses applications et son futur._

Scala est un langage de programmation conçu à l'École Polytechnique Fédérale de Lausanne. Il intègre les paradigmes de programmation fonctionnelle avec ceux de la programmation orientée objet. Son typage est statique mais le système d'inférence de types du compilateur permet de se passer, dans un grand nombre de cas, de la lourdeur habituellement imposée par cette catégorie de langages. Scala est compilé en bytecode pour la machine virtuelle Java ou .Net ce qui lui permet de s'interfacer de manière transparente avec l'ensemble des librairies existantes pour ces plateformes. Plus de détails sur le langage sont disponibles sur son [site officiel](http://www.scala-lang.org/).

Jeudi 15
--------

La conférence a démarré par l'exposé liminaire de Martin Odersky. Considéré comme le père de Scala, Martin a débuté par un bref historique avant de continuer par une présentation détaillée de la version 2.8.RC1 qui vient juste d'être livrée. Cette version, contient un nombre important de modifications car, au vu du succès du langage, c'était pour l'équipe qui développe le langage, la dernière occasion d'y intégrer facilement des modifications qui cassent la compatibilité amont. 

N'ayant pas ma machine à cloner sous la main, je n'ai pu assister qu'à l'une des deux sessions. Mais les deux sujets qui semblent revenir le plus souvent sont la programmation parallèle et les DSL (langages dédiés à un domaine). 

### Programmation concurrente

Plusieurs présentations ont traité de la programmation parallèle :

 * [CCSTM](http://days2010.scala-lang.org/node/138/148) : une librairie pour la gestion _mémoire transactionnelle_;
 * Une [librairie de collections concurrentes](http://days2010.scala-lang.org/node/138/140);
 * Le [traitement parallèle d'évènements typés](http://days2010.scala-lang.org/node/138/149);
 * [Structures de données parallèles](http://days2010.scala-lang.org/node/138/150).

### DSL

Le sujet des DSL est également à l'ordre du jour car, comme [Ruby](http://www.ruby-lang.org), Scala se prête bien à l'écriture de DSL embarqués (écrits en réutilisant la syntaxe d'un langage existant et non en créant une syntaxe spécifique). Scala est un langage typé, mais son compilateur est capable d'inférer un grand nombre d'informations sur les types sans que le développeur ait a les expliciter dans le code. Couplée à sa grande souplesse syntaxique, l'inférence de types permet de produire des DSL très expressifs et pratiquement sans sucre syntaxique ajouté. La concision du code atteinte par Scala est telle qu'on a souvent l'impression d'écrire du code dans un langage dynamique comme Ruby. En revanche, on profite du compilateur Scala pour détecter un nombre plus important d'erreurs présentes dans le programme écrit en DSL ce qui aide beaucoup le développeur pour détecter et corriger les erreurs rapidement.

Vendredi 16
-----------

La journée a démarré sur ces mêmes sujets par l'exposé liminaire de Kunle Olukotun, directeur du [Pervasive Parallelism Laboratory](http://ppl.stanford.edu/wiki/index.php/Pervasive_Parallelism_Laboratory) (PPL) de l'université de Stanford, sur l'utilisation des DSL embarqués écrits en Scala, pour développer des programmes exécutables sur des architectures parallèles hétérogènes. Actuellement, les développeurs sont obligés d'investir un effort important en terme d'adaptation de leurs programmes à ces architectures spécifiques. L'objectif du PPL est de fournir aux développeurs des DSL qui abstraient les spécificités et la complexité du calcul parallèle dans un domaine donné. Le développeur peut alors se focaliser sur l'expression de son problème, le DSL prenant en charge l'optimisation et la génération du code spécifique à l'architecture cible. 
 
Le point clé de cette journée était l'écosystème des outils qui se développent autour de Scala :

 *  [SBT](http://days2010.scala-lang.org/node/138/165) : un moteur de production de livrables pour des projets Scala ou Java;
 *  La nouvelle [IDE eclipse](http://days2010.scala-lang.org/node/138/163) pour Scala 2.8;
 *  [Scala Modules](http://days2010.scala-lang.org/node/138/166) : un DSL pour faciliter l'intégration dans un environnement OSGi.

La [dernière présentation](http://days2010.scala-lang.org/node/138/169) de la journée a tenté, sur le ton de l'humour, de nous apprendre comment faire entrer Scala par la petite porte au sein de l'entreprise. David Copeland d'OPower nous à présenté la manière dont il avait fait entrer Scala, avec succès, dans son entreprise jusque là focalisée sur Java.
 
Conclusion
----------

L'impression que je retire de cette conférence est que Scala est arrivé à maturité. Son écosystème d'outils et de librairies n'est qu'au début de son développement mais ceux déjà disponibles laissent entrevoir d'énormes possibilités en terme de productivité pour le développeur. Le sentiment général parmi les conférenciers et les participants semble être que la seule chose qui manque encore à Scala pour prendre son essor dans le monde de l'entreprise est le support d'un grand nom de l'informatique. Cette première conférence et l'annonce de la création d'une «Fondation Scala» est un grand pas dans cette direction.

A titre personnel, la conférence m'a permis d'échanger autour de Scala. Et force est de constater qu'au vu de la maturité du langage et après un peu de temps passé à comprendre les nouveaux concepts qu'il introduit, il me semble de plus en plus intéressant pour développer des applications à destination de la plateforme Java (Machine virtuelle Java). Non seulement, à cause du gain de productivité qu'il apporte, mais également et surtout parce qu'il permet une approche de plus haut niveau qui réduit le temps passé à coder au profit du temps passé à améliorer le design.
 
Mise à jour: L'un des partenaires de la conférence, OPower, à sponsorisé la mise en ligne de [vidéos de la conférences](http://days2010.scala-lang.org/node/136).
