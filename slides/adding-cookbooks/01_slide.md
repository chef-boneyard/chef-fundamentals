# Adding Cookbooks

.notes These course materials are Copyright © 2010-2011 Opscode,
Inc. All rights reserved.  This work is licensed under a Creative
Commons Attribution Share Alike 3.0 License. To view a copy of this
license, visit http://creativecommons.org/licenses/by-sa/3.0/; or send
a letter to Creative Commons, 171 2nd Street, Suite 300, San
Francisco, California, 94105, USA.

# Objectives

At completion of this unit you should...

* Understand how knife handles merging multiple cookbooks.
* Understand recipe ordering.
* Apply multiple recipes to a node through a role.
* Be able to create a new cookbook and start writing simple recipes.

# Getting More Cookbooks

Knife can download more cookbooks.

    knife cookbook site install apt
    knife cookbook site install apache2

When using Git, multiple cookbooks are committed to the repository.

# Git Commit Log

    % git log commit 49e45373f294ace0c721a98f2155d4d2b558fee8 Author:
    jtimberman <joshua@opscode.com> Date: Tue Apr 26 15:27:53 2011
    -0600

       Import apt version 1.1.1

    commit 5ead5af876235c0e7bc37972ea22134cd10af7ba Author: jtimberman
    <joshua@opscode.com> Date: Tue Apr 26 12:45:57 2011 +0000

        Import fail2ban version 1.0.0

# Git Branches

    % git branch
     chef-vendor-apt
     chef-vendor-fail2ban

# Git Tags

    % git tag
    chef-vendor-apt-1.1.1
    chef-vendor-fail2ban-1.0.0

# Infrastructure as Code

This is a source code repository.

Treat it like one!

We can go back to any point in time, any version of a cookbook, and
inspect differences.

# Comparing Differences

    git checkout chef-vendor-apt
    git checkout chef-vendor-fail2ban-1.0.0
    git diff master

# Use Your VCS

Knife currently integrates with Git in the “cookbook site install”
command.

You can use any version control system you like.

There will be additional steps required to untar cookbooks,
branch/merge, etc, depending on your tool.

Open Source contributions welcome!

# Recipe Ordering

Recipes applied to a node are processed in the order specified.

Recipes are specified in a Role or Node run list.

Run lists are ordered lists (Ruby/JSON arrays).

# Recipe Ordering

Recipes can include other recipes, too. Use “include_recipe”

Chef will process the included recipe in place, adding its resources
to the resource collection.

Included recipes from other cookbooks require metadata dependency.

# Dependencies

Cookbook dependencies are implied when using part(s) of one cookbook
in another, such as recipe inclusion.

Cookbook dependencies are explicitly defined in metadata.

Use the “depends” keyword in the metadata.

# Cookbook Metadata

The metadata defines the additional cookbooks required that might not
appear in the run list explicitly.

The server reads the metadata as JSON, it does not parse the recipes
(which are Ruby).

This is a security feature, the Chef Server does not execute the Ruby
code.

# Run Lists

During the initial part of the Chef run the node’s run list is
expanded.  Roles' run lists are expanded.

Chef uses the expanded run list to determine what recipes are
included, and which cookbooks to download from the server.

Dependencies in the metadata are also downloaded.

Cookbooks are versioned. If the node has a Chef Environment, and the
environment specifies cookbook versions, those versions will be
downloaded.

# Using Roles

We typically apply multiple related recipes to a node via a role.

A base role is a great place to apply the 'fail2ban' recipe, and also
a recipe to update the APT cache on Debian/Ubuntu systems.

# Simple Base Role

    @@@ruby
    name "base"
    description "Base role is applied to all systems"
    run_list(
      "recipe[apt]",
      "recipe[fail2ban]"
    )

# Order Matters

We’ll put apt before fail2ban to make sure the package cache is
updated.

The default action in the fail2ban package resource is upgrade, so apt
will ensure we always have the latest version including security
fixes.

# Upload Base Role

Use knife to upload the base role.

    knife role from file base.rb

This will look for the role file (base.rb) in the roles sub-directory
of the Chef Repository.

# Apply Role to a Node

First remove the fail2ban recipe, it will be applied via the base role.

    knife node run list remove NODE ‘recipe[fail2ban]’

