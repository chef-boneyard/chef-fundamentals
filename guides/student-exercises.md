# Unit 1 - Getting Started

In this exercise, we will:

* Verify the Ruby and Chef installation.
* Create a Chef repository.
* Copy the configuration to the repository.
* Clean up changes to the repository in Git.
* Explore the chef-repo and Chef commands.


## Verify Ruby and Chef Installation

Before the class, students will have either installed Ruby, RubyGems
and the Chef gem on their workstation according to the workstation
setup guide, or will be set up to use an Amazon AWS EC2 instance
running Ubuntu 10.04 preconfigured. Verify the installation:

    ruby -v
    gem -v
    knife -v

    > ruby -v
    ruby 1.9.2p180 (2011-02-18 revision 30909) [x86_64-darwin10.6.0]

    > gem -v
    1.7.2

    > knife -v
    Chef: 0.10.4

Versions should be these or higher.

## Create Chef Repository

The first step to building infrastructure as code with Chef is to
create a source code repository. We're going to use Git with Opscode's
skeleton starter repository, because the default tools integrate with
it already. Don't worry if you aren't familiar with Git, we're going
to walk through the steps required.

    > git clone git://github.com/opscode/chef-repo.git chef-repo

Change your current working directory to chef-repo and create a .chef
sub-directory. This is where knife configuration and the certificates
will live. From now on, when we refer to the `chef-repo`, use this
directory.

Next, let's work from the Repository, and copy the configuration and
certificates required for interacting with Opscode Hosted. Replace
ORGNAME and USERNAME with your values.

Copy the files downloaded in Getting Starte Unit 1 Lab Part 1 to the
.chef directory.

    knife.rb
    ORGNAME-validator.pem
    USERNAME.pem

The content of `knife.rb` should look like the following.

    current_dir = File.dirname(__FILE__)
    log_level                :info
    log_location             STDOUT
    node_name                "USERNAME"
    client_key               "#{current_dir}/USERNAME.pem"
    validation_client_name   "ORGNAME-validator"
    validation_key           "#{current_dir}/ORGNAME-validator.pem"
    chef_server_url          "https://api.opscode.com/organizations/ORGNAME"
    cache_type               'BasicFile'
    cache_options( :path => "#{ENV['HOME']}/.chef/checksums" )
    cookbook_path            ["#{current_dir}/../cookbooks"]

USERNAME will be your Opscode login name and ORGNAME will be the short
name of the Organization you created. From now on, whenever you are in
the chef-repo directory, Knife will use your Opscode Hosted Chef user
account.

## Clean Up Git Repository

Since we have modified the repository by adding files, it is no longer
a pristine copy of the repository we cloned. You can commit the
changes to the repository and save them, or you can choose to ignore
select files in the .chef directory. It is common to share
repositories amongst several people in an organization, so we might
want to be flexible and let others use their own files.

Run `git status` to show that the repository has uncommitted changes:

    > git status
    # On branch master
    # Untracked files:
    #   (use "git add <file>..." to include in what will be committed)
    #
    #	.chef/
    nothing added to commit but untracked files present (use "git add" to track)

We can ignore files with a `.gitignore` in the root of the
repository. The validation certificate is one that we probably want
everyone to be able to access.

Modify the `.gitignore` file in the chef-repo directory so its contents
look like this:

    .rake_test_cache
    .chef/

Now add the .gitignore file to changes to commit and commit it to the repository.

    > git add .gitignore
    > git commit -m "ignore the .chef directory"

Now we'll add the validation certificate and commit it.

    > git add .chef/ORGNAME-validator.pem
    > git commit -m 'add the validation certificate'
    > git status
    # On branch master
    # Your branch is ahead of 'origin/master' by 2 commits.
    #
    nothing to commit (working directory clean)

The message about being ahead of origin/master is because we cloned
the Opscode repository from GitHub. For now we will ignore that
message.

