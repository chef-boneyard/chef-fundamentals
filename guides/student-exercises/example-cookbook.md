Example Cookbook
======================

## Objectives

* Download an existing cookbook
* Explore the cookbook's contents
* Upload cookbook to the Chef Server
* Install and run Chef on a new system with the cookbook

## Acceptance Criteria

* `fail2ban` cookbook extracted in local repository
* `fail2ban` cookbook uploaded to Chef Server
* Target system has Chef installed
* Target system has node and client objects in Chef Server
* Target system applied `fail2ban` cookbook successfully
* Answer the questions

## Download fail2ban Cookbook

Use knife to download the `fail2ban` cookbook from the Chef Community site. The downloaded file is a `.tar.gz` that should be extracted into the `cookbook_path` directory

## Explore Cookbook

After extracting the cookbook, examine the contents of the files it contains. This information will be used to answer questions.

## Upload Cookbook

Upload the cookbook to the Chef Server.

## Install Chef on Target

Your instructor will provide an IP address and login credentials for a target system. Log into that system and perform the following:

* Install Chef (use the [full-stack installer](htp://opscode.com/chef/install))
* Configure `chef-client` with the `client.rb` file.
* Copy the validation certificate file to the configuration directory.

## Run chef-client on Target

Create a JSON file on the node to specify that `fail2ban` should be in the run list. Run `chef-client` using the JSON file with the `-j` option.

## Questions

What `knife` commands can be used to gather information about the cookbook from the Community Site?


What version of the `fail2ban` cookbook was downloaded? What versions are available?


How many recipes does the `fail2ban` cookbook have? What is its name?


What entry goes in the node's run list to apply the `fail2ban` cookbook?


What resources does the `fail2ban` cookbook configure on the node?


Where did Chef cache the cookbook?
