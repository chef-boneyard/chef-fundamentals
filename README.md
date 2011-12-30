# Setup

```
gem install bundler
bundle install
cd slides
showoff serve
```

Browse to localhost:9090

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

The `slides/Rakefile` has a task to set up the directories and an
initial `01_slide.md` file for each of the sections in showoff.json.

## Slide Style Guide

This is spartan and will be embellished.

* Create sections as directories.
* Use PNGs for images.
* Directory names should be lower case words as a title, "getting-started"
* Each directory should have a 01_slide.md

# Resources

http://daringfireball.net/projects/markdown/syntax
https://github.com/schacon/showoff

# Contributing

See LICENSE.txt for licensing of these materials and how you may use
them.

See CONTRIBUTING for information on how you can contribute changes to
these materials.