> In a normal work environment, we might change the remote
> origin to an internal git server we manage, or maybe
> even host the code with GitHub in a private repository.

The repository is now clean. We can check connectivity to Opscode
Hosted Chef by running some Knife commands with it. We will discuss
knife's "client" sub-command later.

    > knife client list

Should return a single client named "`ORGNAME-validator`" where
ORGNAME is your organization's short name.

## Explore Chef Commands

When Chef is installed, it comes with a number of commands.

* ohai
* chef-client
* chef-solo
* knife
* shef

Each command supports `-h` or `--help` to get more information about
its usage. Knife also has a `help` sub-command that can be used to
bring up Unix/Linux style `man(1)` pages for its other
sub-commands. We will use Knife extensively in this course, and we
will use chef-client (but not chef-solo) quite a lot as well, so
become familiar with the command-line output, options and usage.

     > ohai -h
     > chef-client -h
     > knife -h
     > knife help
     > knife help list

On Windows platforms, the man pages can be viewed as HTML in your
favorite web browser. They will be located in the `distro/common/html`
directory where the Chef gem is located. Use `gem environment gemdir`
to get the location where the Chef gem is installed.

    > gem environment gemdir
    C:/opscode/chef/embedded/lib/ruby/gems/1.9.1

We will also refer to the help output later on as we introduce new
sub-commands for knife. For example, earlier we used `knife client
list`. We can get help for the `client` sub-commands:

     > knife client -h

We can also see the Unix `man(1)` page with `knife help`:

     > knife help client

We will introduce new knife commands in each Unit's exercises like this.

On your local system, go ahead and run ohai. You might want to pipe it
through `more`.

    > ohai | more

A lot of information is gathered about your system automatically. This
kind of data is gathered when Chef runs, too.

## Explore Chef Repository

The Chef Repository we cloned earlier contains a number of files and
directories. Knife is preconfigured to work with a variety of
these. Go ahead and take a look around!

On Linux / Unix, use the tree command if it is installed, or use find.

    > tree
    .
    |-- README.md
    |-- Rakefile
    |-- certificates
    |   `-- README.md
    |-- chefignore
    |-- config
    |   `-- rake.rb
    |-- cookbooks
    |   `-- README.md
    |-- data_bags
    |   `-- README.md
    |-- environments
    |   `-- README
    `-- roles
        `-- README.md

    6 directories, 9 files

    > find *
    README.md
    Rakefile
    certificates
    certificates/README.md
    chefignore
    config
    config/rake.rb
    cookbooks
    cookbooks/README.md
    data_bags
    data_bags/README.md
    environments
    environments/README.md
    roles
    roles/README.md

On Windows, use the tree command, or navigate to the `chef-repo`
folder in Explorer.

    > tree
    .
    |-- README.md
    |-- Rakefile
    |-- certificates
    |   `-- README.md
    |-- chefignore
    |-- config
    |   `-- rake.rb
    |-- cookbooks
    |   `-- README.md
    |-- data_bags
    |   `-- README.md
    |-- environments
    |   `-- README
    `-- roles
        `-- README.md

The knife sub-commands associated with data_bags, environments and
roles automatically look in those directories to load files. We will
get to them in later units.

# Unit 2 - Anatomy of a Chef Run

In this exercise, we will:

* Create a local Chef configuration for our workstation.
* Run chef-client in debug mode.
* Review the output log as a group.


## Knife Commands

This exercise will introduce the following Knife sub-commands:

    > knife configure -h
    > knife help configure
    > knife node -h
    > knife help node
    > knife client -h
    > knife help client

## Create a Local Managed Node

We don't have a managed node yet, so we're going to create one on the
local workstation system. Some Chef users choose to manage their
workstations with Chef, but that is not what we're going to do
here. We're simply going to run Chef to observe the anatomy of a Chef
run. We can delete the node later.

