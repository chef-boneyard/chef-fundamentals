# Data Bags

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribute Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Objectives

At completion of this unit you should...

* Understand what data bags are.
* Know how to create new data bags as JSON.
* Search data bags.
* Know how to access data bag items in a recipe.
* Use a simple data bag to manage users.

# Data Bags

Data bags are containers of “items” that are plain JSON data.

Items can contain any arbitrary key/value pairs, such as user information, application setup parameters or DNS entries.

They are centrally available to recipes for processing.

# Data Bags

Data bags are indexed separately for search.

They are not tied specifically to roles or nodes.

Use data bags for storing information that is “infrastructure-wide”.

# Creating Data Bags

Data bags go in the “data_bags” directory of the chef-repo.

Create a directory for the bag itself.

Put items in JSON files in the bag’s directory.

# Creating Data Bags

    % mkdir data_bags/users
    % touch data_bags/users/USERNAME.json
    % $EDITOR data_bags/users/USERNAME.json
    {
      "id": "USERNAME"
    }
    % knife data bag create users
    % knife data bag from file users USERNAME.json

# Search Data Bags

Each bag gets a new index created.

Search can be done just like nodes or roles with knife or in a recipe.

    knife search BAG QUERY
    search(:bag, "QUERY")

# Search Data Bags: Knife

    % knife search users "id:USERNAME"
    1 items found
    
    _rev:       6-c1c943b594f79bfae3ed043cca3a9de6
    chef_type:  data_bag_item
    data_bag:   users
    id:         USERNAME

Substitute USERNAME with your local username.

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

# data_bag

    @@@ruby
    users = data_bag("users")
    puts users #=> ["$USER"]

The results will be an array of all the item names, but not their
contents.

# data_bag_item

    @@@ruby
    user = data_bag_item("users", "USERNAME")
    puts user #=> data_bag_item[USERNAME]
    puts user['id'] #=> USERNAME

This returns a `data_bag_item` object. It is like a hash.

Access the item’s keys like hash keys.

# Managing Users

A common thing to manage with data bags is users.

Each user gets their own item in a data bag (often “users”).

Use a data bag search block to iterate over all users and have them get created on managed nodes.

Managing users in a data bag makes it easy to integrate with other parts of the infrastructure using Chef.

# Users Data Bag

We’ve already created a users data bag.

We have a user but it doesn’t have any useful data that we can use in a recipe.

# knife data bag show

    % knife data bag show users USERNAME

    {
      "id": "USERNAME",
      "groups": "sysadmin",
      "comment": "USERNAME",
      "uid": 2003,
      "shell": "/bin/bash",
      "ssh_keys": "ssh-rsa AAAAB3Nz...yhCw== USERNAME@localhost"
    }

# Users Cookbook

We’ll use the “users” cookbook with the “sysadmins” recipe to apply on the existing nodes.

This recipe uses a search for users in the group “sysadmin”.

# users::sysadmin

    @@@ruby
    sysadmin_group = Array.new
    search(:users, 'groups:sysadmin') do |u|
      sysadmin_group << u['id']
      # ...
      user u['id'] do
        uid u['uid']
        gid u['gid']
        shell u['shell']
        comment u['comment']
        supports :manage_home => true
        home home_dir
      end
      # ...
    end

# chef-client output

    INFO: Processing user[USERNAME] action create (users::sysadmins line 41)
    INFO: user[USERNAME] created

When running chef-client on the node, we’ll see that the user is created, amongst other output from the recipe.

# Summary

You should now be able to...

* Describe data bags
* Create new data bags as JSON.
* Search data bags.
* Access data bag items in a recipe.
* Use a simple data bag to manage users.

# Additional Resources

* http://wiki.opscode.com/display/chef/Data+Bags
* http://wiki.opscode.com/display/chef/Encrypted+Data+Bags
* http://json.org/

# Lab Exercise

## Data Bags