Then add the role to the node.

    knife node run list add NODE ‘role[base]’

# Run chef-client

Running Chef on the target node will cause the APT recipe to be
applied, then fail2ban.

Since fail2ban was applied previously, it will not actually take any
action on the node (unless there’s a new package version!).

# Creating Cookbooks

New cookbooks can be created with Knife.

This will create a skeleton directory structure with all the
components of a cookbook.

# knife cookbook create

    % knife cookbook create webserver
    ** Creating cookbook webserver
    ** Creating README for cookbook: webserver
    ** Creating metadata for cookbook: webserver

# Default Cookbook Contents

    tree cookbooks/webserver
    cookbooks/webserver
    ├── README.rdoc
    ├── attributes
    ├── definitions
    ├── files
    │   └── default
    ├── libraries
    ├── metadata.rb
    ├── providers
    ├── recipes
    │   └── default.rb
    ├── resources
    └── templates
        └── default

# Pre-generated Content

Knife pre-generates some information in the cookbook.

* README.rdoc
* metadata.rb
* recipes/default.rb

The values used come from configuration options in the knife cookbook create command.

# Configure Knife

Knife is not configured for creating new cookbooks by default. It has three settings that can customize the pre-filled content.

* `cookbook_email`
* `cookbook_copyright`
* `cookbook_license`

# knife cookbook create

    cookbook_copyright "Opscode, Inc."
    -C, --copyright COPYRIGHT

    cookbook_license "apachev2"
    -I, --license LICENSE

    cookbook_email "cookbooks@opscode.com"
    -E, --email EMAIL

Used to prefill default recipe, README and metadata

`cookbook_*` goes in knife.rb, and corresponds to command-line options

Licenses available: apachev2, gplv2, gplv3, mit, none

# Note About Software Licenses

Cookbooks are distributable, be careful about the software license you choose.

You can use whatever you like.

Be aware of what you include in your cookbook, though!

Opscode uses Apache 2.0 Software License. Use the license you prefer.

* This is not legal advice, ask a lawyer if unsure.

# Modifying the Cookbook

Once the cookbook is created, it is time to start modifying it to do something meaningful.

Typically, this will involve editing the default recipe.

    cookbooks/webserver/recipes/default.rb

Edit this with your favorite text editor.

# Cookbook Components

Not all the cookbook components are used.

They are created so you don’t have to remember them all.

Remove the ones you know you don’t want.

# Common Cookbook Components

The most common cookbook components:

* attributes
* recipes
* metadata
* README :)
* files and / or templates

# Cookbook Attributes

Cookbook attributes files set up sane defaults to be used in the cookbook.

We will cover them in depth next unit.

# Cookbook Recipes

Most things in Chef recipes will be resources to configure.

Some things in Recipes will be pure Ruby syntax and helper code.

Your first recipes will be simple.

# Ruby DSL

A DSL is a domain-specific language

* Recipes are a pure Ruby DSL
* Recipes are processed in two phases
* Compilation and Execution happen in the order specified.

# Attributes in Recipes

The node object is available in recipes, and we can access the attributes like a Ruby hash key.

* `node['fqdn']`
* `node['kernel']['machine']`

Attributes can come from cookbook files, roles, or the node itself.

# Common Chef Resources

Chef supports a couple dozen resources, but a few are most commonly used. We’ll mostly work with the common ones.

* package
* service
* template, cookbook_file, remote_file
* user
* execute

# Common Resources: Package

    @@@ruby
    package "tar" do
      action :install
    end

    package "tar" do
      version "1.16.1-1"
      action :install
    end

    package "tar" do
      action :upgrade
    end

    package "tar" do
      action :remove
    end

# Common Resources: Service

    @@@ruby
    service "example_service" do
      supports :status => true, :restart => true, :reload => true
      action [ :enable, :start ]
    end

.notes Recall the 'supports' meta-parameter controls the way the service's provider is going to work with the service on the system.
:status => true, Chef will use the built in status command for the provider to check if the service is running. Commonly this is like '/etc/init.d/SERVICE status.'
:restart => true, Chef will use the built in restart command for the provider to restart the service. Commonly this is like '/etc/init.d/SERVICE restart.'
:reload => true, Chef can send a reload action to a service only if it supports reload. Chef will use the reload command for the provider usually to tell the daemon about a configuration change. Commonly this is like '/etc/init.d/SERVICE reload.'

