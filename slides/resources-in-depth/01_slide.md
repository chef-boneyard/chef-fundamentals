# Resources In Depth

Section Objectives:

* Understand the components of resources.
* Know the commonly used resources in Chef.
* Write recipes using common resources.

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribution Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Special Exercise format

This unit has a special exercise format. We will use `shef` to
interactively write resources and observe the results.

Not all resources in the examples will work exactly as written. The
goal is to understand the syntax and how Chef processes the resource,
not to configure specific things on the system.

Students should be logged into the provided remote target instance
rather than their local workstation. `shef` should be started as a
privileged user (e.g., `sudo shef`).

.notes Shef in Windows powershell or cmd.exe, backspace doesn't behave
precisely as expected.

# Shef

`shef`, the Chef Shell (or Console) operates under different execution
contexts and the prompt indicates the context.

Shef starts in the "main" context. The prompt will look like this:

    chef >

This is where general Chef Server API calls can be made through
Shef-defined methods.

The `help` command is available in all contexts and displays the
built-in commands available.

# Shef

Two other contexts are available.

`attributes` is the context of editing a "cookbook attributes" file.

    chef > attributes
    chef:attributes >

`recipes` is the context of the Chef Recipe DSL.

    chef > recipe
    chef:recipe >

We are going to write resources using the Recipe DSL, so ensure you
are in the recipe context.

.notes Using the recipe context allows direct demonstration of the
"compile" vs "execute" phases of node convergence. We're not actually
doing the "execute" part in these demonstrations.

# Chef Resource

Syntax example of Chef Resources:

    @@@ruby
    type "name"

    type "name" do
      parameter "value"
      parameter 0
      action :action
    end

The values of parameters can be various Ruby data type objects
depending on the specific resource. If parameters are not specified,
Chef will use its default value.

The action, if specified, **must** begin with a colon (Ruby symbol).

# Shef Simple Example

    chef:recipe > file "/tmp/foo"
    => <file[/tmp/foo] @name: "/tmp/foo" @noop: nil @before: nil @params:
    {} @provider: nil @allowed_actions:
    [:nothing, :create, :delete, :touch, :create_if_missing] @action:
    "create" @updated: false @updated_by_last_action: false @supports:
    {} @ignore_failure: false @retries: 0 @retry_delay: 2
    @immediate_notifications: [] @delayed_notifications: []
    @source_line: "(irb#1):1:in `irb_binding'" @resource_name: :file
    @path: "/tmp/foo" @backup: 5 @cookbook_name: nil @recipe_name: nil
    @enclosing_provider: nil>

.notes We will come back to this very shortly.

# Common Resources

By the "80/20 rule", you'll use a small subset of the available Chef
resources most of the time.

* File management
* Packages
* Users and Groups
* Services/daemons
* Executable programs

# File management

Configuration management often entails writing configuration files, so
Chef has a number of built-in resources to manage files.

* `file`
* `cookbook_file`
* `remote_file`
* `template`
* `link`
* `directory`
* `remote_directory`

# Content Idempotence

When managing file contents, Chef uses SHA-256 checksums on the target
file content to determine if a file should be updated. If the checksum
of the content matches, Chef makes no further changes.

`file` (with `content` parameter), `cookbook_file`, `remote_file` and
`template` resources all use checksums.

.notes SHA-256 was chosen because no known collisions have been found. MD5
has known published collision attacks, as does SHA-0. SHA-1 has
theoretical collisions found.

# file

The `file` resource is the basis for other "file" resources:

* `cookbook_file`
* `remote_file`
* `template`

The "name attribute" for all file resources is the target file path on
the system, e.g. "`/etc/passwd`."

The default action is to create the file.

# file example

Type this into your Shef session (recipe context):

    @@@ruby
    file "/tmp/shef_created_this" do
      owner "root"
      group "root"
      mode 0644
    end

* path - name attribute, target file path on the system
* owner - user that should own the file, default is user running chef
* group - group that should own the file, default is group running chef
* mode - octal mode of the file, specify as a number with a leading 0,
  or as a string, default is the umask of the chef process

.notes owner, group, mode on Windows do not work on Windows before
Chef 0.10.10, and full inheritance and ACL support will be available
after that release.

# file parameters

* content - a string to write to the file, overwrites existing content
* backup - number of backups to keep, default is 5

.notes the "content" parameter is useful to copy the content of one
file to another as well, in an idempotent fashion.

