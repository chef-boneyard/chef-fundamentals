# Troubleshooting

Section Objectives:

* Work with Chef's logger
* Find and debug Chef's stack traces
* Recognize HTTP status codes from the Chef Server
* Gracefully handle errors in recipes
* Understand Chef's exception handlers

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribute Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# Chef Log Locations

Chef configures a "logger" object.

The logger sends messages to STDOUT by default

The logger can be configured to send to a specific log in the
client.rb, or on the command-line.

    # in /etc/chef/client.rb
    log_location STDOUT
    log_location "/var/log/chef/client.log"

    # command-line
    chef-client -L /var/log/chef/client.log

# Chef Log Locations

Chef can run as a system service a variety of ways. On Linux/Unix
systems, it is common to use an init script,
e.g. `/etc/init.d/chef-client`. The default location for the log file is:

* `/var/log/chef/client.log`

Other daemonization methods vary, e.g., runit captures STDOUT, to a
file:

* `/etc/sv/chef-client/log/main/current`

# Chef Log Level

"INFO" with "verbose" logging output is the default. `verbose_logging`
shows messages about the resource being processed when Chef runs.

    # in /etc/chef/client.rb
    log_level :info
    log_level :debug
    verbose_logging true
    verbose_logging false

    # command-line
    chef-client -l debug

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

# Chef Stack Traces

Chef completely halts execution when it encounters an unhandled exception.

It doesn't know if the error is something serious that can cause state issues that are unrecoverable.

It is up to you to fix the error and re-run Chef.

Stack traces are left behind in a file, by default
`/var/chef/cache/chef-stacktrace.out`. The directory location is
`file_cache_path` in `/etc/chef/client.rb`.

# Chef Stack Traces

Dumping the entire stack is helpful because it shows all the methods
that were called and which line in the library they are found.

However, knowing where to look is the key.

# Common Errors

We're going to look at some common errors that are easily
resolved. The lines with the full path to the Ruby file can be long so
the non-unique part will be truncated.

* non-existent cookbook
* error in a template
* non-zero return code from a command
* invalid search query syntax

