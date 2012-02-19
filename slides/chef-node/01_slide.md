# Chef Node

Section Objectives:

* Properties of the Chef::Node object
* Common attributes from ohai
* Original location of node attributes
* Node's run list
* Methods available in recipes
* Use `knife` and `shef` to examine nodes

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribution Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Chef::Node

The first class citizen in Chef is the `Chef::Node`. The server stores
a copy of the node, and indexes it for search.

Ultimately, the node itself is the authority for its own
configuration. The `Chef::Node` object contains the run list and
attributes about the node itself.

An instance of `Chef::Node` is created for the Chef run, available in
recipes, templates and other cookbook components as "`node`".

# Node Name

Every node has a name which must be unique. By default, Chef will use
the node's fully-qualified domain name, since that is fairly safe.

You can override the name using `node_name` in the client config, or
by passing `-N NAME` to `chef-client`.

The `Chef::Node` object has a method, `name` to return the node's
name.

    @@@ruby
    # In some recipe...
    node.name # => "www1.example.com"

# Node Attributes

Node's have attributes, which are data about the node
itself. Attributes can come from a variety of sources in Chef. Most
commonly:

* Automatic (ohai)
* Cookbooks (attributes or recipes)
* Roles

Other locations are available, but they are out of scope for this course.

# Ohai: Automatic Attributes

Ohai determines many node attributes automatically for Chef. This is
the first thing that Chef does when it runs.

Values such as `platform`, `platform_version`, `ipaddress`, `hostname`
are determined by Ohai.

These are "automatic" attributes and cannot be overridden.

You can find the attributes of a node by running "ohai".

# Example Attributes

The exact attributes will vary by node based on the platform and the
available plugins. The following list is an abbreviated example from
an Ubuntu Linux node. Each of these may have values that are strings,
arrays or hashes.

* `chef_packages` - version information about Chef and Ohai
* `cpu` - information about installed processor(s)
* `current_user` - UID of the process
* `dmi` - if `dmi_decode` is installed, DMI information is available
* `domain` - the node's domain, e.g., the non-hostname portion of the `fqdn`.
* `etc` - users and groups on the system
* `filesystem` - mounted/available filesystems

# Example Attributes (continued)

* `fqdn` - fully qualified domain name, e.g. `hostname -f`
* `hostname` - first portion of `fqdn`.
* `ipaddress` - IP address of the interface with the default route
* `keys` - host SSH keys
* `languages` - information about programming languages
* `lsb` - Linux systems with LSB packages may have information here
* `macaddress` - the MAC address of the interface with the default route.
* `memory` - memory statistics/information
* `network` - information about all the installed network interfaces
* `ohai_time` - the Unix Epoch when Ohai was called

# Example Attributes (continued)

* `os` - the operating system kernel
* `os_version` - the OS kernel version
* `platform` - the platform/distribution
* `platform_version` - the version of the platform/distribution
* `recipes` - all the recipes appearing in the expanded run list
* `roles` - all the roles appearing in the expanded run list
* `uptime` - the system's uptime
* `uptime_seconds` - uptime in seconds, useful for calculation
* `virtualization` - if the node is virtualized, additional
  information is avaialble

# Attribute Values

Node attributes can be used in recipes and templates.

