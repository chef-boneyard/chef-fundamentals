# Anatomy of a Chef Run

.notes These course materials are Copyright © 2010-2011 Opscode, Inc. All rights reserved. 
This work is licensed under a Creative Commons Attribution Share Alike 3.0 License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Objectives

At completion of this unit you should...

* Understand Chef API Clients
* Understand Chef nodes
* Be able to query and manipulate node run lists
* Understand Chef node convergence phases
* Understand the difference between notification handler types

# Anatomy of a Chef Run

<center><img src="anatomy-of-chef-run.png"></center>

.notes This diagram represents the process of running chef.

# Anatomy of a Chef Run

* Build the node
* Synchronize cookbooks
* Compile resource collection
* Configure the node
* Run notification handlers

# Build the Node

Before anything else happens, the system is profiled with Ohai.

Unless a node name was specified (node_name in /etc/chef/client.rb or chef-client -N), Chef will use the fully qualified domain name.

Node names should be unique.

# API Clients

API Clients authenticate with the Chef Server.

Chef uses Signed Header Authentication across all API requests. Portions used:

* HTTP method (GET/PUT/POST/DELETE)
* Request body in Base64
* Timestamp (use ntp!)
* Client ID (node name)

# API Authentication

Does /etc/chef/client.pem exist?

* Use it to sign requests

Does /etc/chef/validation.pem exist?

* Request a new API client key
* Or fail

Was a new client key generated?

* Use it to sign requests

# API Authentication Process

<center><img src="authn-flow.png" height="454" width="879"></center>

# Users are Special API Clients

With Opscode Hosted Chef, individual users have special API clients.

Users are associated to an organization with Role-Based Access Control.

Users are granted privileges to Server-side objects with access control lists.

Users are global to Opscode Hosted Chef. Clients (like systems running chef-client) are specific per organization.

# Node Objects

After the client has authenticated with the Server, Chef retrieves the node object from the server.

Node objects represent a set of data called attributes and a list of
configuration to apply called a run list.

# Node Object

Chef::Node is like a Hash, but it is internally a collection of hashes.

* Display order is arbitrary.
* Priority handled internally in Chef.

# Node Object: JSON

    @@@javascript
    {
      "name": "i-d51b00bf",
      "json_class": "Chef::Node",
      "chef_type": "node",
      "chef_environment": "_default",
      "default": { ... },
      "normal": { ... },
      "override": { ... },
      "automatic": { ... },
      "run_list": [ ... ]
    }

# Node Run List Expands

The run list can contain recipes and roles. Roles can contain recipes and also other roles.

Chef expands the node’s run list down to the recipes. The list roles
and recipes are saved as attributes, too.

# Synchronize Cookbooks

Chef downloads from the Chef Server all the cookbooks that appear as recipes in the node’s run list.

Chef also downloads all cookbooks that are listed as dependencies which might not appear in the run list.

If the node's "chef_environment" specifies cookbook versions, the Chef
downloads the version specified. Otherwise the latest available
version is downloaded.

# Cookbook Metadata

If a recipe from another cookbook is included in a recipe, it isn’t automatically downloaded.

Some cookbooks don’t actually have recipes, and instead provide helper code, libraries or other assets we want to use. 

To ensure that cookbooks that have components we need, we declare
dependencies in cookbook metadata.

# Cookbook Cache

Cookbooks are stored on the local system in the directory configured by "`file_cache_path`". The default is `/var/chef/cache` unless changed in `/etc/chef/client.rb`.

Cookbooks that have not changed are not downloaded again, the cached copy will be used.

# Chef Run

This starts when you see:

    INFO: Starting Chef Run for NODE_NAME

The run context is created with the node and the cookbook collection.

# Load Cookbooks

Once the cookbooks are synchronized to the local system, their
components are loaded in the following order:

* Libraries
* Providers
* Resources
* Attributes
* Definitions
* Recipes

# Cookbook Files and Templates

Cookbook static assets (files) and dynamic assets (templates) are not retrieved or loaded at this time.

They are retrieved from the server and rendered when needed by a resource in a recipe.

# Node Convergence

Convergence is when the configuration management system brings the
node into compliance with policy.

In other words, the node is configured based on the roles and recipes
in its run list.

Convergence in Chef happens in two phases.

* Compile
* Execute

# Convergence: Compile

Compile phase

* Chef recipe Ruby DSL is evaluated
* Ruby code is executed directly
* Recognized resources are added to the Resource Collection

# Convergence: Execute

Execute phase

* Chef runs the specified actions for each resource
* Providers know how to perform the actions

# Report and Exception Handlers

At the end of the Chef Run, report and exception handlers are triggered.

* Report handlers run when Chef exits cleanly
* Exception handlers run when Chef exits abnormally with an unhandled exception.

# Report Handlers

Normal, clean exit:

    INFO: Chef Run complete in 42.72288 seconds
    INFO: Running report handlers
    INFO: Report handlers complete

# Exception Handlers

Abnormal exit from unhandled exception:

    ^CFATAL: SIGINT received, stopping
    FATAL: SIGINT received, stopping
    ERROR: Running exception handlers
    ERROR: Exception handlers complete

# Summary

You should now be able to...

* Describe Chef API Clients
* Describe Chef nodes
* Be able to query and manipulate node run lists
* Describe the two Chef node convergence phases
* Describe the difference between report and exception handlers

# Additional Resources

* http://wiki.opscode.com/display/chef/Anatomy+of+a+Chef+Run
* http://wiki.opscode.com/display/chef/Authentication
* http://wiki.opscode.com/display/chef/Chef+Client
* http://wiki.opscode.com/display/chef/Nodes
* http://wiki.opscode.com/display/chef/Attributes
* http://wiki.opscode.com/display/chef/Evaluate+and+Run+Resources+at+Compile+Time

# Lab Exercise

## Anatomy of a Chef Run