# Common Resources: Template

    @@@ruby
    # Config file from a template
    template "/tmp/config.conf" do
      source "config.conf.erb"
    end

    # Config file from a template with variable map
    template "/tmp/config.conf" do
      source "config.conf.erb"
      variables(
        :config_var => node["configs"]["config_var"]
      )
    end

# Common Resources: Cookbook File

    @@@ruby
    cookbook_file "/usr/local/bin/apache2_module_conf_generate.pl" do
      source "apache2_module_conf_generate.pl"
      mode 0755
      owner "root"
      group "root"
    end

# Common Resources: Remote File

    @@@ruby
    remote_file "/tmp/testfile" do
      source "http://www.example.com/tempfiles/testfile"
      mode "0644"
      checksum "08da002l"
    end

# Common Resources: User

    @@@ruby
    user "random" do
      comment "Random User"
      uid "1000"
      gid "users"
      home "/home/random"
      shell "/bin/zsh"
      password "$1$JJsvHslV$szsCjVEroftprNn4JHtDi."
    end

    # System User
    user "systemguy" do
      comment "system guy"
      system true
      shell "/bin/false"
    end

.notes For password setting, you need to generate the password
appropriate for your system.

# Common Resources: Execute

    @@@ruby
    execute "generate-module-list" do
      command "/usr/local/bin/apache2_module_conf_generate.pl /usr/lib/httpd/modules /etc/httpd/mods-available"
      action :run
    end

The command parameter calls a cookbook_file dropped off by an earlier
example.

# Providers Take Idempotent Actions

Providers know how to run the right commands or API calls to configure
a resource by taking the appropriate action.

Providers should be written so they are idempotent and don’t configure
the resource if it is in the declared state.

Providers are sometimes platform specific (apt, yum, useradd,
chkconfig, etc).

# Cookbook Metadata

Metadata

* Information about the cookbook
* Instructs server what to distribute (dependencies)
* Specifies the version
* “Packaging” information

# Cookbook Metadata

Use metadata to define dependencies.

Dependencies are required when using a cookbook’s components in
another cookbook.

    depends “classroom”

# Cookbook Metadata

    @@@ruby
    maintainer       "YOUR_COMPANY_NAME"
    maintainer_email "YOUR_EMAIL"
    license          "All rights reserved"
    description      "Installs/Configures webserver"
    version          "0.0.1"
    depends          "apache2"

# Cookbook Assets

Cookbooks have two kinds of assets that aren’t strictly Ruby code that can be distributed.

* Files (`cookbook_file` resource)
* Templates (`template` resource)

# Cookbook Assets: Files

Files

* Static assets
* file, `cookbook_file`, `remote_file` resources
* For directories, `remote_directory` resource
* File specificity

.notes "file" is a basic resource that manages the properties of a file - owner, mode, and basic content.
"`cookbook_file`" is a file distributed from the cookbook.
"`remote_file`" is a file downloaded from a remote source.
An early common development pattern is to start using `cookbook_file` for static configuration files.

# Cookbook Assets: Templates

Templates

* Dynamic
* ERB (embedded ruby)
* Access to `node` and `variables`
* template resource
* File specificity

.notes Templates are dynamically rendered by Chef during convergence. The content of the rendered template is compared via SHA256 checksum to the target file.
Variables can be passed into the template resource as a parameter. The node object is accessible in a template as well.

# File Specificity

File specificity is how Chef determines source file for files and templates

Directory locations under files or templates

* host-FQDN
* PLATFORM-VERSION
* PLATFORM
* default

# File Specificity Example

    % ls cookbooks/classroom/files/*
    host-www.example.com/thing.conf
    ubuntu-10.04/thing.conf
    ubuntu/thing.conf
    default/thing.conf

# Summary

You should now be able to...

* Explain how knife handles merging multiple cookbooks.
* Describe recipe ordering through inclusion.
* Apply multiple recipes to a node through a role.
* Create a new cookbook and understand the components.

# Lab Exercise

## Adding Cookbooks