# cookbook_file

Use `cookbook_file` to transfer files from the cookbook to the node.

* static configuration files
* small binary files
* inherits from `file`, so permission parameters can be used.

.notes Do not store and transfer large files in the cookbook, this is
for static content like scripts (e.g. plugins)

# cookbook_file example

Do not type this resource into Shef, as it will fail to converge.

    @@@ruby
    cookbook_file "/usr/local/bin/cpan_install" do
      source "cpan_install"
      mode 0755
    end

* source - the file name to transfer, located in the `files/default`
  directory of the cookbook
* cookbook - _optional_ a different cookbook where the file is

# remote_file

Use `remote_file` to download a file from a remote URL, rather than
using, e.g., `wget`.

Remote files are retrieved from the source URI, which can be anything
but most often an "`http://`" or "`ftp://`" URL.

The default action is to create the target file by downloading the
source URI.

# remote_file example

Type the following in your Shef session (recipe context):

    @@@ruby
    remote_file "/tmp/chef-install.sh" do
      source "http://opscode.com/chef/install.sh"
      mode 0755
      checksum "3dd0daa5"
    end

* path - name attribute, the target file to write
* source - the URI of the file to download
* mode, owner, group - inherited from `file`
* checksum - a portion or all of the SHA-256 checksum of the file

.notes `remote_file` is not idempotent unless you give it a checksum, or
otherwise guard the resource. We'll talk about ways to do that later
on (`not_if`/`only_if`)

# remote_file download location

It is common to download files to a temporary location, such as
`/tmp`. However, some distributions or platforms will remove this on
reboot. Thus `/var/tmp` may be desirable, but various system policies
define particular characteristics about the `/var` filesystem.

In Chef, it is common practice to use the location where Chef caches
cookbooks and other files as the download location.

    @@@ruby
    remote_file "#{Chef::Config[:file_cache_path]}/install.sh" do
      source "http://opscode.com/chef/install.sh"
    end

.notes It is not required to type all this resource in since we used a
similar one previously.

# template

Create dynamically rendered configuration files with ERb templates and
the `template` resource.

This is the most common resource for configuration files.

The entire Chef `node` object is available. Additional variables can
be passed into the template as a parameter.

The default action will create the target file rendered from the
template.

# template example

Do not type this resource into Shef, as it will fail to converge.

    @@@ruby
    template "/var/www/index.html" do
      source "index.html.erb"
      variables :welcome => "<h1>It works!</h1>"
    end

* path - the name attribute, target path to the file to render
* mode, owner, group - inherited from `file`
* source - the erb file in the `templates/default` directory to use
* cookbook - _optional_ another cookbook where the template is
* variables - a hash of variables to pass into the template, which
  must be specified using Ruby symbols as keys

# ERb syntax

Access node attributes:

    <%= node['ipaddress'] %>
    <%= node['fqdn'] %>
    <%= node['ec2']['public_hostname'] %>

Access `variables` parameter:

    <%= @welcome %>

Use Ruby code, e.g. conditionals:

    <% if node['platform'] =~ /debian/ -%>
    I'm on Debian.
    <% end -%>

.notes Do not type this into Shef, as it will do us no good.

# Data driven configuration

Templates are dynamically rendered from node data and the variables
parameter.

Common use cases:

* Service configuration for *this* node's IP address (`node['ipaddress']`)
* Determine particular configuration values based on the node's
  platform (`node['platform']`)
* Service configuration to connect to another node's IP address (via a
  search)

.notes We will cover search and how do go about this in a later section.

# directory

Create a directory with the `directory` resource. It is the basis for
the `remote_directory` resource as well.

Like `file`, the "name attribute" for directories is the path to the
target directory.

The default action is to create the specified directory.

# directory example

Type this into your Shef session (recipe context):

    @@@ruby
    directory "/var/cache/chef" do
      mode 0755
    end

    directory "/var/www/sites/mysite" do
      owner "www-data"
      recursive true
    end

* path - name attribute, the target path of the directory
* mode, owner, group - work like the `file` resource (but directory
  does not inherit from `file`)
* recursive - recursively create the directory tree, like `mkdir -p`

# Packages

Package management is typically handled by the configuration
management system to ensure that the state of the system has the
correct software installed.

Packages are installed using the default package management system
chosen by platform using the `package` resource.

One of the keys to cross-platform support in Chef is having package
management providers for various platforms.

The default action is to install the named package.

# package examples