Access node attributes like Ruby hash keys.

    @@@ruby
    node["fqdn"] # => "www1.example.com"
    node["chef_packages]["chef"]["version"] # => "0.10.8"

# Attribute Conditionals

Attributes can be used to set up conditional checks using Ruby "if", "unless" and "case" statements.

This is commonly done to match a node's platform.

# Attribute Conditionals

    @@@ruby
    case node["platform"]
    when "debian", "ubuntu"
      package "apache2-doc" do
        action :install
      end
    end

    package "apache2" do
      case node["platform"]
      when "centos","redhat"
        package_name "httpd"
      end
      action :install
    end

# platform? Method

The `platform?` method will return `true` if one of the parameters
matches the `node["platform"]` attribute. The method takes a comma
separated list of platform names (lower-case).

    @@@ruby
    # on an ubuntu system:
    platform?("ubuntu") # => true

This isn't a `Chef::Node` method, but worth mentioning here as an
alternate way to check the node's platform.

# Attributes in Strings

Use Ruby's #{} string interpolation to embed an attribute in a
string. For example, we can set an attribute where the Apache
directory is, and write a configuration file there dynamically.

    @@@ruby
    template "#{node["apache"]["dir"]}/ports.conf" do
      # ...
    end

# Variables in a Template

Templates can take a parameter attribute called `variables`. These
variables become available to the template. This allows us to shortcut
typing longer attributes.

`variables` is a hash, each key must be a symbol (starting with `:`), and the value can be
anything. Here we use a node attribute:

    @@@ruby
    template "#{node["apache"]["dir"]}/ports.conf" do
      # ...
      variables :apache_listen_ports => node["apache"]["listen_ports"]
    end

# Variables in a Template

When referring to the variables in the actual template, we must use a
class instance variable (starting with `@`).

    #This file generated via template by Chef.
    <% @apache_listen_ports.each do |port| -%>
    Listen <%= port %>
    NameVirtualHost *:<%= port %>

    <% end -%>

# Node Attributes in Templates

We can also use node attributes directly.

    ServerRoot "<%= node["apache"]["dir"] %>"
    # ...
    <% if node["platform"] == "debian" || node["platform"] == "ubuntu" -%>

Node attributes used in a template can come from the cookbook itself,
or one auto detected by chef such as "platform".

.notes We do *not* need to use the `@` symbol on the node.

# Cookbook Attributes

Node attributes can come from cookbooks.

Use an attributes file in a cookbook to set defaults.

These default values are to ensure that some value is set.

All the attributes files in all cookbooks on the node are loaded and
all attributes are applied to the node object.

# cookbooks/apache2/attributes/default.rb

Use the "default" keyword to set the attributes.

The node object is implied, `default` is a method of Chef::Node.

    default["apache"]["listen_ports"] = [ "80","443" ]
    default["apache"]["contact"] = "ops@example.com"
    default["apache"]["timeout"] = 300

# Recipe Attributes

Setting attributes is the same as calling a method on the node object,
and this can also be done in a recipe.

Do this when you want to calculate a value for the attribute.

Calculated values can come from Ruby expressions, searches, external
data sources and more.

# Attributes in Recipes

When setting attributes in recipes, use the node.set method.

This allows additional overrides to be used, but has higher priority
than cookbook attributes files "default" and "set" and role
"`default_attributes`" values.

    @@@ruby
    node.set['unicorn']['workers'] = node['cpu']['total'].to_i * 4

# Role Attributes

Node attributes can be set in roles using "`default_attributes`".

This overrides the attributes set using "default" in the cookbook's attributes file.

# roles/webserver.rb

    name "webserver"
    description "Systems that serve HTTP traffic"
    run_list(
      "recipe[apache2]",
      "recipe[apache2::mod_ssl]"
    )
    default_attributes(
      "apache" => {
        "listen_ports" => [ 80, 443, 8080 ]
      }
    )

# Node attribute? Method

The `Chef::Node#attribute?` method can be used to check if the node
has a given attribute. Use it with a top-level key to find sub-keys.

    @@@ruby
    node.attribute?("fqdn") # => true
    node.attribute?("aaaaa") # => false
    node['chef_packages'].attribute?("chef") # => true

# Run List

The run list is an array of roles and/or recipes to apply on the node.

The run list is expanded during run time to determine the cookbooks to
download (by recipe name).

The expanded list of roles are stored in the attribute
`node["roles"]`.

The expanded list of recipes are stored in the attribute
`node["recipes"]`.

Recipes that are included via `include_recipe` method are *not* expanded.

# Run List

The run list is an array, which means it is ordered by numeric index
of the elements it contains.

The expanded run list's recipes are in the order they appear in run
list elements (node itself or in roles).

Recipes are applied on the node in the order specified.

# Node role? method

The `Chef::Node#role?` method will check if the node's run list has
the specified role.

    @@@ruby
    node.role?("base")

# Node recipe? method

Use the `Chef::Node#recipe?` method to check if the node has a given
recipe or not.

First, the run list is consulted for the named recipe. Second, the
node's run state is checked for recipes Chef has seen via `include_recipe`.

    @@@ruby
    node.recipe?("webserver")

# Node Run State

When Chef runs, it stores ephemeral information about the node in the
`run_state`. This is per-run data that is not persisted to the Chef
Server.

`run_state` is a method of the node object that returns its contents.

An example of content in the run state is the list of recipes that
Chef has seen as they are loaded.

# Chef Environment

Every node has an environment. If you do not explicitly set one,
`_default` is used.

Chef environments allow you to set cookbook version constraints to
nodes in a particular environment, e.g. "production".

Retrieve the value of the node's environment with the
`Chef::Node#chef_environment` method.

    @@@ruby
    node.chef_environment # => "_default"

We will discuss Chef Environments later.

# Examining Chef::Node

Use `knife node list` to list all the node objects on the Chef Server.

    > knife node list

Use `knife node show` to display a node object on the Chef Server.

    > knife node show i-2b58ad4a
    Node Name:   i-2b58ad4a
    Environment: _default
    FQDN:        domU-12-31-39-07-D5-65.compute-1.internal
    IP:          107.20.14.15
    Run List:    role[base], role[webserver]
    Roles:       base, webserver
    Recipes:     apt, chef-client, webserver
    Platform:    ubuntu 10.04

# Examining Chef::Node

`knife node show` has several options to modify the output of the node
object. We can show only the run list:

    > knife node show i-2b58ad4a -r
    run_list:
        role[base]
        role[webserver]

The Chef environment:

    > knife node show -E i-2b58ad4a
    chef_environment:  _default

Or a specific attribute:

    > knife node show i-2b58ad4a -a ec2.public_hostname
    ec2.public_hostname:  ec2-107-20-14-15.compute-1.amazonaws.com

# Examining Chef::Node

The `Chef::Node` is actually a JSON object but the format returned is
a bit nicer for humans to read. Show the data in another format with
`-F` and specify a format. Valid alternate formats are JSON ("j"),
YAML ("y") or Text (default, or "t")

    > knife node show i-2b58ad4a -Fj
    > knife node show i-2b58ad4a -r -Fj
    > knife node show i-2b58ad4a -a ec2 -Fj

# Examining Chef::Node in Shef

We can use `shef` to examine properties of the `Chef::Node`
object. Similar to `chef-client`, `shef` loads a `node` object on
startup, including attributes from Ohai.

    > shef
    loading configuration: none (standalone shef session)
    Session type: standalone
    ...
    Ohai2u jtimberman@localhost.localdomain!
    chef >

# Chef::Node in Shef

Show the `Chef::Node` object itself:

    chef > node
     => <Chef::Node:0xb75b9c @name="localhost.localdomain">

Show the node's name:

    chef > node.name
     => "localhost.localdomain

Show the node's environment and run list:

    chef > node.chef_environment
     => "_default"
    chef > node.run_list
     =>

# Chef::Node in Shef

Show some attributes about the node:

    chef > node['ipaddress']
     => "10.0.0.100"
    chef > node['platform']
     => "ubuntu"
    chef > node['platform_version']
     => "11.10"

# Summary

* Properties of the Chef::Node object
* Common attributes from ohai
* Original location of node attributes
* Node's run list
* Methods available in recipes
* Use `knife` and `shef` to examine nodes

# Questions

* In a recipe, how do you get the node's name?
* What are some commonly available attributes?
* Where can attributes come from?
* What kind of attributes should be set in a cookbook attributes file?
* When should attributes be set from a recipe?
* Are recipes included with `include_recipe` in the `node["recipes"]` attribute?
* What order are recipes applied to the node by Chef?
* What is the node's run state? Is it saved on the server?

# Additional Resources

* http://wiki.opscode.com/display/chef/Nodes
* http://wiki.opscode.com/display/chef/Attributes
* http://wiki.opscode.com/display/chef/Environments

# Lab Exercise

Chef Node

* View the node object on the Chef Server with knife
* Create new attributes on the node with a cookbook and role.
* Use node attributes in a conditional in the recipe.
