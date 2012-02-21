# Lab Exercise: Anatomy of a Chef Run

Lab objectives

* Configure remote target to run `chef-client`
* Successful `chef-client` run with debug logging

Acceptance Criteria

* Client and node objects exist on Chef Server
* Debug log output from `chef-client`
* Answer the questions

# Prepare Remote Target

You will be issued with a remote system.  Connect with the credientials provided.

Install Chef onto the system.

# Configure Chef Client

Create a Chef client configuration which will allow your node to speak to the Chef server, in readiness for being managed using Chef.

# Run Chef

Run `chef-client` on the local system with debug logging, and send the output to a file. Use the output file, along with the command-line tools to answer the following questions.

# Questions

1. What is the name of the node and client created on the Chef Server?
1. What commands can be used to get these values?
1. What are two ways to change the name of the node and client at `chef-client` run time?
1. What are the platform and platform version of the node?
1. Does the node have a run list?
1. What is the IP address detected for the node? Is it the correct default IP address?
1. Is the API client an admin?
1. Does the validation key file still exist? Why?
1. What kind of HTTP request is made to save the node? When does this occur?
