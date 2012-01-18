# Cookbooks Recipes Resources

.notes These course materials are Copyright © 2010-2011 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribution Share Alike 3.0 License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Objectives

# Resources

Resources are the fundamental configuration object.

Chef manages resources on the node so they comply with policy.

# Resources

Declare some aspect of policy

* The apache2 package should be installed.
* The application user should be created.

# Resources

Abstract the details. The commands:

* apt-get install apache2
* useradd application

Become resources:

* package “apache2”
* user “application”

# Resources

Resources take idempotent actions through providers.

Providers know how to determine the current state.

Providers do not take action if the resource is in the declared state.

# Resources

Non-compliance with policy is a bug.

Chef converges the node to make it compliant with the policy.

A single Chef run should completely converge the node.

# Resources

Resources are data driven.

* Packages have versions
* Users have home directories, shells and numeric IDs.

This data can come from multiple sources, either by documenting in the code itself or an external source.

# Resources Chef Can Configure

* directories, files, templates, remote files
* packages, services, users, groups
* scripts, commands, ruby code blocks
* subversion and git code repositories
* application deployment, HTTP requests
* network interfaces, filesystem mounts

Chef includes over 25 different kinds of resources.

# User-created Resources

Chef is flexible and extensible and new resources and providers can be
created.

* Cookbook LWRPs (Lightweight DSL)
* Cookbook Libraries

# What Does a Resource Look Like?

* Resources have a type
* Resources have a name
* Resources take parameter attributes
* Resources specify the action to take

# Example Resource

    @@@ ruby
    template "/tmp/config.conf" do
      source "config.conf.erb"
      owner "root"
      group "root"
      mode 0644
      action :create
    end

.notes in the absense of parameters, default values are assumed. same
with actions

# Resources Have Sane Defaults

Each resource has a "name attribute."

This corresponds to a parameter attribute as the default value.

Parameter attributes all have default values internal to Chef. Specify your own to be explicit, or to change the default.

Resources also have a default action. The default value depends on the
resource type.

# Recipes

Recipes are the majority of the code written to execute the policy.

Recipes contain declarative resources.

Supporting code for recipes can also be written as libraries, or new
resources.

# Recipes

Internal Ruby domain-specific language.

* You need a 3rd generation programming language.
* You can’t be limited by the language.

# Recipes

Recipes are processed in the order they are written.

* Ruby code is evaluated.
* Ruby recognized as Resources are added to the resource collection.
* Chef walks the resource collection to configure the resources.
* Providers take action to configure the resource as it was declared.

# How Are Recipes Applied?

Nodes have a list of recipes they will run.

This run list can include recipes that also include other recipes.

These are applied to the node in the order listed.

# Cookbooks

Cookbooks are packages for recipes.

They are organized into directories.

Cookbooks contain one or more recipes.

Cookbooks can also contain other components.

# Cookbooks

Download existing cookbooks

* http://community.opscode.com

Create new cookbooks

Modify cookbooks

Upload cookbooks to the Chef Server

# Summary


# Lab Exercise
