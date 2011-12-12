# Example Cookbook

.notes These course materials are Copyright © 2010-2011 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribution Share Alike 3.0 License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Objectives

At completion of this unit you should...

* Know how to get cookbooks with Knife and the Opscode Chef Community site
* Understand the components of a cookbook
* Be able to upload a cookbook to Opscode Hosted Chef
* Add a cookbook’s recipe to a node’s run list.

# Chef Cookbooks

Cookbooks are “packages” for Chef recipes and other helper components.

You can create new cookbooks or download cookbooks from the Cookbooks
site.

# Cookbooks are Sharable

We design cookbooks to be sharable with others. Attributes, allow you
to interchange data. Use file specificity to customize the files and
templates for particular platforms. Resources, providers and libraries
you can extend cookbooks.

These features of cookbooks bring flexibility and code reuse for your
infrastructure.

# Cookbook Basics

Cookbooks contain a variety of components. The most commonly used are:

* recipes
* files and templates
* metadata

# Getting Cookbooks

Opscode hosts the Chef Community site

* http://community.opscode.com

Install cookbooks from the site in the local chef-repo with knife’s vendor pattern.

    knife cookbook site install COOKBOOK

Or, download a .tar.gz with knife:

    knife cookbook site download COOKBOOK

# Getting Cookbooks

<center><img src="getting-cookbooks.png"></center>

# knife cookbook site install

    knife cookbook site install fail2ban

* Uses “vendor branch” pattern with Git.
* Downloads the cookbook from the site as a tarball
* Checks out a “vendor branch” for the cookbook
* Untars the cookbook into the vendor branch
* Commits the changes with git
* Merges back into master

# knife cookbook site download

    knife cookbook site download fail2ban
    tar -zxvf fail2ban-1.0.0.tar.gz -C cookbooks

Simply downloads a tarball of the cookbook from the site.

Use this if you’re not using Git.

Follow the pattern: Create a branch, merge, etc.

# fail2ban Cookbook

“fail2ban” blocks SSH and other TCP connections via firewall rules based on failed login attempts.

It’s useful to have as a security measure.

This cookbook will set up fail2ban on a Linux node.

# Knife Cookbook Site Commands

    knife cookbook site show fail2ban
    knife cookbook site search fail2ban
    knife cookbook site search security

# Install fail2ban in Repository

    > knife cookbook site install fail2ban
    Installing fail2ban to /Users/jtimberman/Development/sandbox/wut-chef-repo/cookbooks
    Checking out the master branch.
    Creating pristine copy branch chef-vendor-fail2ban

# Install fail2ban in Repository

    Downloading fail2ban from the cookbooks site at version 1.0.0 to /Users/jtimberman/Development/sandbox/wut-chef-repo/cookbooks/fail2ban.tar.gz
    Cookbook saved: /Users/jtimberman/Development/sandbox/wut-chef-repo/cookbooks/fail2ban.tar.gz
    Removing pre-existing version.
    Uncompressing fail2ban version /Users/jtimberman/Development/sandbox/wut-chef-repo/cookbooks.
    removing downloaded tarball

# Install fail2ban in Repository

    1 files updated, committing changes
    Creating tag cookbook-site-imported-fail2ban-1.0.0
    Checking out the master branch.
    Updating b99add3..555e908
    Fast-forward
     cookbooks/fail2ban/metadata.json                   |   36 ++++
     cookbooks/fail2ban/metadata.rb                     |   11 +
     cookbooks/fail2ban/recipes/default.rb              |   36 ++++
     .../fail2ban/templates/default/fail2ban.conf.erb   |   34 ++++
     cookbooks/fail2ban/templates/default/jail.conf.erb |  202 ++++++++++++++++++++
     5 files changed, 319 insertions(+), 0 deletions(-)
     create mode 100644 cookbooks/fail2ban/metadata.json
     create mode 100644 cookbooks/fail2ban/metadata.rb
     create mode 100644 cookbooks/fail2ban/recipes/default.rb
     create mode 100644 cookbooks/fail2ban/templates/default/fail2ban.conf.erb
     create mode 100644 cookbooks/fail2ban/templates/default/jail.conf.erb
    Cookbook fail2ban version 1.0.0 successfully installed

# Common Recipe Patterns

The fail2ban cookbook follows a common recipe pattern.

* Install a package
* Create configuration files from templates
* Manage a service

