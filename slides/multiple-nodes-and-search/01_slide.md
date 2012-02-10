# Multiple Nodes And Search

Section Objectives:

* Chef search indexes
* Search using knife
* Search in a recipe
* Knife bootstrap and ssh
* Integrating systems with search

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribution Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

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

When data bags are created, a search index is also created, and the
index is the same name as the bag.

Data Bags are covered in detail later.

# Knife Search

Search returns the entire object found on the server, but the output
is abbreviated.

* -m will display "normal" node attributes in the output.
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

    knife search INDEX "field:pattern"

In a recipe:

    search(:INDEX, "field:pattern")

Search patterns can be an exact match, range match or a wildcard match.

# Query Syntax: Exact Match

    knife search node "role:webserver"
    knife search node "ipaddress:10.1.1.26"
    knife search node "kernel_machine:x86_64"

Search for a node with an attribute set to a particular value.

Sub-key attributes (`node["kernel"]["machine"]`) are flattened with underscores.

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

Search for all nodes with hostname starting with "app".

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

Results can be aggregated and assigned to a local variable for later
processing in a recipe.

The entire JSON object from the server is returned.

.notes that the entire object is returned in the results can be a
significant use of system memory

# Aggregate Results: Collect Values

    ip_addrs = search(:node, "role:webserver").map {|n| n["ipaddress"]}

Aggregated results can be passed a Ruby block with the "map" method
and the value of a single attribute can be extracted.

# Block Search

    @@@ruby
    search(:node, "role:webserver") do |match|
      puts match["ipaddress"]
    end

Provide a code block to process the results directly.

We'll see this more with Data Bags.

# Common Search Usage

Search is used for a wide variety of purposes in Chef. Common uses
are:

* load balancers that need to pool a number of web servers.
* application servers that need to find the master database server.
* monitoring tools that need to find all the nodes of a particular
  type for custom monitoring.

# Knife Sub-commands

Knife has two built in subcommands that use search.

* knife status
* knife ssh

# Bootstrapping Chef

The process of installing and initially running Chef is referred to as
"Bootstrapping Chef". The general steps are:

* Update the local system's package cache (OS dependent).
* Install Ruby and RubyGems.
* Install Chef as a RubyGem.
* Copy the validation certificate file.
* Create a Chef client configuration pointing at the right Chef
  Server.
* Create a JSON file with the node's initial Run List.
* Run Chef using the JSON file.

# Installing and Running Chef

    @@@sh
    curl http://opscode.com/chef/installsh | sudo bash
    mkdir /etc/chef
    vi /etc/chef/validation.pem
    vi /etc/chef/client.rb
    vi /etc/chef/first-boot.json

    sudo chef-client -j /etc/chef/first-boot.json

# Bootstrap Sub-command

Knife bootstrap will run a single script on a target host. It uses
`knife ssh` under the covers. Instead of a query it uses a manual list.

Uses "bootstrap templates" by name. Several come with Chef.

These are typically used to install Chef on the remote system and tell
Chef to run with a particular run list.

# Knife Bootstrap Templates

The templates that come with Chef:

* archlinux-gems
* centos5-gems
* chef-full
* fedora13-gems
* ubuntu10.04-apt
* ubuntu10.04-gems

The
[Default Bootstrap Template](https://github.com/opscode/chef/raw/master/chef/lib/chef/knife/bootstrap/chef-full.erb)
(chef-full, as of Chef 0.10.10+) will install the Chef Full Stack package that we
have been using. Other bootstrap templates are available.

# Knife Bootstrap Customization

The bootstrap "templates" are really just shell scripts.

You can create your own bootstrap templates.

Use `-d` to specify the distribution style.

    ./.chef/bootstrap/DISTRO.erb
    $HOME/.chef/bootstrap/DISTRO.erb

# Knife Bootstrap

    (
    cat <<'EOP'
    <%= config_content %>
    EOP
    ) > /etc/chef/client.rb

Bootstrap scripts are processed as ERB templates.

"`config_content`" comes from Bootstrap Context, which is a shortcut to
using the Chef Config values from Knife.

# Knife Bootstrap Usage

Use the `-d DISTRO` option to specify a different bootstrap template.

    knife bootstrap IPADDRESS -r 'role[webserver]' -x ubuntu -i ~/.ssh/ec2.pem

    knife bootstrap IPADDRESS -r 'role[webserver]' -x ubuntu -i ~/.ssh/ec2.pem -d ubuntu10.04-mine-gems

# Integrate a New Node

Integrating a new node into the infrastructure is easy with knife
bootstrap.

We'll add a load balancer that will sit in front of the web server.

# Add Load Balancer

Presumably, the new system has been provisioned and has an OS
installed on it, ready to SSH and be managed with Knife Bootstrap and
Chef.

During the exercise, the instructor will provide a second remote
target system that will be used for the load balancer system.

# Load Balancer Cookbook

The load balancer will need a cookbook to install and configure the
load balancer software.

Haproxy is the load balancer software.

The cookbook is available from the Community Site.

A new role will be created that will be applied with the "knife
bootstrap" command.

# Load Balancer Role

    @@@ruby
    name "lb"
    description "web server load balancer"
    run_list(
      "recipe[haproxy::app_lb]"
    )

The `haproxy::app_lb` recipe will perform a search for web servers.

# Bootstrap Load Balancer

    knife bootstrap IPADDRESS -r 'role[lb]' -x ubuntu --sudo

IPADDRESS is the IP of the target system. We can use FQDN here.

Default Ubuntu installs enable the "ubuntu" user to sudo.

# Haproxy Uses Search

The `app_lb` recipe in the haproxy cookbook uses search. The default
recipe we looked at in the last section does not.

Any new nodes with "`role[webserver]`" will be automatically detected by
the load balancer whenever it runs Chef.

The role to use is set by an attribute in the haproxy cookbook. It can
be modified via a role.

    node['haproxy']['app_server_role']

.notes When growing the infrastructure, new web servers can be created
with the "webserver" role. Then run chef-client on the load balancer
node and the new servers will be automatically added to the pool for
haproxy.

# Summary

* Chef search indexes
* Search using knife
* Search in a recipe
* Knife bootstrap and ssh
* Integrating systems with search

# Questions

* What is the search query language used by Chef?
* What are two search indexes are created by default?
* How can node searches display only the run list of the results?
* What are two other knife subcommands that use search? Which is the
  basis of `knife bootstrap`?
* What tasks are performed by `knife bootstrap`?
* How does `knife bootstrap` know what system to connect?
* How does the haproxy cookbook load balance the web servers? What
  recipe is used? How is it different from the default?

# Additional Resources

* http://wiki.opscode.com/display/chef/Search
* http://wiki.opscode.com/display/chef/Knife+Bootstrap
* http://wiki.opscode.com/display/chef/Knife+Built+In+Subcommands

# Lab Exercise

Multiple Nodes And Search

* Add load balancer role with haproxy recipe
* Bootstrap a new node with load balancer role
* Use knife ssh to rerun chef client