We will use the `config` directory in the `chef-repo`. Use Knife to
create a configuration file, and then run `chef-client` using that
configuration file.

    > knife configure client config/

If your workstation is Windows, some settings must be added to
`config/client.rb`. Edit the file with a text editor and add the
following lines:

    client_key "#{ENV['HOME']}/chef-repo/config/client.pem"
    validation_key "#{ENV['HOME']}/chef-repo/config/validation.pem"
    checksum_path "#{ENV['HOME']}/.chef/checksums"
    file_cache_path "#{ENV['HOME']}/.chef/cache"

Now run `chef-client` with the configuration file:

    > chef-client -l debug -L debug.log -c config/client.rb

We will review the output file (debug.log) of this command as a group.

We can also inspect the node object and client object on Opscode
Hosted Chef with Knife. Use the `node` and `client` sub-commands.

## Questions

Use the debug output and knife commands to answer the following questions:

1. What is the name of the node created by Chef?



2. What is the platform of the node?



3. Does it have anything in its run list?



4. What is the IP address?



5. What is the name of the client?



6. Is the client an admin?



# Unit 3 - Example Cookbook

In this exercise, we will:

* Download the fail2ban cookbook.
* Explore the fail2ban cookbook's contents.
* Upload the cookbook to Opscode Hosted Chef.
* Use `knife bootstrap` to run chef on a new node with the cookbook.


## Knife Commands

This exercise will introduce the following Knife sub-commands:

    > knife cookbook site -h
    > knife help cookbook site
    > knife cookbook -h
    > knife help cookbook
    > knife bootstrap -h
    > knife help bootstrap

## Download fail2ban Cookbook

We're going to retrieve the fail2ban cookbook from the Chef Community
Site using Knife's "vendor branch" pattern. This command will
"install" the cookbook in the cookbooks directory and integrate it
with the Git repository. Use the `knife cookbook site` commands to
explore the Community Site through its API.

    > knife cookbook site search fail2ban
    > knife cookbook site show fail2ban

If you are not using Git, skip down to "Manual cookbook download"
below. Otherwise, we install cookbooks into the local repository using
Knife's "vendor branch" pattern. This command will "install" the
cookbook in the cookbooks directory and integrate it with the Git
repository.

    > knife cookbook site install fail2ban

The preferred and recommended way to get new cookbooks in your
repository is to retrieve them from the community site, rather than
trying to find them in forked repositories on GitHub.

Once you have downloaded the cookbook, take a look at the recipe, and
other parts of it. When downloading cookbooks, if they have a README
file be sure to read that as it may contain important information
about using the cookbook. While the fail2ban cookbook doesn't have
any, some cookbooks may require specific attribute settings.

The recipe is in the `chef-repo` directory under
`cookbooks/fail2ban/recipes/default.rb`.

## Review Changes to Git Repository

The `cookbook site install` sub-command will make a number of changes
to the Git repository. We can review these changes with git.

    > git log
    > git tag
    > git branch
    > git checkout chef-vendor-fail2ban
    > git checkout master

If you feel adventurous, you can use `git diff` to compare the master
branch with other branches or tags, too.

## Manual Cookbook Download

To download a cookbook from the community site and extract it to the
cookbooks directory so Knife can find it for upload, run the following
commands:

    > knife cookbook site download fail2ban
    > tar -zxvf fail2ban-1.0.0.tar.gz -C cookbooks

Make note of these commands, we will download more cookbooks later in
this course, and you'll need to run these to download them.

Then use your version control system's commands to commit / save
changes to the local repository and create branches and tags as
required for your workflow. You can then remove the .tar.gz file that
knife downloaded.

## Upload Cookbook and Apply to Node

Now that we have downloaded a cookbook, integrated it into our Git
repository, and reviewed its contents, let's apply it to a
system. First, let's upload it to Opscode Hosted Chef.

    > knife cookbook upload fail2ban

