More Cookbooks
======================

## Objectives

* Apply apt, chef-client, and fail2ban recipes via a base role
* Include the apache2 recipe in the webserver recipe
* Download and examine the haproxy cookbook

## Acceptance Criteria

* "base" role is created with the apt, chef-client and fail2ban
  recipes.
* "base" role is applied to the remote target system.
* Answer the questions

## Download cookbooks

Download the following cookbooks to the local repository and extract
them in the `cookbooks` directory.

* apt
* chef-client
* fail2ban
* apache2

With a single knife command, upload all four cookbooks.

## Create base role

In the roles directory, create a `base.rb` for the base role. It
should have a run list containing the `apt`, `fail2ban` and
`chef-client` recipes.

    recipe[apt],
    recipe[fail2ban],
    recipe[chef-client]

## Add apache2 to webserver cookbook

Use `include_recipe` in the `webserver` cookbook's default recipe to
add the `apache2` default recipe.

Update the `webserver` cookbook's `metadata.rb` file to depend on the
`apache2` cookbook.

## Update the roles

Upload both the `base` and `webserver` roles to the Chef Server.

## Add the base role

Add the base role to be the first item in the remote target node's run list.

Run `chef-client` on the remote target node.

## Download haproxy cookbook

Download and extract the haproxy cookbook to the `cookbooks`
directory. Examine its contents but do not upload it to the Chef
Server yet. Do not apply it to the node's run list, that will be done
in the next exercise.

## Questions

What knife command was used to download the cookbooks from the Chef
Community Site?

What knife command was used to upload the cookbooks to the Chef
Server?

How many recipes are in the chef-client cookbook? Which one is used by
the default recipe?

How many recipes are in the apache2 cookbook? Are any of them used in
the default recipe?

Are dependencies required in the metadata for the chef-client or
apache2 cookbook?

Show the remote target node on the Chef Server with knife to answer
the following questions.

What values does it have for the `roles` and `recipes` attributes? Are
they different from earlier exercises?

What is in the node's attribute space for `apache` and `chef_client`?
