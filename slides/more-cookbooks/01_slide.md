# More Cookbooks

Section Objectives:

* Download additional cookbooks with knife
* Apply multiple cookbooks with a role
* Override a cookbook attribute from a role
* Create another new cookbook

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribution Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Download Cookbooks

Use knife to download additional cookbooks from the Opscode Chef
Community site.

    > knife cookbook site download apt
    > knife cookbook site download chef-client
    > knife cookbook site download fail2ban
    > knife cookbook site download apache2

# Extract Cookbooks

Knife downloads the cookbook from the community site as a `.tar.gz`
file. The file name will be the cookbook name, with the version. For
example, "`apache2-1.0.8.tar.gz`".

    > knife cookbook site download apache2
    Downloading apache2 from the cookbooks site at version
    1.0.8 to $PWD/apache2-1.0.8.tar.gz
    Cookbook saved: $PWD/apache2-1.0.8.tar.gz
    > tar -zxvf apache2-1.0.8.tar.gz -C cookbooks/
    > ls cookbooks/apache2

# Examine the Cookbooks

Remember, you're probably going to run Chef on the nodes as a
privileged user. After extracting the cookbooks, examine their
contents to see what they're doing.

Most cookbooks have a README file that describe what the purpose of
the cookbook is, what it does and how to use it.

Chef doesn't actually execute anything on the system unless it is
in a recipe. Start by looking at the recipes that are in the
cookbooks so you know what they're going to do.

# Upload cookbooks

Once code has been reviewed, upload the cookbooks to the server. All
cookbooks can be uploaded at one time.

    > knife cookbook upload -a
    Uploading apache2                 [1.0.8]
    Uploading apt                     [1.3.0]
    Uploading chef-client             [1.0.4]
    Uploading fail2ban                [1.0.0]
    Uploading webserver               [1.0.0]
    upload complete

# Recipe Ordering

Recipes applied to a node are processed in the order specified.

Recipes are specified in a Role or Node run list.

Run lists are ordered lists (Ruby/JSON arrays).

# Recipe Ordering

Recipes can include other recipes, too. Use "`include_recipe`".

Chef will process the included recipe in place, adding its resources
to the resource collection.

Included recipes from other cookbooks require metadata dependency.

# Cookbook Dependencies

Remember, cookbook dependencies are *assumed* when using part(s) of one
cookbook in another, such as recipe inclusion, but they are not
created automatically.

Cookbook dependencies are explicitly defined in metadata. Use the
"`depends`" keyword. This will cause Chef to download the dependency
cookbook from the server.

Downloading a cookbook as a dependency from another does not cause it
to be applied on the node. It merely makes the code/contents available
for another cookbook to use it.

# Cookbook Metadata

The metadata defines the additional cookbooks required that might not
appear in the run list explicitly.

The server reads the metadata as JSON, it does not parse the recipes
(which are Ruby).

This is a security feature, the Chef Server does not execute the Ruby
code.

# Using Roles

We typically apply multiple related recipes to a node via a role.

A `base` role is a great place to apply things that need to be done
for all systems.

# Sample base Role

    @@@ruby
    name "base"
    description "Base role is applied to all systems"
    run_list(
      "recipe[apt]",
      "recipe[fail2ban]",
      "recipe[chef-client]"
    )

# Order Matters

We put `apt` before other cookbooks in the list to make sure the
system package cache is updated for package resources.

For example, the default action in the `fail2ban` package resource is
upgrade, so apt will ensure we always have the latest version
including security fixes.

.notes Other platforms (e.g. RHEL family) might have a yum-related
recipe instead of apt. Like yum::epel

# Apply Role to Node

Applying the role to the node can be done with knife.

    > knife node run list add NODE 'role[base]'

However, this appends the role to the end of the node's run list. To
insert it before another item, you need to edit the node directly.

    > knife node edit NODE

The `$EDITOR` environment variable must be set, or specified with
`knife node edit`'s `-e` or `--editor` option.

    > knife node edit NODE -e vi

.notes On Windows, use cmd.exe not powershell, else an erroneous entry
will be made (e.g., recipe[roles]). The %EDITOR% variable can be set
to, e.g. set EDITOR="C:\Program Files\Windows NT\Accessories\wordpad.exe"

# Apply role to Node

    @@@javascript
    {
      "run_list": [
        "role[base]",
        "role[webserver]"
      ]
    }

# Run Chef

Running Chef on the target node will cause the cookbooks in the role's
run list to be downloaded just like when only a recipe is in the
node's run list.

    > sudo chef-client
    INFO: *** Chef 0.10.8 ***
    INFO: Run List is [role[base], role[webserver]]
    INFO: Run List expands to [apt, fail2ban, chef-client, webserver]

# Common Patterns

Many things automated with Chef follow a pattern:

* Install a software package.
* Write configuration files.
* Enable and start a service.

Let's walk through another example of this pattern.