You can then review the cookbook on Opscode Hosted Chef with:

    > knife cookbook show fail2ban
    > knife cookbook show fail2ban latest
    > knife cookbook show fail2ban latest recipes default.rb

Your instructor will provide you with the IPADDRESS of a system that
you can SSH to and "bootstrap" to apply the recipe. The following
command should be entered as a single line:

    > knife bootstrap IPADDRESS --sudo -x ubuntu -r "recipe[fail2ban]" -Popstrain_0150

Once the system is finished running Chef, you should see lines similar to the following:

    INFO: Chef Run complete in 10.57275 seconds
    INFO: Running report handlers
    INFO: Report handlers complete

Now use the `knife node` sub-commands (list, show) to determine the
node that was created and take a look at it. Some interesting things
to look at:

    > knife node show NODENAME -r
    > knife node show NODENAME -a chef_packages

# Unit 4 - Roles

In this exercise, we will:

* Create a new "base" role.
* Move the fail2ban recipe to the base role.
* Apply the role to a node and run Chef.

## Knife Commands

This exercise will introduce the following Knife sub-commands:

    > knife role -h
    > knife help role
    > knife ssh -h
    > knife help ssh

## Create the Base Role

First, create a new role named `base`. We will use the Role Ruby DSL,
and the file will be `chef-repo/roles/base.rb`. It should looke like this:

    name "base"
    description "Base role is applied to all nodes"
    run_list(
      "recipe[fail2ban]"
    )

Next, upload the role to Opscode Hosted Chef.

    > knife role from file base.rb

## Apply the Role

Get the node name of the system we bootstrapped earlier and review its run list.

    > knife node list
    > knife node show NODENAME -r

Remove the fail2ban recipe and apply the base role to the node. Use
single quotes.

    > knife node run list remove NODENAME 'recipe[fail2ban]'
    > knife node run list add NODENAME 'role[base]'

Use knife to SSH to the system and run chef-client. The option `-a
ec2.public_hostname` should be used as written, it tells `knife ssh`
to open the connection to the value of that attribute. Type the
following command as written on a single line:

    > knife ssh "role:base "sudo chef-client" -x ubuntu -Popstrain_0150 -a ec2.public_hostname

This sub-command will be used later for running chef-client. You can review the man page and help output of the `knife ssh` command to find out other ways that the sub-command can be invoked and used.

# Unit 5 - Adding Cookbooks

In this exercise, we will:

* Download some new cookbooks.
* Apply the new cookbooks to the `base` role.
* Create a new cookbook for deploying our web server.
* Apply the web server configuration with a new role.


## Knife Commands

This exercise does not introduce any new Knife sub-commands, but does use a new sub-command of `knife cookbook`, "create".

    > knife cookbook create -h

## Download New Cookbooks

We're going to download some additional cookbooks. The node that we
have been working with is going to be a web server, so we will use
apache2. We want to make sure that the Ubuntu package cache is up to
date, so we will use apt. Finally, we want to set up the chef-client
to run as a system service, and will use chef-client. If you're using
Git for the chef-repo, use the install command:

    > knife cookbook site install apache2
    > knife cookbook site install apt
    > knife cookbook site install chef-client

If you're not using Git, download the cookbooks and extract their
contents in `cookbooks/`:

    > knife cookbook site download apache2
    > knife cookbook site download apt
    > knife cookbook site download chef-client

    > tar -zxvf apache2-1.0.0.tar.gz -C cookbooks
    > tar -zxvf apt-1.2.0.tar.gz -C cookbooks
    > tar -zxvf chef-client-1.0.2.tar.gz -C cookbooks

Replace the filenames with the ones downloaded if the version numbers
are not the same.

Add the following recipes to the `base` role before the
`recipe[fail2ban]` in the run_list.

* `recipe[apt]`
* `recipe[chef-client]`

The run_list is a comma-separated list.

Don't put the apache2 recipe in the run_list of the base role. We will
create a specific role for it later.

