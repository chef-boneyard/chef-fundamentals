# Roles

Section Objectives:

* Understand the components of a role
* Create roles with the Ruby DSL
* View roles on the Chef Server
* Apply roles to nodes

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribute Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Components of a Role

Run list

* recipes
* roles

Attributes applied to the nodes that have this role.

* default
* override

# Creating Roles

The `roles` directory in the chef-repo contains the roles for the infrastructure.

Write roles using a Ruby DSL.

    $EDITOR roles/base.rb

Upload the roles to the Chef Server. Knife will automatically look for
the specified file in the `roles` directory.

    knife role from file base.rb

Roles are converted to JSON on the server.

# Ruby or JSON

Chef supports Ruby DSL or JSON for role files in chef-repo.

* Ruby DSL roles require less syntax.
* Ruby is converted to JSON by Knife when uploading.
* Chef Server stores roles as JSON.
* `knife role show` displays JSON and can be redirected to a file.

.notes We commonly use the Ruby DSL for creating roles, because it is
a bit lighter syntax, and allows flexibility of using Ruby idioms.

# Building Roles

What kind of roles do we need?

* `base` role.
* per-service roles.
* platform roles.

# Base Role

We use a role called `base` that gets applied to every system in the infrastructure.

This contains the basics that all systems should have.

# roles/base.rb

    @@@ruby
    name "base"
    description "Base role applied to all nodes."

    run_list(
      "recipe[chef-client]",
      "recipe[fail2ban]",
      "recipe[users]"
    )

    default_attributes(
      "chef_client" => {
        "server_url" => "https://api.opscode.com/organizations/ORGNAME",
        "validation_client_name" => "ORGNAME-validator"
      },
    )

# Per-service Roles

In a service oriented architecture, each different service should have
its own role with the recipes that determine how to fulfill that
service.

For example, it is common in a web-application architecture to have
webservers.

# roles/webserver.rb

    @@@ruby
    name "webserver"
    description "Systems that serve HTTP traffic"
    run_list(
      "recipe[apache2]",
      "recipe[apache2::mod_ssl]"
    )
    default_attributes(
      "apache" => {
        "listen_ports" => [ 80, 443 ]
      }
    )

# Per-service Roles

We create separate roles that are service specific. This allows us to
break up services to run on separate hosts, or on a single
host. Common roles in web applications:

* `database_master`
* `load_balancer`
* `memcached`
* `monitoring`
* `anything_you_want`

# Platform Roles

In a heterogenous infrastructure where multiple platforms are present,
it makes sense to have platform-specific roles to do things specific
to that platform.

Set up package repositories or service management tools and default
attributes.

# roles/ubuntu.rb

    @@@ruby
    name "ubuntu"
    description "Role applied to all Ubuntu systems."

    run_list(
      "recipe[apt]",
      "role[base]"
    )

    default_attributes(
      "chef_client" => {
        "init_style" => "upstart"
      }
    )

# roles/fedora.rb

    @@@ruby
    name "fedora"
    description "Role applied to all Fedora systems."

    run_list(
      "recipe[yum]",
      "role[base]"
    )

    default_attributes(
      "chef_client" => {
        "init_style" => "init"
      }
    )

# Apply Roles to a Node

Use knife:

    knife node run list add NODE 'role[base]'

# Summary

* Understand the components of a role
* Create roles with the Ruby DSL
* View roles on the Chef Server
* Apply roles to nodes

# Questions

* What are the components of a role?
* What is the difference between the Ruby DSL and JSON for roles?
* What knife command is used to add a role to a node's run list?
* What knife command sends the role to the Chef Server from the local repository?

# Lab Exercise

Roles

* Understand how to create roles with the Ruby DSL
* Upload a role to the Chef Server with knife
* Modify a node's run list directly with knife
