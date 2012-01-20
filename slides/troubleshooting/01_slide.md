# Troubleshooting

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribute Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Objectives

At completion of this unit you should...
Understand how to debug Chef’s Ruby stack traces
Recognize common HTTP status codes from the Chef Server
Know how to turn on debug logging to get more information
Be able to handle errors gracefully in your recipes
Describe the exception handler functionality

# Chef Log Locations

    @@@ruby
    # in /etc/chef/client.rb
    log_location STDOUT
    log_location "/var/log/chef/client.log"

    # command-line
    chef-client -L /var/log/chef/client.log

Chef configures a “logger” object.

The logger sends messages to STDOUT by default

The logger can be configured to send to a specific log in the
client.rb, or on the command-line.

# Chef Log Locations

`/etc/init.d/chef-client`

* `-L /var/log/chef/client.log`

E.g., runit captures STDOUT

* `/etc/sv/chef-client/log/main/current`

# Chef Stack Traces

Chef completely halts execution when it encounters an unhandled exception.

It doesn’t know if the error is something serious that can cause state issues that are unrecoverable.

It is up to you to fix the error and re-run Chef.

Stack traces are left behind in a file, by default `/var/chef/cache/chef-stacktrace.out`. The directory location is `file_cache_path` in /etc/chef/client.rb.

# Chef Stack Traces

Dumping the entire stack is helpful because it shows all the methods that were called and which line in the library they are found.

However, knowing where to look is the key.

# Common Errors

We’re going to look at some common errors that are easily resolved.

* non-existent cookbook
* error in a template
* non-zero return code from a command
* invalid search query syntax

# Non-existent cookbook

    INFO: HTTP Request Returned 412 Precondition Failed: {"non_existent_cookbooks":["fail2ban"],"cookbooks_with_no_versions":[],"message":"Run list contains invalid items: no such cookbook fail2ban."}
    ERROR: Running exception handlers
    FATAL: Saving node information to /var/chef/cache/failed-run-data.json
    ERROR: Exception handlers complete
    FATAL: Stacktrace dumped to /var/chef/cache/chef-stacktrace.out
    FATAL: Net::HTTPServerException: 412 "Precondition Failed"

# Non-existent cookbook

    $ cat /var/chef/cache/chef-stacktrace.out
    Generated at 2011-09-21 02:23:21 +0000
    Net::HTTPServerException: 412 "Precondition Failed"
    /opt/opscode/embedded/lib/ruby/1.9.1/net/http.rb:2303:in `error!'
    /opt/opscode/embedded/lib/ruby/gems/1.9.1/gems/chef-0.10.4/lib/chef/rest.rb:237:in `block in api_request'

# Template Error

    [Fri, 28 Jan 2011 08:50:50 -0700] ERROR: Running exception handlers
    [Fri, 28 Jan 2011 08:50:50 -0700] ERROR: Exception handlers complete
    /usr/lib/ruby/1.8/chef/mixin/template.rb:43:in `render_template': undefined local variable or method `ode' for #<Erubis::Context:0x7fd34e83cd08> (Chef::Mixin::Template::TemplateError)
            from /usr/lib/ruby/1.8/chef/provider/template.rb:100:in `render_with_context'
            from /usr/lib/ruby/1.8/chef/provider/template.rb:39:in `action_create'
            from /usr/lib/ruby/1.8/chef/resource.rb:395:in `send'
            from /usr/lib/ruby/1.8/chef/resource.rb:395:in `run_action'
            from /usr/lib/ruby/1.8/chef/runner.rb:53:in `run_action'
            from /usr/lib/ruby/1.8/chef/runner.rb:89:in `converge'
            from /usr/lib/ruby/1.8/chef/runner.rb:89:in `each'
            from /usr/lib/ruby/1.8/chef/runner.rb:89:in `converge'
    ...

# Template Error


    /usr/lib/ruby/1.8/chef/mixin/template.rb:43:in `render_template':
    undefined local variable or method `ode' for
    #<Erubis::Context:0x7fd34e83cd08> (Chef::Mixin::Template::TemplateError)

Clearly from the first message, this is a template error.

What template is this from?

# Template Error

The last part of the stack trace comes from dumping the exception from the exception handler.

We need to scroll up in the output to find what template and where.