Type this into your Shef session (recipe context):

    @@@ruby
    package "apache2" do
      action :install
    end

Do not type this one in, as the package version may change.

    package "apache2" do
      version "2.2.14-5ubuntu8.7"
    end

* package_name - name attribute, name of the package
* version - specific version to install, uses the latest version
  detected by the provider (e.g., apt cache or yum repo data)

# Additional package examples

Do not type this resource into Shef, as it will fail to converge.

    @@@ruby
    package "apache2" do
      package_name "httpd"
    end

    package "apache2" do
      action :upgrade
    end

Type this into your Shef session (recipe context):

    package "portmap" do
      action :remove
    end

.notes Be aware of removing packages that have dependencies, as Chef
may not be able to determine all the correct dependency resolution on
its own and other "package removal" resources may be required.

# Package Providers

Chef supports many package management systems. The default is
determined by the node's platform:

* `apt_package` - Debian/Ubuntu
* `yum_package` - RHEL/CentOS/Fedora/Amazon
* `macports_package` - Mac OS X
* `freebsd_package` - FreeBSD (ports system)
* `pacman_package` - Archlinux
* `portage_package` - Gentoo

# Package Providers

* `dpkg_package` - install a .deb
* `rpm_package` - install a .rpm
* `gem_package` - RubyGems
* `easy_install_package` - Python easy-install

.notes dpkg requires that the file is downloaded to the local system.
rpm can retrieve over HTTP just like the rpm command, but it will not
follow redirects properly; e.g., EPEL RPMs.

# Cookbook Package Resources