## Create Web Server Cookbook

This example cookbook will be used to illustrate how we might create a
cookbook that performs a particular purpose, such as deploying our web
site/application. In this case, the web site is very simple, a single
index.html page, created from a template.

Create the web server cookbook. Refer to the help output of the
sub-command for other options that can be passed.

    > knife cookbook create webserver

Create a `default.rb` recipe in the cookbook. It should include the
apache2 (default) recipe and set up a template resource for the
index.html page for the default Apache 2 virtual host.

    include_recipe "apache2"

    template "/var/www/index.html" do
      source "index.html.erb"
      owner node["apache"]["user"]
      group node["apache"]["user"]
      mode 0644
    end

Next, create the template for the index.html page. This template will
use some node attributes rendered by ERB when Chef creates it on the
node.

Save the following as
`chef-repo/cookbooks/webserver/templates/default/index.html.erb`.

    <html>
      <head>
        <title>Welcome to <%= node['hostname'] %></title>
      </head>
      <body>
        You have reached:
        <ul>
          <li><b>FQDN:</b> <%= node['fqdn'] %></li>
          <li><b>Public FQDN:</b> <%= node['ec2']['public_hostname'] %></li>
          <li><b>Platform:</b> <%= node['platform'] %></li>
          <li><b>Platform Version:</b> <%= node['platform_version'] %></li>
          <li><b>Run List (roles):</b> <%= node.run_list.roles.join(", ") %></li>
        </ul>
      </body>
    </html>

Because we're including the apache2 default recipe in the webserver
default recipe, instead of on the node's run list, we need to declare
a dependency on the apache2 cookbook in the webserver cookbook's
metadata. Add the depends line to
`chef-repo/cookbooks/webserver/metadata.rb`.

    maintainer       "YOUR_COMPANY_NAME"
    maintainer_email "YOUR_EMAIL"
    license          "All rights reserved"
    description      "Installs/Configures webserver"
    long_description IO.read(File.join(File.dirname(__FILE__), 'README.rdoc'))
    version          "0.0.1"
    depends "apache2"

We will apply the webserver recipe with a role named webserver. Create
the role as `chef-repo/roles/webserver.rb`.

    name "webserver"
    description "Systems that serve HTTP requests"
    run_list(
      "recipe[webserver]"
    )

Upload the cookbook and the role to Opscode Hosted Chef. We can then
take a look at information about the cookbook on Opscode Hosted Chef
and see that it is correct.

    > knife cookbook upload -a
    > knife role from file webserver.rb
    > knife role from file base.rb
    > knife cookbook list
    > knife cookbook show webserver
    > knife cookbook show webserver 0.0.1
    > knife cookbook show webserver latest recipes

Apply the role to the node we have been working with already and use
"knife ssh" to run Chef. The `knife ssh` command should be entered as
shown (with single quotes) on a single line.

    > knife node run list add NODENAME 'role[webserver]'
    > knife ssh "role:webserver" "sudo chef-client" -x ubuntu -Popstrain_0150 -a ec2.public_hostname

View the node data on Opscode Hosted Chef to see the run list. We can
retrieve just the EC2 public hostname, and navigate to that in a web
browser.

    > knife node show NODENAME -r
    > knife node show NODENAME -a ec2.public_hostname

Navigate to the web page:

* http://ec2-public-hostname-from-above/

# Unit 6 - Cookbook Attributes

In this exercise, we will:

* Add attributes to the web server cookbook and modify the template to display them.
* Modify the webserver role to get a different value.

## Knife Commands

This exercise does not introduce any new Knife sub-commands.

## Create Attributes File

The cookbook creation command in the previous exercise also created an
"attributes" directory in the webserver cookbook. We will create a
default.rb, `chef-repo/cookbooks/webserver/attributes/default.rb` with
the following content:

    default["webserver"]["origin"] = "This came from the cookbook"

