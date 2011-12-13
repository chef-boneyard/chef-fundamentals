# Introduction To Search

.notes These course materials are Copyright © 2010-2011 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribution Share Alike 3.0 License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Objectives

At completion of this unit you should...

* Understand what search indexes are.
* Know which search indexes are created by default
* Be able to search with knife.
* Be able to search in a recipe.

# Search

The Chef Server provides a powerful full text search engine.

The search engine is based on Apache SOLR.

The search query language is modified SOLR Lucene.

# Default Search Indexes

Four search indexes are created by default.

* node
* client
* role
* environment

# Search Indexes

When data bags are created, a search index is also created, and the index is the same name as the bag.

Data Bags are covered in detail later.

# Knife Search

Search returns the entire object found on the server, but the output is abbreviated.

* -m will display “normal” node attributes in the output.
* -l will display all node attributes in the output.
* -a ATTRIBUTE will display only the selected attribute.
* -i will display only the ID of the matching items.
* For nodes, -r will display the run list of the node.

# Knife Search

    knife search node "platform:ubuntu"
    knife search node "platform:ubuntu" -r
    knife search node "role:webserver"

# Query Syntax

Queries follow a basic pattern, with Knife:

    knife search INDEX “field:pattern”

In a recipe:

    search(:INDEX, “field:pattern”)

Search patterns can be an exact match, range match or a wildcard match.

# Query Syntax: Exact Match

    knife search node "role:webserver"
    knife search node "ipaddress:10.1.1.26"
    knife search node "kernel_machine:x86_64"

Search for a node with an attribute set to a particular value.

Sub-key attributes (`node[“kernel”][“machine”]`) are flattened with underscores.

# Query Syntax: Ranges

    knife search node "cpu_total:[4 TO 8]"
    knife search node "cpu_total:{4 TO 8}"

Range searches are inclusive with square brackets [].

Range searches are exclusive with curly braces {}.

# Query Syntax: Wildcards

    knife search node "hostname:app*"
    knife search node "fqdn:app1*.example.com"
    knife search node "hostname:*"
    knife search node "*:*"

Search for all nodes with hostname starting with “app”.

Search for all nodes with fqdn that starts with app1 and ends with .example.com

Search for all nodes that have a hostname at all.

Search for all nodes.

# Query Syntax: Boolean

    knife search node "(NOT cpu_total:1)"
    knife search node "role:webserver AND chef_environment:production"
    knife search node "cpu_total:2 OR kernel_machine:x86_64"

Search for all nodes except those with 1 CPU.

Search for all production web servers using the chef_environment.

Search for all nodes that have 2 CPUs or are 64bit.

# Recipe Search

Two kinds of search can be used in recipes.

* Aggregated
* Block

# Aggregate Search Results

    pool = search("node", "role:webserver")

Results can be aggregated and assigned to a local variable for later processing in a recipe.

The entire JSON object from the server is returned.

.notes that the entire object is returned in the results can be a
significant use of system memory

# Aggregate Results: Collect Values

    ip_addrs = search(:node, "role:webserver").map {|n| n["ipaddress"]}

Aggregated results can be passed a Ruby block with the “map” method
and the value of a single attribute can be extracted.

# Block Search

    @@@ruby
    search(:node, "role:webserver") do |match|
      puts match["ipaddress"]
    end
    
Provide a code block to process the results directly.

We’ll see this more with Data Bags.    

# Common Search Usage

Search is used for a wide variety of purposes in Chef. Common uses are:

* load balancers that need to pool a number of web servers.
* application servers that need to find the master database server.
* monitoring tools that need to find all the nodes of a particular
  type for custom monitoring.

# Knife Sub-commands

Knife has two built in subcommands that use search.

* knife status
* knife ssh

# Summary

You should now be able to...

* Describe what search indexes are.
* Describe which search indexes are created by default
* Search with knife.
* Search in a recipe.

# Lab Exercise

## Introduction To Search
