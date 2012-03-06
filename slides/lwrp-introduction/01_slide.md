# Lwrp Introduction

Section Objectives:

* Components of LWRPs
* Resource DSL
* Provider DSL
* Build an LWRP from scratch
* Opscode Example LWRPs

.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribution Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.

# When to use LWRPs

Lightweight Resources and Providers are used when you want to abstract
some repeated pattern of behavior on the system with a declarative
interface that doesn't already exist in Chef.

Examples of LWRPs use:

* Adding configuration for a yum repository
* Managing a service with a new supervision system, e.g. bluepill
* Subsystem user management, e.g. samba
* Tools that use "conf.d" configuration inclusion

# Components of an LWRP

LWRPs have two components, the resource and the provider. They live in
the `resources` and `providers` directory of cookbooks, respectively.

Chef uses the cookbook name and the resource and provider file names
to create the resource that is used in recipes. If the filename is
"default" then only the cookbook name needs to be used in the recipe.
For example, if we have a cookbook named "mouse":

    | File                    | Resource Name | Generated Provider    |
    |-------------------------|---------------|-----------------------|
    | mouse/resources/default | mouse         | Chef::Resource::Mouse |
    | mouse/providers/default | mouse         | Chef::Provider::Mouse |

# Using the LWRP in a Recipe

When we use the resource in a recipe, we write something like this:

    @@@ruby
    mouse "Squeak" do
      action :say
    end

# Resource DSL

The resource DSL is very simple. There are two methods used,
`actions` and `attribute`.

The `actions` method specifies the allowed actions that can be used
in a recipe. It is a comma separated list of Ruby symbols. Each action
corresponds to an `action` method in the provider.

The `attribute` method specifies a new parameter attribute for the
resource. The attribute's name must be the first parameter, and it
should be a Ruby symbol. Optionally, a hash of validation parameters
can be specified.

.notes We will discuss validation parameters in more detail when we
get to building an example.

# Provider DSL

The provider DSL only defines a single method, `action`. You specify
the action name with a Ruby symbol, which is one of the allowed
actions defined in the resource `actions` list. The method takes a
block, which is the code that does whatever is required to configure
the resource for that particular action.

Each provider's action methods can re-use Chef Resources, meaning you
can use existing Chef resources for free inside the action methods.

.notes We're going to look at building an example next.

# Build an LWRP from Scratch

We're going to build a contrived example to illustrate the various
components of an LWRP. The resource will be named `mouse`, and we will
make use of it in `recipe[mouse]`. The recipe should be applied to a
node's run list. At each step, upload the cookbook and run Chef to
observe the output.

    > mkdir -p cookbooks/mouse/recipes
    > touch cookbooks/mouse/recipes/default.rb

Write the resource in a recipe. Since this is in the cookbook named
`mouse`, the resource is named `mouse`.

    @@@ruby
    mouse "Squeak"

# Run Chef

Upload the cookbook, apply the recipe to a node and run Chef:

    > knife cookbook upload mouse
    > knife node run list add NODENAME 'recipe[mouse]'
    NODENAME$ sudo chef-client

When Chef runs we will see this:

    FATAL: NoMethodError: undefined method `mouse' for #<Chef::Recipe:0x007fa1f4842bf8>

# Create the Resource

Create the resource so Chef knows about it. Resources go in the
`resources` directory in the cookbook. The file for the resource is
`default.rb`.

    > mkdir -p cookbooks/mouse/resources
    > touch cookbooks/mouse/resources/default.rb

Upload the cookbook again and run Chef; we will see this error:

    INFO: Processing mouse[Squeak] action nothing (mouse::default line 1)
    ERROR: mouse[Squeak] (mouse::default line 1) has had an error
    ERROR: mouse[Squeak] (cookbooks/mouse/recipes/default.rb:1:in
    `from_file') had an error: mouse[Squeak] (mouse::default line 1)
    had an error: ArgumentError: Cannot find a provider for
    mouse[Squeak] on mac_os_x version 10.7.3