This sets a "default" level attribute, which is low priority and can
be overridden easily elsewhere.

Upload the cookbook, then view the webserver node, specifying the
attribute. Is it set?

    > knife cookbook upload webserver
    > knife node show NODENAME -a webserver.origin

Now run chef-client on the node like we did in the last unit's
exercise, then run the above `knife node show` command again. Did it
change?

    > knife ssh "role:webserver" "sudo chef-client" -x ubuntu -Popstrain_0150 -a ec2.public_hostname

Next we will override the setting of the attribute from the
cookbook with a setting in the role. We'll do this by adding a
`default_attributes` block in the `roles/webserver.rb` file.

    name "webserver"
    description "Systems that serve HTTP requests"
    run_list(
      "recipe[webserver]"
    )
    default_attributes(
      "webserver" => {
        "origin" => "This came from the role"
      }
    )

Upload the role and view the node again.

    > knife role from file webserver.rb
    > knife node show NODENAME -a webserver.origin

Now run chef-client on the node again and view the attribute.

    > knife ssh "role:webserver" "sudo chef-client" -x ubuntu -Popstrain_0150 -a ec2.public_hostname

# Unit 7 - Introduction to Search

In this exercise, we will:

* Use knife to search nodes and roles.
* Explore options for retrieving a specific attribute or ID.

## Knife Commands

This exercise will introduce the following Knife sub-command:

    > knife search -h
    > knife help search

## Searching Nodes

    > knife search node "role:webserver"
    > knife search node "platform:ubuntu"
    > knife search node "platform:ubuntu" -a platform_version
    > knife search node "*:*" -i
    > knife search node "*:*" -r

## Searching Roles

    > knife search role "default_*_webserver_origin:This"
    > knife search role "run_list:*webserver*"

## Questions

Use knife search queries to answer the following questions. Query key hints are provided in the question.

1. How many nodes are running Ubuntu? (platform)



2. What is the run list of all the nodes?



3. What is the node name of systems that are webservers? (role)



4. Do any roles have a recipe in the apache2 cookbook in their run list?



5. Do any nodes have a recipe from the apache2 cookbook? (recipes)



# Unit 8 - Adding a New Node

In this exercise, we will:

* Download a new cookbook and create a role to use it.
* Bootstrap a new node to become a load balancer.

## Knife Commands

This exercise does not introduce any new Knife sub-commands.

## Download Cookbook for Load Balancer

First, we're going to download the cookbook that will be used to apply
the load balancer configuration with the haproxy software. After
downloading the cookbook, inspect the `app_lb.rb` recipe, and examine
what is different between it and the default recipe.

    > knife cookbook site install haproxy

Or, if you're not using Git:

    > knife cookbook site download haproxy
    > tar -zxvf haproxy-1.0.3.tar.gz -C cookbooks

After downloading the cookbook, view the `app_lp.rb` and examine what
is different between it and the `default.rb` recipe.

Next, create an `lb` role that will be used on systems that will
balance the load to our webserver. The file `chef-repo/roles/lb.rb`
should look like this:

    name "lb"
    description "Systems that balance the load"
    run_list(
      "recipe[haproxy::app_lb]"
    )
    default_attributes(
      "haproxy" => {
        "member_port" => "80"
      }
    )

Upload the cookbook, load the role and then bootstrap the new instance
provided by the instructor.

    > knife cookbook upload haproxy
    > knife role from file lb.rb
    > knife bootstrap IPADDRESS --sudo -x ubuntu -Popstrain_0150 -r "role[base],role[lb]"
    > knife ssh 'role:lb' 'cat /etc/haproxy/haproxy.cfg' -x ubuntu -Popstrain_0150 -a ec2.public_hostname

Access the haproxy system's admin console, which runs on port 22002 by
default. Retrieve the `ec2.public_hostname` value from the node's attributes.