.notes The haproxy cookbook is used to follow the *pattern*; the
implementation details are moot.

# haproxy Cookbook

Download the haproxy cookbook.

    > knife cookbook site download haproxy
    > tar -zxvf haproxy-1.0.4.tar.gz -C cookbooks

We will explore the haproxy cookbook for this pattern because we'll
revisit it in the next section on search.

.notes In the next section on search we'll look at the parts of the
haproxy cookbook that are modified to adapt the recipe for this
flexibility.

# haproxy default recipe

The default recipe follows the pattern.

    @@@ruby
    package "haproxy"

    template "/etc/default/haproxy"

    service "haproxy"

    template "/etc/haproxy/haproxy.cfg" do
      notifies :restart, "service[haproxy]"
    end

# haproxy default recipe

The haproxy software is available as a package named `haproxy` for
many platforms.

    package "haproxy" do
      action :install
    end

.notes On RHEL, it is available from EPEL, use yum::epel in the base role.

# haproxy default template

On Debian/Ubuntu, the service is controlled by a config file
`/etc/default/haproxy`.

    @@@ruby
    template "/etc/default/haproxy" do
      source "haproxy-default.erb"
      owner "root"
      group "root"
      mode 0644
    end

.notes Due to this specific file, the recipe won't work on RHEL
systems. If the exercises are done on RHEL systems, then we can
comment this out when it is used.

# haproxy default template

The `haproxy-default.erb` template came from the package, with a modification.

    @@@sh
    # Set ENABLED to 1 if you want the init script to start haproxy.
    ENABLED=1
    # Add extra flags here.
    #EXTRAOPTS="-de -m 16"

The haproxy init script checks the value of `ENABLED` and exits if it
is `0`.

If Chef tries to start the service, it would not actually
start unless this is modified, hence it must be managed by Chef.

# haproxy service

We will manage the haproxy service with this recipe. It will be
enabled at boot time and started if it is not running.

    @@@ruby
    service "haproxy" do
      supports :restart => true, :status => true, :reload => true
      action [:enable, :start]
    end

The init script supports the restart, status and reload commands.

     service haproxy
     Usage: /etc/init.d/haproxy {start|stop|reload|restart|status}

# haproxy configuration template

We will manage a semi-static configuration file as a template. Next
unit we will discuss a more dynamic configuration.

    @@@ruby
    template "/etc/haproxy/haproxy.cfg" do
      source "haproxy.cfg.erb"
      owner "root"
      group "root"
      mode 0644
      notifies :restart, "service[haproxy]"
    end

We could have used the `:reload` action, since the service supports it.

# haproxy configuration template

The source template is in
`cookbooks/haproxy/templates/default/haproxy.cfg.erb`.

The default configuration file that comes with the package is not
tuned for general use and will need to be modified.

This is why the default file does not enable the service.

We ship a plain configuration template in the cookbook that you can
modify. In the next unit we'll look at an example of this kind of
modification.

# haproxy configuration template

The source template for the configuration file uses some attributes
set by the cookbook. Excerpts from the template:

    listen application 0.0.0.0:<%= node["haproxy"]["incoming_port"] %>
      balance  <%= node["haproxy"]["balance_algorithm"] %>

    <% if node["haproxy"]["enable_admin"] -%>
    listen admin 0.0.0.0:22002
      mode http
      stats uri /
    <% end -%>

# haproxy attributes

    @@@ruby
    default['haproxy']['incoming_port'] = "80"
    default['haproxy']['enable_admin'] = true
    default['haproxy']['balance_algorithm'] = "roundrobin"

.notes This syntax is because we access the attributes *like* a Hash.

# haproxy modifying attributes

We can modify the attributes directly editing the cookbook, or even
better, by applying them with a role appropriate to the task.

    @@@ruby
    name "load_balancer"
    description "Systems that balance the load"
    run_list(
      "recipe[haproxy]"
    )
    default_attributes(
      "haproxy" => {
        "incoming_port" => "8080",
        "enable_admin" => false
      }
    )

.notes This syntax is because we are creating the attributes *as* a
Hash.

# Summary

* Download additional cookbooks with knife
* Apply multiple cookbooks with a role
* Override a cookbook attribute from a role
* Create another new cookbook

# Questions

* What knife command is used to download cookbooks from the Chef
  Community site?
* What is the first thing one should do after downloading a cookbook?
* How can all cookbooks be uploaded at one time?
* How does Chef determine what recipes to apply on a node?
* How does Chef determine what order to apply recipes on a node?
* How does Chef determine what cookbooks to download?
* Can Chef download cookbooks for a node that aren't in its run list?
* What is a common recipe pattern used in many cookbooks?

# Lab Exercise

More Cookbooks

* Apply apt, chef-client, and fail2ban recipes via a base role
* Include apache2 recipe in webserver recipe
* Download and examine the haproxy cookbook
