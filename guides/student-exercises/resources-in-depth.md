Resources In Depth
======================

## Objectives

* Understand components of resources
* Write resources into shef and observe the outcome

## Acceptance Criteria

* Answer the questions

## Use Shef to write resources

During the lecture, the instructor will provide examples that are used
to interactively observe what happens in writing Chef resources. Use
that information and explore on your own to answer the questions below.

You can interact with a resource as a Ruby object by loading it with
the `resource()` method. The syntax is:

    chef:recipe > my_resource = resource("type[name]")

Parameters are methods that can be called with a value to set that
parameter. For example,

    chef:recipe > my_resource.some_parameter "value"

When called without a value, `shef` will return the current value of
the parameter.

    chef:recipe > my_resource.some_parameter
     => "some-value"

An action can be triggered immediately with the `run_action` method.

    chef:recipe > my_resource.run_action(:action)

A full example:

    > shef
    chef > recipe
    chef:recipe > file "sample"
    chef:recipe > my_resource = resources("file[sample]")
    chef:recipe > my_resource.content "Example resource content"
    chef:recipe > my_resource.run_action(:create)
    INFO: file[sample] created file sample
    chef:recipe > IO.read("sample")
     => "Example resource content"

(`IO.read` is a Ruby method to read in the content of a file)

## Quesetions

When creating a `file` resource, what are the allowed actions?

What is the action for a `file` resource if it is not specified? A
`package` resource?

If Chef is run as a non-privileged user, what happens if a different
`owner` is specified for any of the file resources?

Resources entered into `shef`'s recipe context are not configured
immediately. Use the recipe help option to determine the command to
run Chef.

Use the recipe help information to determine the command to list all
the resources in the current session. What is it?

Create a service resource. What actions are allowed? What is the
default action?

What does the service resource support by default? (supports
parameter)

In `shef`, create a `service[foo]` resource with no other
parameters. If you run Chef, what is the current state of the service?
How does Chef determine this?