* http://ec2.public_hostname:22002/
* http://ec2.public_hostname/

The result should be the same as going to the webserver's public
hostname directly.

# Unit 9 - Data Bags

In this exercise, we will:

* Download a cookbook for managing users.
* Create a users data bag item.
* Apply the user recipe in the base role and run Chef on all nodes to create users.

## Knife Commands

This exercise introduces the following Knife sub-commands:

    > knife data bag -h
    > knife help data bag

## Download Users Cookbook

We're going to use the users cookbook published by Opscode to manage
our user on the target systems. We'll add the `users::sysadmins`
recipe to the base role.

    > knife cookbook site install users

If you're not using Git:

> knife cookbook site download users
> tar -zxvf users-.tar.gz -C cookbooks

Modify the `base.rb` role file and add "`recipe[users::sysadmins]`" to the
end of the run_list and upload the base role.

## Generate User Data Bag Item

We're going to create a new user data bag item as JSON. Type the
following into a text editor and save it as
`chef-repo/data_bags/users/USERNAME.json`. Replace USERNAME in the
filename and the content with your username. Create the users
directory in `chef-repo/data_bags` if it doesn't exist.

    {
      "id": "USERNAME",
      "groups": "sysadmin",
      "comment": "USERNAME",
      "uid": 2003,
      "shell": "/bin/bash",
      "ssh_keys": "ssh-rsa SSH public key info from opstrain.pubt"
    }

## Upload Data Bag Item

First create the users data bag, then upload the user data bag item.

    > knife data bag create users
    > knife data bag from file users USERNAME.json

## Apply and Verify

Apply the changes to all nodes that have the base role in their run
list with knife ssh, then verify login to one of the systems.

    > knife ssh 'role:base' 'sudo chef-client' -x ubuntu -Popstrain_0150 -a ec2.public_hostname

Use knife ssh to log into the system and view the user's
Chef-generated authorized_keys file. Replace USERNAME with the value
you used in the data bag item.

    > knife ssh 'role:base' 'sudo cat ~USERNAME/.ssh/authorized_keys' -x ubuntu -Popstrain_0150 -a ec2.public_hostname

# Unit 10 - Troubleshooting

In this exercise, we're simply going to simulate a common failure as
described in the discussion slides - a cookbook that does not exist
appears in the run list of a node.


## Delete Cookbook

First, delete the fail2ban cookbook from Opscode Hosted Chef, then
attempt to run chef-client on the node.

    > knife cookbook delete fail2ban
    > knife ssh 'role:base' 'sudo chef-client' -x ubuntu -Popstrain_0150 -a ec2.public_hostname

What error occurred?

## Review Stack Trace

Log into the system to review the stack trace reported back from the
failed Chef run.

    > ssh ubuntu@IPADDRESS
    $ cat /var/chef/cache/chef-stacktrace.out
    $ sudo chef-client -l debug

Did the chef-client output report any other data artifacts? What are
they?

## Re-upload Cookbook

Fix the problem by uploading the cookbok and re-run Chef.

    > knife cookbook upload fail2ban
    > knife ssh 'role:base' 'sudo chef-client' -x ubuntu -Popstrain_0150 -a ec2.public_hostname

# Appendix A. Course Artifacts

Congratulations! You have completed the Opscode Chef Fundamentals
course student exercises. You should now have a complete Chef
repository that you can now start using in your own environment to
manage infrastructure. Modify the repository by downloading new
cookbooks, creating your own, and creating roles and data bags.

Opscode Hosted Chef customers can contact Opscode support through the
Help site, or email Support.

* http://help.opscode.com
* support@opscode.com

Users of the Open Source Chef Server can get help from the community
through the mailing lists, IRC, and the Opscode Chef Community Site.

* irc.freenode.net #chef
* http://lists.opscode.com
* http://community.opscode.com

Opscode's open source projects have a ticket tracker available.

* http://tickets.opscode.com