# Package Resource

    @@@ruby
    package "fail2ban" do
      action :upgrade
    end

The upgrade action is like install but if Chef determines a newer version is available, it will update.

The fail2ban package will only have security fixes so it is safe to upgrade.

# Service Resource

    @@@ruby
    service "fail2ban" do
      supports [ :status => true, :restart => true ]
      action [ :enable, :start ]
    end

The fail2ban service specifies a meta-parameter, “supports.” This affects the way the service provider works with the service init script.

We can specify multiple actions by passing an array. These are processed by Chef in order.

# Template Resources

    @@@ruby
    %w{ fail2ban jail }.each do |cfg|
      template "/etc/fail2ban/#{cfg}.conf" do
        source "#{cfg}.conf.erb"
        owner "root"
        group "root"
        mode 0644
        notifies :restart, "service[fail2ban]"
      end
    end

This is two template resources set up by using the Ruby “each” loop over an array.

The %w{ } syntax creates a quoted array of words.

The resource takes several parameters.

# Template Source Files

    fail2ban/templates/
    `-- default
        |-- fail2ban.conf.erb
        `-- jail.conf.erb

For templates, the source files are ERB (embedded Ruby), generated by the node.

Dynamic templates and static files in cookbooks follow rules of file specificity.

# File Specificity

File specificity is how Chef determines source file for files and templates
Directory locations under files or templates:

* host-FQDN
* PLATFORM-VERSION
* PLATFORM
* default

FQDN is `node[‘fqdn’]`. PLATFORM is `node[‘platform’]` and VERSION is `node[‘platform_version’]`.

# File Permissions

The file resource type has three file permission parameters.

* owner
* group
* mode

These are inherited by template, `cookbook_file` and `remote_file` resources.

Be explicit.

# Notification meta-parameter

    @@@ruby
    resource "name" do
      notifies :action, "resource_type[resource_name]", :timing
    end

Notifications tell a resource take an action when this resource is updated.

The basic form of a notification parameter for a resource.

Timing can be :delayed or :immediately. :delayed is the default.

# Template Notifications

    @@@ruby
    %w{ fail2ban jail }.each do |cfg|
      template "/etc/fail2ban/#{cfg}.conf" do
        #...
        notifies :restart, "service[fail2ban]"
      end
    end

Notifications can be sent or received between resources.

“Subscribes” is the opposite of notifies.

Multiple notifications with the same action get stacked but only triggered the first time.

# Cookbook Upload

Upload the cookbook to Opscode Hosted Chef with knife.

    knife cookbook upload fail2ban

This uploads the contents of the chef-repo/cookbooks/fail2ban
directory.

# knife cookbook upload fail2ban

    > knife cookbook upload fail2ban
    Uploading fail2ban             [1.0.0]
    upload complete

    > knife cookbook upload fail2ban -VV
    [ more verbose DEBUG output ]

    > knife cookbook show fail2ban 1.0.0

Check Ruby and ERB (template) syntax.

Create a “sandbox,” which is in a private Amazon S3 bucket with filenames as checksums.

Write the cookbook manifest.

# Add Recipe to a Node

The cookbook sets the namespace for the recipe to add to the node. The default recipe can be omitted.

    recipe[fail2ban]
    recipe[fail2ban::default]

Are equivalent.

# Add Recipe to a Node

Use knife to add a recipe to a node’s run list.

    knife node run list add NODE ‘recipe[fail2ban]’

Use quotes to prevent shell meta-character expansion.

Run Chef on the node and it will apply the recipe.

# Add Recipe to a Node

Knife's bootstrap subcommand will SSH to a node, automatically install
Chef, configure the node to connect to the server and run Chef with
the specified run list.

    knife bootstrap NODE -r "recipe[fail2ban]"

# Summary

You should now be able to...

* Describe common components of a cookbook
* Get cookbooks with Knife and the Opscode Chef Community site
* Upload a cookbook to Opscode Hosted Chef
* Add a cookbook’s recipe to a node’s run list.

# Additional Resources

* http://wiki.opscode.com/display/chef/Cookbooks
* http://community.opscode.com/cookbooks
* http://www.slideshare.net/jtimberman/mwrc2011-cookbook-design-patterns
* http://jtimberman.github.com/guide-to-writing-chef-cookbooks

# Lab Exercise

## Example Cookbook
