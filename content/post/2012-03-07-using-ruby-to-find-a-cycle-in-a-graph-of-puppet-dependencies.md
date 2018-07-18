---
title: "Using Ruby to find a cycle in a graph of Puppet dependencies"
date: 2012-03-07T22:08:00
comments: true
categories:
  - Ruby
  - QuickTips
  - Graph
  - Puppet
---
Today, I had a small issue while using [Puppet](http://puppetlabs.com/). To make it quick: I got a circular dependency in my Puppet recipes and Puppet failed with a verbose but not so helpful message:

{{< highlight none >}}
err: Could not apply complete catalog: Found dependency cycles in the following relationships:
User[root] => File[/usr/share/locale/locale.alias], Package[python-setuptools] ...
# INSERT TONS of other dependencies here
... try using the '--graph' option and open the '.dot' files in OmniGraffle or GraphViz
{{< / highlight >}}

Of course, I tried the '--graph' option but, due to it's size, the generated diagram was anything but readable.

I decided to script my way out of this tangle and, with help from [this forum entry](http://www.ruby-forum.com/topic/218281), I was able to quickly piece together a Ruby script that detects cycles in a graph:
{{< highlight ruby >}}
require 'rubygems'
require 'rgl/connected_components'
require 'rgl/adjacency'

graph = RGL::DirectedAdjacencyGraph[
  *File.open("data.txt").read.split(/,[ \n]+/).each.map do |line|
    line.strip.chomp(",").split(' => ')
  end.flatten]

inv_comp_map = {}
graph.strongly_connected_components.comp_map.each do |v, n|
  (inv_comp_map[n] ||= []) << v
end
puts inv_comp_map.values.delete_if { |scc| scc.size == 1 }.inspect
{{< / highlight >}}

The script reads a {{data.txt}} file containing lines of the form:
{{< highlight none >}}
User[dom] => File[foo],
User[bar] => User[dom],
File[foo] => User[bar],
User[foobar] => File[foo]
{{< / highlight >}}

and outputs :
{{< highlight none >}}
[["User[dom]", "File[foo]", "User[bar]"]]
{{< / highlight >}}

To use the script:
{{< highlight none >}}
gem install rgl
cat > detect_cycle.rb # copy and paste the script
cat > data.txt        # copy and paste Puppet's dependency information
ruby detect_cycle.rb
{{< / highlight >}}

Rejoice ! ;-)
