# Roles

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribute Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Objectives

At completion of this unit you should...

* Understand the components of a role.
* Know recommended practices for roles.
* Be able to create a new role.
* Be able to apply roles to nodes.

# Components of a Role

Run list

* recipes
* roles

Attributes

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

In a service oriented architecture, each different service should have its own role with the recipes that determine how to fulfill that service.

For example, it is common in a web-application architecture to have webservers.

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

You should now be able to...

* Describe the components of a role.
* Describe recommended practices for roles.
* Create a new role.
* Apply roles to nodes.

# Lab Exercise

## Roles
