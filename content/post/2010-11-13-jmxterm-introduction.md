---
title: "JMXTerm : JMX en ligne de commande"
date: 2010-11-13
categories:
 - JMX
 - Monitoring
---
_Il est parfois utile de pouvoir consulter les informations de supervision d'une machine virtuelle Java qui s'exécute sur un serveur distant. Malheureusement, l'agent JMX n'est pas toujours configuré pour un accès distant ou cet accès distant est difficile du fait de la topologie réseau._

Si le JDK est installé sur la machine distante, il est toujours possible de se connecter au serveur en faisant suivre une session X et de lancer un _jconsole_ directement sur le serveur :

{{< highlight none >}}
monposte:~ dom$ ssh -X mon-serveur
Warning: untrusted X11 forwarding setup failed: xauth key data not generated
Warning: No xauth data; using fake authentication data for X11 forwarding.
Linux mon-serveur 2.6.32-1-686 #1 SMP 

root@mon-serveur ~ $ jconsole
{{< / highlight >}}

Ceci dit, lancer une session X à travers une session SSH, elle-même prolongée dans une autre session SSH, n'est pas très performant d'un point de vue réseau. Et _jconsole_ est une application Swing ce qui n'arrange en rien le taux de rafraîchissement. Cette solution est fonctionnelle mais, à moins d'avoir beaucoup de temps et de patience, elle n'est pas pratique du tout.

# JMXTerm à la rescousse

C'est dans une situation de ce type que j'ai découvert [JMXTerm](http://www.cyclopsgroup.org/projects/jmxterm/) un client pour agents [JMX](http://download.oracle.com/javase/1.5.0/docs/guide/management/agent.html) en ligne de commande.

Un exemple valant mieux qu'un long discours, la session ci-dessous correspond à la connexion à une machine virtuelle dont le PID est 25152 et à l'affichage des informations relatives à la zone de mémoire du tas :

{{< highlight none >}}
monposte:jmxterm $ jps
25188 Jps
25152 jar
monposte:jmxterm $ java -jar jmxterm-1.0-alpha-4-uber.jar 25152
Welcome to JMX terminal. Type "help" for available commands.
$>open 25152
#Connection to 25152 is opened
$>get -b java.lang:type=Memory HeapMemoryUsage
#mbean = java.lang:type=Memory:
HeapMemoryUsage = { 
  committed = 85000192;
  init = 0;
  max = 129957888;
  used = 20722240;
 };
{{< / highlight >}}

JMXTerm permet également de lister les _Beans_ disponibles :

{{< highlight none >}}
$>beans -d java.lang
#domain = java.lang:
java.lang:name=CMS Old Gen,type=MemoryPool
java.lang:name=CMS Perm Gen,type=MemoryPool
java.lang:name=Code Cache,type=MemoryPool
java.lang:name=CodeCacheManager,type=MemoryManager
java.lang:name=ConcurrentMarkSweep,type=GarbageCollector
java.lang:name=Par Eden Space,type=MemoryPool
java.lang:name=Par Survivor Space,type=MemoryPool
java.lang:name=ParNew,type=GarbageCollector
java.lang:type=ClassLoading
java.lang:type=Compilation
java.lang:type=Memory
java.lang:type=OperatingSystem
java.lang:type=Runtime
java.lang:type=Threading
{{< / highlight >}}

ou d'exécuter une action sur un _Bean_, comme par exemple demander une exécution du ramasse-miettes:

{{< highlight none >}}
$>run -b java.lang:type=Memory gc
#calling operation gc of mbean java.lang:type=Memory
#operation returns: 
null
{{< / highlight >}}

ou un vidage de la zone de mémoire du tas dans un fichier pour débogage :

{{< highlight none >}}
$>run -b com.sun.management:type=HotSpotDiagnostic dumpHeap /tmp/heap.bin true
#calling operation dumpHeap of mbean com.sun.management:type=HotSpotDiagnostic
#operation returns: 
null
{{< / highlight >}}

# Automatisation

Il est parfois nécessaire d'automatiser une tâche depuis un script. Par exemple, l'extraction du nombre de fils d'exécution utilisés par une instance de Jetty peut s'écrire :

{{< highlight none >}}
root@mon-serveur:~# echo get -s -b \
  org.mortbay.thread:id=0,type=boundedthreadpool threads | \
  java -jar jmxterm-1.0-alpha-4-uber.jar -v silent  -n -l 25152
10
{{< / highlight >}}

Cette dernière utilisation n'est cependant pas très performante car une machine virtuelle Java est lancée pour chaque interaction avec le serveur. Elle n'est pas non plus très élégante. Si l'automatisation devient plus complexe et qu'un environnement JRuby est disponible il existe une alternative à JMXTerm que j'aborderai dans un prochain billet.
