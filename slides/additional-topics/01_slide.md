# Additional Topics

Section Objectives:

* Questions from earlier sections
* Data bags
* Environments
* Lightweight Resources/Providers
* Knife Plugins

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribution Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Overview Section

This section is an overview of intermediate to advanced topics.

The order is structured by level of general interest.

We will not have time to cover these in great detail. See the
__Additional Resources__ list for more details.

We may not have time to cover them all entirely.

This section does not have a hands on exercise.

# Questions

Answer outstanding questions from earlier sections.

# Data Bags

Data bags are containers of "items" that are plain JSON data.

Items can contain any arbitrary key/value pairs, such as user
information, application setup parameters or DNS entries.

They are centrally available to recipes for processing.

# Data Bags

Data bags are indexed separately for search.

They are not tied specifically to roles or nodes.

Use data bags for storing information that is "infrastructure-wide".

# Creating Data Bags

Data bags go in the "data_bags" directory of the chef-repo.

Create a directory for the bag itself.

Put items in JSON files in the bag's directory.

We do not have a Ruby DSL for Data Bag items because they are free
form and can contain anything you want.

# Data Bag Use Case

As data bags can contain anything, they have unlimited uses. Some
common use cases we practice or have seen:

* User management
* Application information
* Hardware inventory
* Authentication credentials

# Encrypted Data Bags

Data bags can be encrypted with a secret key.

The key is something you generate on your local machine and do not
send to the Chef Server.

You still have to distribute the key file to systems that will need
access to encrypted data bags.

You probably already had a key distribution problem (think SSL).

# Example: Managing Users

    > mkdir data_bags/users
    > touch data_bags/users/USERNAME.json
    > $EDITOR data_bags/users/USERNAME.json
    {
      "id": "USERNAME"
      "groups": "sysadmin",
      "comment": "USERNAME",
      "uid": 2003,
      "shell": "/bin/bash",
      "ssh_keys": "ssh-rsa SSH_PUBLIC_KEY USERNAME@localhost"
    }
    > knife data bag create users
    > knife data bag from file users USERNAME.json

# Search Data Bags

Each bag gets a new index created.

Search can be done just like nodes or roles with knife or in a recipe.

    knife search BAG QUERY
    search(:bag, "QUERY")

Encrypted data bags cannot be searched because the contents are...
encrypted.

# Search Data Bags: Knife

    > knife search users "id:USERNAME"
    1 items found

    _rev:       6-c1c943b594f79bfae3ed043cca3a9de6
    chef_type:  data_bag_item
    data_bag:   users
    id:         USERNAME

Other keys in the item besides id can be used for the QUERY.

# Search Data Bags: Recipe


    user_list = search("users", "*:*")
    # or...

    search("users", "*:*") do |u|
      # something with u
    end

Query here is wildcard for all users, but this could be any query
relevant to information stored in the data bag items.

# Data Bags in Recipes

Besides using Search in recipes, data bags and data bag items can be loaded directly.

    data_bag()
    data_bag_item()

# Environments

Environments are used to enforce cookbook version constraints based on
logical infrastructure environments.

Different logical environments in the infrastructure may include
"production," "staging," "development" and so on.

Environments are assigned to nodes. They are managed as separate
entities on the Chef Server.

# Cookbook Version Constraints

Environments are most often used to set version constraints for
cookbooks.

This means that when Chef runs, it will only download the version
specified for the node's environment.

Version constraints can use operators to determine the version.

# Version Constraint Operators

    = Equal to
    > Greater than
    < Less than
    >= Greater than or equal to
    <= Less than or equal to

Advanced operators are available similar to those of RubyGems, see the
Version Constraint documentation.

# Example Environment

    name "production"
    description "Systems in production"
    cookbook_versions(
      "apache2" => "1.0.8",
      "mysql" = "1.0.0"
    )

# Environments: Nodes

The `Chef::Node` object has a method, `chef_environment` that can be
used to return the node's environment.

The default environment if none is assigned is `_default`. Version
constraints cannot be applied to the `_default` environment.

Searches can be constrained to the node's environment with a boolean
operator in the query.

    search(:node, "platform:ubuntu AND chef_environment:production")

# Environments: Configuration

The node gets its environment from the `Chef::Config[:environment]`
setting.

This can be passed to the `chef-client` command through the `-E`
option.

It can be set directly in the `/etc/chef/client.rb` configuration
file.

    environment "production"

# Environments: Knife

The knife `environment` sub-command can be used to work with
environments in the Chef Repository or on the Chef Server similar to
roles.

In the Chef Repository, environments are Ruby DSL or JSON files in the
`environments` directory.

    > knife environment --help
    > knife environment from file production.rb

Similar to roles, pick your file workflow between Ruby or JSON.

Knife bootstrap can set the node's environment at the first Chef run
with the `-E` option.

    > knife bootstrap IPADDRESS -E 'production'

# Environments: Recipes

The purpose of environments is to enforce versions of cookbooks for
particular systems. However, they can be used to constrain searches or
affect other behavior in recipes.

For example, search for other nodes that share *this* node's
environment:

    @@@ruby
    search(:node, "chef_environment:#{node.chef_environment}")

# LWRPs

LWRPs are "Lightweight Resources and Providers".

They are custom resources and providers that go in a cookbook.

They introduce a new DSL that is lighter weight than the full Ruby
classes Chef uses in the core library for resources and providers.

# LWRPs in a Cookbook

Lightweight resources and providers are automatically loaded from the
`resources` and `providers` directories from the cookbook.

They are named by joining the cookbook name and the file name. For
example, the `apt` cookbook contains:

    apt/resources
    -> repository.rb

    apt/providers
    -> repository.rb

The resource used in a recipe is `apt_repository`. If the Ruby file
name is `default.rb` then just the cookbook name is used.

# When to Use LWRPs

Use custom lightweight resources and providers when you want to create
a new resource that doesn't exist in Chef, or to modularize reuse of a
number of resources.

# Knife Plugins

Knife is a plugin-based system.

Plugins can be installed as RubyGems, or created in the
`.chef/plugins/knife` directory (in your home directory or in the Chef
Repository).

Opscode provides a number of plugins, such as interacting with
Cloud Computing providers like Amazon EC2.

* knife-ec2
* knife-rackspace
* knife-openstack
* knife-windows

# Summary

* Data bags
* Environments
* Lightweight Resources/Providers
* Knife Plugins

# Additional Resources

* http://wiki.opscode.com/display/chef/Data+Bags
* http://wiki.opscode.com/display/chef/Environments
* http://wiki.opscode.com/display/chef/Version+Constraints
*
  http://wiki.opscode.com/display/chef/Lightweight+Resources+and+Providers+%28LWRP%29
* http://wiki.opscode.com/display/chef/Knife+Plugins
