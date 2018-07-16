---
title: "JMX4R : JMX en Ruby"
date: 2010-11-24
categories:
 - JMX
 - Ruby 
 - Monitoring
---
_Lorsqu'on souhaite superviser et administrer une application Java pendant son exécution la solution qui s'impose est JMX. Malheureusement, écrire un client JMX n'est pas trivial. Une solution possible est l'utilisation de JMX4R une gemme Ruby conçue dans cette optique._

Dans le [billet précédent](/2010/11/13/jmxterm-introduction.html), j'ai présenté [JMXTerm](http://www.cyclopsgroup.org/projects/jmxterm/), un client JMX en ligne de commande très pratique pour accéder interactivement à JMX. Il a cependant le défaut d'être relativement difficile et lourd à automatiser. [JMX4R](https://github.com/jmesnil/jmx4r/) est une façade Ruby développée autour des API JMX par [Jeff Mesnil](http://jmesnil.net/weblog/). Elle permet de construire facilement des clients JMX ou des _Beans_ déployés dans un serveur. 

Un petit exemple valant mieux qu'un long discours, voilà un petit script qui correspond à une problématique que j'ai rencontrée lors du développement de [Sipatra](http://confluence.cipango.org/display/DOC/Sipatra). 

L'installation de `jmx4r` se fait simplement par la commande:

{{< highlight bash >}}
gem install jmx4r
{{< / highlight >}}

Le script est le suivant :

{{< highlight ruby >}}
require 'rubygems'
require 'jmx4r'
require 'csv'

app_name = "testapp"
app_bean = "org.cipango.sipapp:type=sipappcontext,name=#{app_name},*"

JMX::MBean.establish_connection :url =>
  "service:jmx:rmi:///jndi/rmi://localhost:3000/jmxrmi"
memory = JMX::MBean.find_by_name "java.lang:type=Memory"
nb_runtimes = JMX::MBean.find_all_by_name("org.jruby:type=Runtime,service=Config,*").size
smgr = JMX::MBean.find_all_by_name("org.cipango:type=sessionmanager,*").first
app = JMX::MBean.find_all_by_name(app_bean).first

File::open("data.csv", "a") do |file|
  CSV::Writer.generate(file) do |csv|
    csv << [
      Time::now.strftime("%Y-%m-%d %H:%M:%S"),
      memory.heap_memory_usage.used, 
      memory.heap_memory_usage.committed,
      memory.heap_memory_usage['max'],
      nb_runtimes,
      smgr.calls,
      app.nb_sessions
      ]
  end
end
{{< / highlight >}}

Pour lancer le script il suffit de taper :

{{< highlight bash >}}
jruby test.rb
{{< / highlight >}}

Pour collecter des données dans le temps le script peut être lancé à intervalles réguliers avec CRON. Les données sont ensuite affichables en utilisant [Gnuplot](http://www.gnuplot.info) ou simplement en chargeant le fichier dans Excel:

{{< highlight gnuplot >}}
set datafile separator ","
set xdata time
set timefmt "%Y-%m-%d %H:%M:%S"
plot "data.csv" using 1:2 index 0 title "Used Memory" with lines
{{< / highlight >}}

Il s'agit d'un exemple très simple mais il montre à quel point il est simple de collecter des données JMX avec JRuby même pour des cas d'usages très spécifiques qui ne sont pas toujours couverts par des outils plus génériques.