# Non-existent cookbook

    INFO: HTTP Request Returned 412 Precondition Failed:
    {"non_existent_cookbooks":["fail2ban"],"cookbooks_with_no_versions":
    [],"message":"Run list contains invalid items: no such
    cookbook fail2ban."}
    ERROR: Running exception handlers
    FATAL: Saving node information to /var/chef/cache/failed-run-data.json
    ERROR: Exception handlers complete
    FATAL: Stacktrace dumped to /var/chef/cache/chef-stacktrace.out
    FATAL: Net::HTTPServerException: 412 "Precondition Failed"

# Non-existent cookbook

    $ cat /var/chef/cache/chef-stacktrace.out
    Generated at 2011-09-21 02:23:21 +0000
    Net::HTTPServerException: 412 "Precondition Failed"
    1.9.1/net/http.rb:2303:in `error!'
    gems/1.9.1/gems/chef-0.10.4/lib/chef/rest.rb:237:in
    `block in api_request'

# Template Error

    chef/mixin/template.rb:43:in `render_template': undefined
    local variable or method `ode' for
    #<Erubis::Context:0x7fd34e83cd08>
    (Chef::Mixin::Template::TemplateError)
            from chef/provider/template.rb:100:in `render_with_context'
            from chef/provider/template.rb:39:in `action_create'
            from chef/resource.rb:395:in `send'
            from chef/resource.rb:395:in `run_action'
            from chef/runner.rb:53:in `run_action'
            from chef/runner.rb:89:in `converge'
            from chef/runner.rb:89:in `each'
            from chef/runner.rb:89:in `converge'
    ...

# Template Error

    chef/mixin/template.rb:43:in `render_template':
    undefined local variable or method `ode' for
    #<Erubis::Context:0x7fd34e83cd08>
    (Chef::Mixin::Template::TemplateError)

Clearly from the first message, this is a template error.

What template is this from?

# Template Error

The last part of the stack trace comes from dumping the exception from
the exception handler.

We need to scroll up in the output to find what template and where.

# Template Error

The output from the run before the exception handler dumped the stack
trace. This shows the offending line ("ode" instead of "node").

    ERROR: template[/tmp/chef-getting-started.txt]
    (/var/cache/chef/cookbooks/getting-started/recipes/default.rb:20:in
    `from_file') had an error:

    Chef::Mixin::Template::TemplateError (undefined local variable
    or method `ode' for #<Erubis::Context:0x7fd34e83cd08>) on line #3:

      1: Welcome to Chef!
      2:
      3: This is Chef version <%= ode[:chef_packages][:chef][:version] %>.

# Non-Zero Return Code

    ERROR: Exception handlers complete
    chef/mixin/command.rb:184:in `handle_command_failures':
    apt-get update returned 100, expected 0 (Chef::Exceptions::Exec)
            from chef/mixin/command.rb:131:in `run_command'
            from chef/provider/execute.rb:49:in `action_run'
            from chef/resource.rb:395:in `send'
            from chef/resource.rb:395:in `run_action'
            from chef/runner.rb:53:in `run_action'
            from chef/runner.rb:89:in `converge'
            from chef/runner.rb:89:in `each'
            from chef/runner.rb:89:in `converge'
            from chef/resource_collection.rb:94

# Non-Zero Return

    /usr/lib/ruby/1.8/chef/mixin/command.rb:184:in
    `handle_command_failures':
    apt-get update returned 100, expected 0 (Chef::Exceptions::Exec)

The first line of the stack trace will show the command and the return
code.

The expected return code by default is zero.

If Chef is run with a TTY, the command output is displayed.

# Non-Zero Return

    chef-client -l debug

Run Chef with debug logging (more on this later).

This generates a lot of output but the error message will be near the
bottom.

# Non-Zero Return

Output is truncated:

    DEBUG: Re-raising exception: Chef::Exceptions::Exec -
    apt-get update returned 100, expected 0
    ---- Begin output of apt-get update ----
    STDOUT:
    Ign http://us.archive.ubuntu.com mylinuxsource/restricted Packages
    Err http://us.archive.ubuntu.com mylinuxsource/main Packages
      404  Not Found [IP: 91.189.88.31 80]
    Err http://us.archive.ubuntu.com mylinuxsource/restricted Packages
      404  Not Found [IP: 91.189.88.31 80]STDERR: W: Failed to fetch h
    W: Failed to fetch http://us.archive.ubuntu.com/ubuntu/dists/
    E: Some index files failed to download, they have been ignored, or old ones used instead.
    ---- End output of apt-get update ----

# Invalid Search

    WARN: HTTP Request Returned 500 Internal Server Error:
    Net::HTTPServerException: 400 "orgapachelucenequeryParserParse
    Exception_Cannot_parse_content____true__or__not_allowed_as_first
    _character_in_WildcardQuery"
    ERROR: Running exception handlers
    ERROR: Exception handlers complete
    net/http.rb:2101:in `error!': 500 "Internal Server Error"
    (Net::HTTPFatalError)
            from chef/rest.rb:233:in `api_request'
            from chef/rest.rb:284:in `retriable_rest_request'
            from chef/rest.rb:214:in `api_request'
            from chef/rest.rb:110:in `get_rest'
            from chef/search/query.rb:37:in `search'
            from chef/mixin/language.rb:138:in `search'
            from {cache}/chef/cookbooks/getting-started/recipes/default.rb:25:in `from_file'
            from chef/cookbook_version.rb:472:in `load_recipe'

# Invalid Search

    ...
    from {cache}/chef/cookbooks/getting-started/recipes/default.rb:25:in `from_file'
    ...

Search query syntax follows SOLR Lucene

If there's a syntax error, we'll see a 500 from the server.

To find where the query is in the recipe, look for the line above.

# Invalid Search

    % sudo sed -n 25p cookbooks/getting-started/recipes/default.rb

We can view the specific line in the cookbook on the node with sed.

Or just view it in your local repository with your favorite editor.

# Fixing Recipe Errors

Fixing recipes errors is relatively straightforward after they have
been detected similar to the approaches above.

* Modify the recipe / cookbook.
* Upload cookbook to Chef Server.
* Run Chef on node(s).

# HTTP Status Codes

The Chef Server has a RESTful HTTP API.

Since Chef speaks HTTP, some conditions may cause an HTTP status code.

Most of the time this should be a "200 OK". Occassionally some error
condition will cause a different kind of HTTP status code.

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

    WARN: HTTP Request Returned 401 Unauthorized: Failed the
    authorization check
    net/http.rb:2101:in `error!': 401 "Unauthorized"
    (Net::HTTPServerException)

# 401 Unauthorized

Common causes:

* system time is not synced
* incorrect node name
* incorrect client configuration
* invalid certificate file (validation or client)

The Chef Server message will indicate what the likely cause is.

# 403 Forbidden

Common causes:

* ACL on Opscode Hosted Chef
* Timeout retrieving files from S3 or other transient S3 error.
* Creating a data bag item when the bag doesn't exist
* Attempting to create an API Client that exists

# 404 Not Found

Common causes

* a source cookbook_file or template was requested but not uploaded
* another file in the cookbook was not found when requested.
* initial run of chef-client when the node doesn't yet exist (normal).

# 409 Conflict

This usually happens when a new client is attempting to save itself as
a node that already exists.

Usually this is a warning or info (depending on context), and can be
ignored, but you should be aware of it.

# 412 Precondition Failed

This usually happens when cookbook metadata isn't updated for a
dependency, or a cookbook wasn't uploaded.

A typo on a recipe name in a run list can also cause a 412 error.

# 500 Internal Server Error

This is rarely a user error, however as discussed SOLR search syntax
errors can cause a 500 error.

Otherwise, something on the server backend had an error that wasn't
handled.

* Opscode's operations staff is notified on 500's.
* We investigate and resolve these as quickly as possible.

# 503 Service Unavailable

Occasionally, Opscode will perform maintenance on Hosted Chef. For
example we recently did a data center migration.

If you see a 503 and weren't expecting it, check
[status.opscode.com](http://status.opscode.com).

If you need help, open a ticket at
[help.opscode.com](http://help.opscode.com) or email
[support@opscode.com](mailto:support@opscode.com).

# Running a Chef Server

If you encounter any of the HTTP status codes, you can enable debug
logging for the server.

Seek help through the community on the mailing list
([lists.opscode.com](http://lists.opscode.com)) or IRC ([irc.freenode.net #chef](irc://irc.freenode.net/chef)).

# Handling Errors in Recipes

Recipes are "just Ruby".

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

# Ignoring Failure

If the failure for a particular resource is acceptable, you can use
the `ignore_failure` meta-parameter to tell Chef not to exit if it
encounters an error configuring that resource.

    @@@ruby
    execute "apt-get update" do
      ignore_failure true
    end

This is available to every resource.

# Exception and Report Handlers

Chef has an API for reporting and exception handling.

Full coverage of the topic is beyond the scope of a "Fundamentals"
course. The Chef wiki has an
[entire page documenting the API](http://wiki.opscode.com/display/chef/Exception+and+Report+Handlers).

We'll take a brief look at the requirements to set up a handler.

# Handler Configuration

Chef has two configuration options for setting up a new
report/exception handler.

* `report_handlers`
* `exception_handlers`

These are both Arrays, to which new instantiation of the handler
classes are added.

# Handler Configuration

    @@@ruby
    # /etc/chef/client.rb...
    #
    # a file dropped off by cookbook_file:
    require '/var/chef/handlers/simple_report_handler'

    report_handlers << SimpleReport::UpdatedResources.new
    exception_handlers << SimpleReport::UpdatedResources.new

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

The handler should inherit `Chef::Handler`

It should define a 'report' method.

Documentation is on the [Chef wiki](http://wiki.opscode.com/display/chef/Exception+and+Report+Handlers).

# Chef Handler Cookbook

Opscode released a cookbook to make managing report handlers easier.

* http://community.opscode.com/cookbooks/chef_handler

Download it to your Chef Repository like any other cookbook:

    knife cookbook site download chef_handler
    tar -zxvf chef_handler-1.0.0.tar.gz -C cookbooks

See the cookbook's README for documentation.

# Summary

* Work with Chef's logger
* Debug Chef's stack traces
* Recognize HTTP status codes from the Chef Server
* Gracefully handle errors in recipes
* Understand Chef's exception handlers

# Questions

* What are two ways to run chef-client with debug log output?
* What does `verbose_logging` show/hide when changed?
* Where does Chef write the stack trace output to by default? What
  configuration setting can change the directory location?
* What is the workflow for correcting cookbook errors?
* What often causes a 401 error?
* How can a 412 error be resolved?
* Describe the report/exception handling feature of Chef.

# Additional Resources

* http://wiki.opscode.com/display/chef/Common+Errors
* http://wiki.opscode.com/display/chef/Exception+and+Report+Handlers
* http://wiki.opscode.com/display/chef/Troubleshooting+and+Technical+FAQ
* http://help.opscode.com/ or support@opscode.com
* http://lists.opscode.com

Several existing handlers are available on the Exception and Report
Handlers page linked above.