# Template Error

    [Fri, 28 Jan 2011 08:50:50 -0700] ERROR: template[/tmp/chef-getting-started.txt] (/var/cache/chef/cookbooks/getting-started/recipes/default.rb:20:in `from_file') had an error:


    Chef::Mixin::Template::TemplateError (undefined local variable or method `ode' for #<Erubis::Context:0x7fd34e83cd08>) on line #3:

      1: Welcome to Chef!
      2:
      3: This is Chef version <%= ode[:chef_packages][:chef][:version] %>.
      4: Running on <%= node[:platform] %>.
      5: Version <%= node[:platform_version] %>.

# Template Error

      3: This is Chef version <%= ode[:chef_packages][:chef][:version] %>.

The output from the run before the exception handler dumped the stack trace.
This shows the offending line (“ode” instead of “node”).

# Non-Zero Return Code

    [Fri, 28 Jan 2011 09:14:17 -0700] ERROR: Running exception handlers
    [Fri, 28 Jan 2011 09:14:17 -0700] ERROR: Exception handlers complete
    /usr/lib/ruby/1.8/chef/mixin/command.rb:184:in `handle_command_failures': apt-get update returned 100, expected 0 (Chef::Exceptions::Exec)
            from /usr/lib/ruby/1.8/chef/mixin/command.rb:131:in `run_command'
            from /usr/lib/ruby/1.8/chef/provider/execute.rb:49:in `action_run'
            from /usr/lib/ruby/1.8/chef/resource.rb:395:in `send'
            from /usr/lib/ruby/1.8/chef/resource.rb:395:in `run_action'
            from /usr/lib/ruby/1.8/chef/runner.rb:53:in `run_action'
            from /usr/lib/ruby/1.8/chef/runner.rb:89:in `converge'
            from /usr/lib/ruby/1.8/chef/runner.rb:89:in `each'
            from /usr/lib/ruby/1.8/chef/runner.rb:89:in `converge'
            from /usr/lib/ruby/1.8/chef/resource_collection.rb:94

# Non-Zero Return


    /usr/lib/ruby/1.8/chef/mixin/command.rb:184:in
    `handle_command_failures':
    apt-get update returned 100, expected 0 (Chef::Exceptions::Exec)

The first line of the stack trace will show the command and the return code.

The expected return code by default is zero.

If Chef is run with a TTY, the command output is displayed.

# Non-Zero Return

    chef-client -l debug

Run Chef with debug logging (more on this later).

This generates a lot of output but the error message will be near the bottom.

# Non-Zero Return

    DEBUG: Re-raising exception: Chef::Exceptions::Exec - apt-get update returned 100, expected 0
    ---- Begin output of apt-get update ----
    STDOUT:
    Ign http://us.archive.ubuntu.com mylinuxsource/restricted Packages
    Err http://us.archive.ubuntu.com mylinuxsource/main Packages
      404  Not Found [IP: 91.189.88.31 80]
    Err http://us.archive.ubuntu.com mylinuxsource/restricted Packages
      404  Not Found [IP: 91.189.88.31 80]STDERR: W: Failed to fetch http://us.archive.ubuntu.com/ubuntu/dists/mylinuxsource/main/binary-amd64/Packages.gz  404  Not Found [IP: 91.189.88.31 80]

    W: Failed to fetch http://us.archive.ubuntu.com/ubuntu/dists/mylinuxsource/restricted/binary-amd64/Packages.gz  404  Not Found [IP: 91.189.88.31 80]

    E: Some index files failed to download, they have been ignored, or old ones used instead.
    ---- End output of apt-get update ----

# Invalid Search

    WARN: HTTP Request Returned 500 Internal Server Error: Net::HTTPServerException: 400 "orgapachelucenequeryParserParseException_Cannot_parse_content____true__or__not_allowed_as_first_character_in_WildcardQuery"
    ERROR: Running exception handlers
    ERROR: Exception handlers complete
    /usr/lib/ruby/1.8/net/http.rb:2101:in `error!': 500 "Internal Server Error" (Net::HTTPFatalError)
            from /usr/lib/ruby/1.8/chef/rest.rb:233:in `api_request'
            from /usr/lib/ruby/1.8/chef/rest.rb:284:in `retriable_rest_request'
            from /usr/lib/ruby/1.8/chef/rest.rb:214:in `api_request'
            from /usr/lib/ruby/1.8/chef/rest.rb:110:in `get_rest'
            from /usr/lib/ruby/1.8/chef/search/query.rb:37:in `search'
            from /usr/lib/ruby/1.8/chef/mixin/language.rb:138:in `search'
            from /var/cache/chef/cookbooks/getting-started/recipes/default.rb:25:in `from_file'
            from /usr/lib/ruby/1.8/chef/cookbook_version.rb:472:in `load_recipe'

# Invalid Search

    ...
    from /var/cache/chef/cookbooks/getting-started/recipes/default.rb:25:in `from_file'
    ...

Search query syntax follows SOLR Lucene

If there’s a syntax error, we’ll see a 500 from the server.

To find where the query is in the recipe, look for the line above.

# Invalid Search

    % sudo sed -n 25p /var/cache/chef/cookbooks/getting-started/recipes/default.rb

We can view the specific line in the cookbook on the node with sed.

Or just view it in your local $EDITOR.

# HTTP Status Codes

Chef has a RESTful HTTP API.

Since Chef speaks HTTP, some conditions may cause an HTTP status code.

Most of the time this should be a “200 OK”.

# HTTP Status Codes

Common status codes.

* 401 unauthorized
* 403 forbidden
* 404 not found
* 409 conflict
* 412 precondition failed
* 500 internal server error
* 503 service unavailable

# HTTP Status Code

    WARN: HTTP Request Returned 401 Unauthorized: Failed the authorization check
    /usr/lib/ruby/1.8/net/http.rb:2101:in `error!': 401 "Unauthorized" (Net::HTTPServerException)

# 401 Unauthorized

Common causes:

* system time is not synced
* incorrect node name
* incorrect client configuration
* invalid certificate file (validation or client)

# 403 Forbidden

Common causes:

* ACL on Opscode Hosted Chef
* Timeout retrieving files from S3 or other transient S3 error.
* Creating a data bag item when the bag doesn’t exist

# 404 Not Found

Common causes

* a source cookbook_file or template was requested but not uploaded
* another file in the cookbook was not found when requested.
* initial run of chef-client when the node doesn’t yet exist (normal).

# 409 Conflict

This usually happens when a new client is attempting to save itself as
a node that already exists.

Usually this is a warning or info (depending on context), and can be
ignored, but you should be aware of it.

# 412 Precondition Failed

This usually happens when cookbook metadata isn’t updated for a dependency, or a cookbook wasn't uploaded.

A typo on a recipe name in a run list can also cause 412 errors.

# 500 Internal Server Error

Rarely this is a user error. Most often because of invalid search syntax.

* Search uses SOLR Lucene syntax.

Otherwise, something on the server backend had an error that wasn’t handled.

* Opscode’s operations staff is notified on 500’s.
* We investigate and resolve these as quickly as possible.

# 503 Service Unavailable

Occasionally, Opscode will perform maintenance on Hosted Chef. For example we recently did a data center migration.

If you see a 503 and weren’t expecting it, check status.opscode.com.

If you need help, open a ticket at help.opscode.com.

# Debug Logging

All the various commands in Chef can enable debug log output.

The location for the log can also be specified.

This is handled with the mixlib-log library.

# Debug logging


    # With command-line arguments
    <command> -l debug
    <command> -L /tmp/debug.log

    # In the appropriate config file
    log_level :debug
    log_location "/tmp/debug.log"

These are applicable to all the commands used by Chef.

shef, chef-client, chef-solo and even ohai.

knife uses -VV

# Debug Logging

    chef-client -l debug -L /tmp/chef-client-$$.log
    knife -VV -L /tmp/knife-debug.log

A couple examples of enabling debug logging. With Knife this is actually -VV (changed in 0.10.0).

`$$` is the PID of the running process.

# Running a Chef Server

If you encounter any of the HTTP status codes, you can enable debug logging for the server.

Seek help through the community on the mailing list (lists.opscode.com) or IRC (irc.freenode.net #chef).

# Handling Errors in Recipes

Recipes are “just Ruby”.

Normally we can use a begin/rescue block to handle errors.

We can also cause Chef to bail out.

# begin..rescue

    @@@ruby
    begin
      aws = data_bag_item('aws', 'main')
    rescue
      raise
    end

# Chef::Application.fatal!

    @@@ruby
    Chef::Application.fatal!("I encountered a problem")

The Chef::Application class has a fatal! method that takes a string printed to FATAL level in the logger, and optionally a return status.

# Exception and Report Handlers

Chef has an API for reporting and exception handling.

Full coverage of the topic is in the Advanced Training class, as well as on the wiki.

We’ll take a brief look at the requirements to set up a handler.

# Handler Configuration

    @@@ruby
    # /etc/chef/client.rb...
    # 
    # a file dropped off by cookbook_file:
    require '/var/chef/handlers/simple_report_handler'
    
    # Chef::Config values:
    report_handlers << AdvancedClass::SimpleReport.new("/tmp/report.txt")
    exception_handlers << AdvancedClass::SimpleReport.new("/tmp/report.txt")

# Handler Example Code


`/var/chef/handlers/simple_report_handler.rb`

    @@@ruby
    module MyOrg
      class SimpleReport < Chef::Handler
        def report
        # ruby code that the handler performs goes here.
        end
      end
    end

The handler should inherit Chef::Handler

It should define a ‘report’ method.

Documentation is on the Chef wiki.

# Chef Handler Cookbook

Opscode released a cookbook to make managing report handlers easier.

* http://community.opscode.com/cookbooks/chef_handler

Install it in your Chef Repository with Knife:

    knife cookbook site install chef_handler

See the cookbook's README for documentation.

# Summary

You should now be able to ...

* understand Chef’s ruby stack traces
* recognize common HTTP status codes
* turn on debug logging to get more information
* handle errors gracefully in your recipes
* describe the exception handler functionality

# Additional Resources

* http://wiki.opscode.com/display/chef/Exception+and+Report+Handlers
* http://wiki.opscode.com/display/chef/Troubleshooting+and+Technical+FAQ
* http://help.opscode.com/

Several existing handlers are available on the Exception and Report
Handlers page linked above.

# Lab Exercise

## Troubleshooting
