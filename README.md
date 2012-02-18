# Overview

Welcome to Chef Fundamentals. This is the source training material
repository for the class. The materials themselves are written in
plain text "[Markdown](http://daringfireball.net/projects/markdown/)"
format, and presented using the
"[ShowOff](https://github.com/schacon/showoff)" Ruby-based presentation
application.

# Setup

Requirements:

* Ruby 1.8.7+
* RubyGems 1.3.7+
* libxml2 and libxslt development headers (e.g., libxml2-dev and
  libxslt-dev on Debian/Ubuntu).

```
gem install bundler
bundle install
cd slides
bundle exec showoff serve
```

Depending on how your local system's Ruby was installed, you may need
to use `sudo` to run `gem install` and `bundle install`. You may also
need to use `bundle exec showoff serve` to run the presentation.

Two URLs are available:

* http://localhost:9090 - "student" view of the slides
* http://localhost:9090 - "presenter view of the slides

When presenting the materials as an instructor, use the "presenter"
view. This will also pop up a second browser window that will advance
with the presenter window. To move forward and back, use the arrow
keys. Down/Right go forward, Up/Left go backward. Spacebar will also
advance slides forward.

# Installed Gems

The source materials in the `slides` directory are set up as a showoff
project. As such, the `showoff` gem is required. In order to generate
PDFs with showoff, the `pdfkit` gem is installed.

Also, the instructor lab setup will use Chef and the Knife EC2 plugin,
so those gems are included as well.

Do not commit `Gemfile.lock` to the repository.

# Slides Directory

Most of the action happens in the slides directory.

## Rakefile

The `slides/Rakefile` has a task to set up the directory and an
initial `01_slide.md` file for the specified section.

    rake mksection SECTION=just-enough-ruby-for-chef
    ** Creating section just-enough-ruby-for-chef **
    - populating 01_slide.md

This does not add the section to `showoff.json`. When contributing a new
section, do not add it to the `showoff.json` file, as that should be
reviewed before modifying the base course for everyone.

## Slide Style Guide

This is spartan and will be embellished.

* Create sections as directories.
* Use PNGs for images.
* Directory names should be lower case words as a title, "getting-started"
* Each directory should have a 01_slide.md, multiple sections may be
  broken up later.

# Guides Directory

See the `instructor-setup.md` guide in the guides directory for
information on how to set up the lab environments for students to use
in the hands on portion of the course.

# Resources

http://daringfireball.net/projects/markdown/syntax
https://github.com/schacon/showoff

# Contributing

See CONTRIBUTING for information on how you can contribute changes to
these materials.

# License and Authors

See LICENSE for licensing of these materials and how you may use
them.

## Authors:

Joshua Timberman <joshua@opscode.com>
Aaron Peterson <aaron@opscode.com>