# Provider Lookup

    INFO: Processing mouse[Squeak] action nothing (mouse::default line 1)
    ERROR: mouse[Squeak] (mouse::default line 1) has had an error
    ERROR: mouse[Squeak] (cookbooks/mouse/recipes/default.rb:1:in
    `from_file') had an error: mouse[Squeak] (mouse::default line 1)
    had an error: ArgumentError: Cannot find a provider for
    mouse[Squeak] on mac_os_x version 10.7.3

Chef looks for a provider for the resource. With LWRPs, the provider
is in the same cookbook as the resource, in the resources directory.

The platform and version are printed because Chef tries to map the
provider to the resource based on the node's platform and platform
version.

# Create the Provider

Create the provider so Chef can find it for the resource. Providers go
in the `providers` directory. The file for the provider is
`default.rb` as we are using the default resource.

    > mkdir -p cookbooks/mouse/providers
    > touch cookbooks/mouse/providers/default.rb

When Chef runs we will see this:

    INFO: Processing mouse[Squeak] action nothing (mouse::default
    line 1)

The default action for a newly created resource is `nothing`.

# Set an Action in the Recipe

Pass the action parameter to the resource in the recipe. The mouse
should say something with the `say` action.

    @@@ruby
    mouse "Squeak" do
     action :say
    end

When Chef runs we will see this:

    FATAL: Chef::Exceptions::ValidationFailed: Option action must be
    equal to one of: nothing! You passed :say.

The resource does not list `say` is a valid action; the default action
for any new resource in Chef is `nothing`.

# Create Allowed Actions List

Use the `actions` keyword in the resource to define an allowed action.

    @@@ruby
    actions :say

The action should be a Ruby symbol (starting with `:`). Now with the
action defined as allowed, when Chef runs we see this:

    INFO: Processing mouse[Squeak] action say (mouse::default line 1)
    ERROR: mouse[Squeak] (mouse::default line 1) has had an error
    ERROR: mouse[Squeak] (cookbooks/mouse/recipes/default.rb:1:in
    `from_file') had an error:
    mouse[Squeak] (mouse::default line 1) had an error: NoMethodError:
    undefined method `action_say'
    for #<Chef::Provider::Mouse:0x007fb661060198>

# Resource Calls Action Method

    INFO: Processing mouse[Squeak] action say (mouse::default line 1)
    ERROR: mouse[Squeak] (mouse::default line 1) has had an error
    ERROR: mouse[Squeak] (cookbooks/mouse/recipes/default.rb:1:in
    `from_file') had an error:
    mouse[Squeak] (mouse::default line 1) had an error: NoMethodError:
    undefined method `action_say'
    for #<Chef::Provider::Mouse:0x007fb661060198>

Chef attempts to call the `say` action method. The method has not been
written in the provider yet, resulting in this error.

# Create the Action Method

Write a simple method that prints a string with `puts`. The name of
the method, `:say`, should be a Ruby symbol.

    @@@ruby
    action :say do
      puts "My name is #{new_resource.name}"
    end

When Chef runs we will see this:

    INFO: Processing mouse[Squeak] action say (mouse::default line 1)
    My name is Squeak

Chef will call the provider's `say` action as the `action_say` method,
as shown earlier with the NoMethodError.

    mouse[Squeak] (mouse::default line 1) had an error: NoMethodError:
    undefined method `action_say'
    for #<Chef::Provider::Mouse:0x007fb661060198>

# Inspecting the Action Method

The action method takes a block. In this example, we used the Ruby
puts method to print the string to standard output.

Our example uses Ruby's string interpolation `#{}` to print the value
of `new_resource.name`.

In a Chef provider, an object called `new_resource` is created, which
represents the resource as it was written in the recipe.

Again, our recipe has this resource:

    @@@ruby
    mouse "Squeak" do
      action :say
    end

# Inspecting the Action Method

    @@@ruby
    mouse "Squeak" do
      action :say
    end

Chef automatically creates an accessor attribute for all new resources
called `name`, which is "Squeak" in our example. When Chef runs, this
provider then prints out this string as that is what we put in the
action.

# Recipe DSL in Providers

Chef's LWRPs allow you to reuse the Chef recipe DSL, which means you
can write resources inside the provider's actions.

Convert `puts` to a Chef `log` resource.

    @@@ruby
    action :say do
      log "My name is #{new_resource.name}"
    end

When Chef runs we will see this:

    INFO: Processing mouse[Squeak] action say (mouse::default line 1)
    INFO: Processing log[My name is Squeak] action write
    (cookbooks/mouse/providers/default.rb line 2)
    INFO: My name is Squeak

# Recipe DSL in Providers

    INFO: Processing mouse[Squeak] action say (mouse::default line 1)
    INFO: Processing log[My name is Squeak] action write
    (cookbooks/mouse/providers/default.rb line 2)
    INFO: My name is Squeak

Chef creates two resources here. First is the enclosing resource, the
`mouse` LWRP. Second is the `log` resource used in the provider.

Note the `INFO:` before the printed string. The `log` resource uses
Chef's logger object, which prints messages at
`Chef::Config[:log_level]` "info" by default.

# Create Resource Attribute Parameters

The `mouse` resource isn't particularly descriptive. It really only
shows the name of itself in the `say` action. Add a couple new
attribute parameters to the resource to use in the provider.

    @@@ruby
    actions :say

    attribute :given_name, :name_attribute => true
    attribute :noise, :default => "quiet"
    attribute :tail, :default => true, :kind_of => [TrueClass, FalseClass]

These attributes use the LWRP Resource DSL's validation parameters.

# Resource Validation Parameters

The `attribute` method takes two parameters, the name of the attribute
and a Hash of additional parameters, also called "validation
parameters".

These attributes are parameters passed into the resource in a recipe,
they are not node attributes.

# Validation Parameters

Validation parameters are used to ensure that the value of the
parameter meets the defined requirements. For example, a parameter
that is used for package name should be a string.

    | Validation      | Meaning                                |
    |-----------------+----------------------------------------|
    | :callbacks      | Hash of Procs, should return true      |
    | :default        | Default value for the parameter        |
    | :equal_to       | Match the value with ==                |
    | :kind_of        | Ensure the value is a particular class |
    | :name_attribute | Set to the resource name               |
    | :regex          | Match the value against the regex      |
    | :required       | The parameter must be specified        |
    | :respond_to     | Ensure the value has a given method    |

# Name and Default Attributes

    @@@ruby
    attribute :given_name, :name_attribute => true
    attribute :noise, :default => "quiet"
    attribute :tail, :default => true, :kind_of => [TrueClass, FalseClass]

Commonly used in the validation parameters are `:name_attribute`
and `:default`.

For the mouse resource, the `:given_name` attribute is
the `:name_attribute`. The `:default` value of `:noise` is the string
"quiet", since a sane default for mice is that they are quiet.

The `:tail` attribute has a default value of true. An Array of valid
Ruby classes is specified for `:kind_of`. It is reasonable to leave
out NilClass because it is `true` by default.

# Extend the Provider's Action

Were Chef to run again, nothing changes in the output because the
provider has not changed. Extend the provider's action with some
additional behavior.

    @@@ruby
    action :say do
      log "My name is #{new_resource.given_name}"
      unless new_resource.noise =~ /^quiet$/
        log "I say #{new_resource.noise}"
      end
      log "I #{new_resource.tail ? 'do' : 'do not'} have a tail"
    end

The unless statement checks whether the mouse has a noise other than
"quiet" and if so, prints a log message with the noise.

The last line includes a Ruby ternary operator that checks whether the
tail attribute was true or false and prints whether the mouse has a
tail or not.

# Extended Provider Output

When Chef runs we will see:

    INFO: Processing mouse[Squeak] action say (mouse::default line 1)
    INFO: Processing log[My name is Squeak] action write
    (cookbooks/mouse/providers/default.rb line 2)
    INFO: My name is Squeak
    INFO: Processing log[I do have a tail] action write
    (cookbooks/mouse/providers/default.rb line 6)
    INFO: I do have a tail

Since the default value for the `noise` attribute in the mouse
resource is `quiet`, it didn't say anything, and since the default
value for `tail` is true, the mouse does in fact have his tail.

# Modify Resource in Recipe

Now let's modify the resource in the recipe.

    @@@ruby
    mouse "Squeak" do
      noise "SQUEAK!"
      tail false
      action :say
    end

# Running the Modified Resource:

When Chef runs we will see:

    INFO: Processing mouse[Squeak] action say (mouse::default line 1)
    INFO: Processing log[My name is Squeak] action write
    (cookbooks/mouse/providers/default.rb line 2)
    INFO: My name is Squeak
    INFO: Processing log[I say SQUEAK!] action write
    (cookbooks/mouse/providers/default.rb line 4)
    INFO: I say SQUEAK!
    INFO: Processing log[I do not have a tail] action write
    (cookbooks/mouse/providers/default.rb line 6)
    INFO: I do not have a tail

Clearly this mouse has squeaked loudly because his tail is missing!

# Default Action

The action had to be specified throughout the example recipe because
the Resource DSL does not [yet] have the ability to specify the
default action. An initialize method can be created in the
`resources/default.rb` to create a default action.

    @@@ruby
    def initialize(*args)
      super
      @action = :say
    end

The action is an instance variable, and must begin with an `@`. The
value must be a Ruby symbol. Now we can remove the `action` in our
resource in the recipe:

    @@@ruby
    mouse "Squeak" do
      noise "SQUEAK!"
      tail false
    end

.notes This is more idiomatic with other resources in Chef, in that they
define default actions so if one is not specified, the resource does
pretty much what you expect.

# Contrived Example

This is a completely contrived example, and it does not do much, nor
would it be sane to continue trying to modify our arbitrary mouse, so
we will examine some LWRPs in Opscode's cookbooks for further
examples.

Things we'll see:

* `load_current_resource`
* resource parameter attributes that track state
* resources that are not `default`
* more validation parameters
* more complex Ruby usage
* `new_resource.updated_by_last_action(true)`

# Opscode LWRPs

* `yum_repository`
* `bluepill_service`
* `samba_user`
* `gunicorn_config`

.notes Crack open the text editor in the resource and provider for
each of these LWRPs.

# Summary

* Components of LWRPs
* Resource DSL
* Provider DSL
* Build an LWRP from scratch
* Opscode Example LWRPs

# Questions

* What are the components of an LWRP?
* What are the resource DSL's two methods?
* What method is used in providers to create actions?
* What are validation parameters? Give two examples.
* What does ":name_attribute" mean? How is it used?

# Additional Resources

* http://wiki.opscode.com/display/chef/Lightweight+Resources+and+Providers+%28LWRP%29
* http://wiki.opscode.com/display/chef/Opscode+LWRP+Resources

# Lab Exercise

Lwrp Introduction