* `homebrew_package` - homebrew cookbook, sets OS X default provider
  to [homebrew](http://mxcl.github.com/homebrew/).
* `dmg_package` - dmg cookbook, for OS X
* `pacman_aur`, `pacman_group` - pacman cookbook, for Archlinux
* `python_pip` - python cookbook, use pip instead of `easy-install`
* `windows_package` - windows cookbook, for Windows systems

Complete list provided by Opscode published cookbooks:
[http://wiki.opscode.com/display/chef/Opscode+LWRP+Resources](http://wiki.opscode.com/display/chef/Opscode+LWRP+Resources)

Download cookbooks from http://community.opscode.com/cookbooks

.notes Trying to find cookbooks for these things on GitHub is prone to
confusion. Use the Community site.

# Users and Groups

Local system groups and users can be managed with the `group` and
`user` resources.

* `group`
* `user`

The commands to manage groups and users vary by platform, so there are
different providers.

The default action for both resources is to create the named group or user.

# group example

Type this into your Shef session (recipe context):

    @@@ruby
    group "admins" do
      gid 999
      members ['joe', 'alice']
    end

* gid - the numeric group id
* members - the list of users that should be members of the group
* append - the members will be appended to an existing group

# user example

Type this into your Shef session (recipe context):

    @@@ruby
    user "joe" do
      comment "Joe User"
      shell "/bin/bash"
      home "/home/joe"
      gid "users"
      uid 1002
      supports :manage_home => true
    end

* username - name attribute, name of the user
* comment - GECOS field or user comment
* shell - user login shell, use full path
* home - user's home directory
* gid - user's primary group, can be name or numeric
* uid - numeric user id
* supports - (Array) indicate that the useradd command supports
  managing the user home directory, e.g. "-m"

# system user example

Type this into your Shef session (recipe context):

    @@@ruby
    user "myapp" do
      system true
      comment "my appliation user"
      shell "/bin/false"
    end

* system - creates a "system" user assigning a UID per the platorm's
  OS policy for user creation (e.g., `/etc/login.defs`).

# user passwords

Chef can manage the user's password as well. The local installation of
Ruby must have Shadow password support on Unix/Linux systems. This is
available in the full-stack installer by installing the `ruby-shadow`
RubyGem, as Ruby 1.9 removed it from the standard library.

Passwords must be supplied using the correct password hashing for the
underlying OS. For example, generate an MD5 hashed password with openssl:

    openssl passwd -1 "theplaintextpassword"

# user password example

Do not type this into Shef.

    @@@ruby
    user "joe" do
      comment "Joe User"
      shell "/bin/bash"
      home "/home/joe"
      gid "users"
      uid 1002
      password "$1$JJsvHslV$szsCjVEroftprNn4JHtDi."
    end

.notes This does not have to be repeated from the earlier example,
especially all the typing of that obnoxious password.

# Services/daemons

Services or application daemons can be managed in Chef using the
`service` resource. Like packages and groups/users, services have
platform dependent commands.

* `service`

The service resource does not have a default action.

.notes Why no default? Because Chef doesn't have a "reasonable" idea
of what it means to say you have a service resource to manage.

# service example

Type this into your Shef session (recipe context):

    @@@ruby
    service "apache2" do
      supports :status => true
      action :enable
    end

* supports - a metaparameter specially supported by service providers,
  in this case indicates that the init script can take a "status"
  command, `:restart` and `:reload` are also available.

# Starting services

As Chef takes action to configure resources in the order written, it
may be sensible to not start a service right away. It is common
practice to follow the pattern:

* Install the package
* Enable the service at boot time
* Write configuration files
* Start the service

# Starting services

Service resources can have a `:start` action to start running the
service. This depends on the provider, but it is typically by using
the init script or control scripts that call the init script. For
example:

* Debian/Ubuntu: `/usr/sbin/invoke-rc.d`
* Red Hat family: `/sbin/chkconfig`

# Starting services

A common way to start a service is to issue a `:restart` action through
resource notification.

    @@@ruby
    service "apache2" do
      supports :restart => true
      action :enable
    end

    template "/etc/apache2/apache2.conf" do
      # ... other parameters
      notifies :restart, "service[apache2]"
    end

# Notifications

Notifications cause resource state to change with the specified action
based on change to another resource.

There are two kinds of notifications, `notifies` and `subscribes`. We
will focus on `notifies`. The take the following form:

    @@@ruby
    resource "my-name" do
      notifies :action, "type[their-name]", :timing
    end

If `resource[my-name]` changes (such as a template being rendered),
then `:action` will be sent to `type[their-name]`. The `:timing` can
be `:delayed` (default if not specified) or
`:immediately`. Notifications are queued, and happen only one time.

.notes Child A, go tell this message (action) to your sibling
(type[their name]), and do so right now (:timing). Or later on.

# Starting services

If the `template[/etc/apache2/apache2.conf]` resource changes, it will
cause the `apache2` service to be restarted. If it were *not* already
running, this usually means it will be started.

    @@@ruby
    service "apache2" do
      supports :restart => true
      action :enable
    end

    template "/etc/apache2/apache2.conf" do
      # ... other parameters
      notifies :restart, "service[apache2]"
    end

# Restarting services

Do not specify the action `:restart` on a service resource in a recipe
as it will restart every time Chef runs. Do use notifies to send the
`:restart` action from configuration files.

When Chef takes the `:restart` action on the resource, by default it
will attempt to "stop" then "start" the service using the provider's
stop and start commands.

If the parameter `supports :restart => true` is set, then Chef will
use the `restart` command for the service.

    sudo /usr/sbin/invoke-rc.d apache2 restart

# Reloading services

You can reload services using their init script's reload command, but
only if the service resource supports reload:

    @@@ruby
    service "apache2" do
      supports :reload => true
    end

This is often used to "HUP" the process to reload the
configuration. Refer to your system's or package's documentation.

# Executing programs

Sometimes to configure a system to do its job, commands or scripts
need to be run. Chef has command and script execution resources:

* `execute`
* `script` (Bash, Csh, Perl, Python, Ruby)

Chef will shell out to the system and run the specified command with
execute, or render the code in a script and execute it.

The `execute` resource is the basis for `script` and its child
resources (`bash`, `csh`, `perl`, `python` and `ruby`).

The default action for execute and script resources is to run the command/script.

# execute examples

Type this into your Shef session (recipe context):

    @@@ruby
    execute "apt-get update" do
      ignore_failure true
    end

Do not type this resource into Shef, as it will fail to converge.

    execute "configure software" do
      command "./configure"
      cwd "/usr/src/software-1.0"
    end

* command - name attribute, the command to run
* cwd - directory where the command is run
* creates - a file that is created by the command
* user - user to run the command
* group - group to run the command
* environment - hash of environment variables to set
* returns - valid command return codes, can be an array
* ignore_failure - meta-parameter that doesn't exit the Chef run if
  the resource fails to be configured

# scripts

The script resource defines a script that will be rendered as a file
and executed by Chef with the given interpreter. There are shortcut
script resources that will use the specified interpreter.

    @@@ruby
    script "install_something" do
      interpreter "bash"
      code <<-EOH
      # the code
      EOH
    end

    bash "install_something" do
      code <<-EOH
      # the code
      EOH
    end

.notes This is useful as a technique to move existing "legacy" shell
scripts and the like into Chef.

# Idempotent commands

By their nature, arbitrary commands and scripts are *not* idempotent,
since Chef just executes the raw commands. They will run every time
Chef runs. Some strategies can be used to make execute and script
resources idempotent.

* use `creates` parameter, if the command creates a file, it won't be
  run if the file already exists.
* use `not_if` or `only_if` metaparameters

# Conditional execution

The `not_if` and `only_if` metaparamters will tell Chef to only take
action on the resource based on the conditional.

* Chef will *not* take action if `not_if` condition is true
* Chef will *only* take action if `only_if` condition is true.

The conditional can be passed a string or a ruby block. A string will
be executed as a command and considered `true` if the return code of
the command is 0. A ruby block will be evaluated as ruby code for `true`/`false`.

# Conditional execution

    @@@ruby
    execute "setenforce 1" do
      not_if "getenforce | grep -qx 'Enforcing'"
    end

    execute "apt-get update" do
      not_if { ::File.exists?('/var/lib/apt/periodic/update-success-stamp') }
      action :nothing
    end

.notes We use the :: in front of File due to Ruby namespacing. A
common guard is "grep -q" or even "grep -qx".

# Uncommon Resources

We're going to discuss a number of additional resources available, but
we won't explore them in this course. They are documented on the Chef Wiki.

* Application deployment
* Interacting with other services
* Filesystem management
* Network configuration
* Special resources

Documentation: http://wiki.opscode.com/display/Chef/Resources

# Application deployment

Full exploration of application deployment is beyond the scope of this course.

* `deploy` - capistrano style deployment
* `scm` - Git, Subversion supported (`git` and `subversion` resources)

# Interacting with other services

* `cron` - creates crontab entry
* `env` - sets system-wide Windows ENV variables (Windows only at this time)
* `erl_call` - make a call to an Erlang process
* `http_request` - makes an HTTP request, useful for interacting with
  HTTP-based APIs

# Filesystem management

* `mdadm` - software RAID on Linux systems
* `mount` - mounts a filesystem

# Network configuration

* `ifconfig` - manage network interfaces on Linux with ifconfig,
  add/delete actions work on most distributions, enable only works on
  EL family
* `route` - manage network routes on Linux with route, writing config
  only works on EL family

# Special resources

* `log` - outputs a log message during the execute phase using the
  Chef logger
* `ohai` - reloads node's automatic attributes, e.g. after deploying a
  new ohai plugin

# Advanced resources

* `breakpoint` - sets a breakpoint used by shef
* `ruby_block` - executes a block of Ruby code, not the same as the
  ruby script provider

.notes A ruby block can be used to promote execution from the compile
phase into the "execution" phase. Between these two, students will see
(and use) `ruby_block` most often.

# Running Chef

If we did kick off a Chef run (enter the execution phase), then all
the resources entered to this point would be configured as we wrote
them.

Many of the resource examples we used are incomplete - we do not have
source files for some of the templates and cookbook_files.

Chef will halt execution when it encounters an unhandled exception.

.notes the `shef` command to enter the execution phase is `run_chef`.

# Summary

* Understand the components of resources.
* Know the commonly used resources in Chef.
* Write recipes using common resources.

# Questions

* What are the components of a resource?
* What are the three resources for managing file contents? How do they
  differ?
* What resource manages running daemons on the node?
* How does Chef run arbitrary user commands on the system?
* What are three package providers for the package resource?
* Name at least two interpreters for script resources.
* Student questions?

# Additional Resources

* http://wiki.opscode.com/display/chef/Resources

# Lab Exercise

Resources In Depth

* Understand components of resources
* Write resources into shef and observe the outcome

# Interacting with Resources

Interact with the Ruby object of a resource by loading it with the
`resources()` method. With no parameters, `resources()` displays an
array of resources in the Resource Collection.

    chef:recipe > resources
    ["file[/tmp/shef_created_this]"]
    chef:recipe > my_resource = resources("file[/tmp/shef_created_this]")
    chef:recipe > my_resource.name
     => "/tmp/shef_created_this"
    chef:recipe > my_resource.path
     => "/tmp/shef_created_this"
    chef:recipe > my_resource.action
     => "create"
    chef:recipe > my_resource.allowed_actions
     => [:nothing, :create, :delete, :touch, :create_if_missing]
