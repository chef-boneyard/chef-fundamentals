# -*- coding: utf-8 -*-
desc "Create the directory and initial slides (with SECTION=name)"
task :mksection do
  section = ENV['SECTION']
  slides_dir = File.join(File.dirname(__FILE__), "slides", section)
  guides_dir = File.join(File.dirname(__FILE__), "guides", "student-exercises")
  puts "** Creating section #{section} **"
  FileUtils.mkdir_p(slides_dir)
  FileUtils.mkdir_p(guides_dir)
  unless File.exists?(File.join(slides_dir, "01_slide.md"))
    puts "- populating slide template #{slides_dir}/01_slide.md"
    File.open(File.join(slides_dir, "01_slide.md"), "a+") do |f|
      f.puts "# #{section_header(section)}\n\n"
      f.puts "Section Objectives:\n"
      f.puts copyright_header
      f.puts "# Summary\n\n\n"
      f.puts "# Questions\n\n*\n\n"
      f.puts "# Additional Resources\n\n*\n\n"
      f.puts "# Lab Exercise\n\n"
      f.puts section_header(section)
    end
    puts "- populating exercises #{File.join(guides_dir, section)}.md"
    File.open(File.join(guides_dir, "#{section}.md"), "a+") do |g|
      g.puts exercise_guide(section)
    end
  end
end

desc "Generate HTML out of Markdown"
task :md_to_html do
  html_dir = File.join("slides", "html")
  sections = showoff_sections
  FileUtils.mkdir_p(html_dir)
  sections.each_with_index do |s,i|
    fn = File.join(html_dir, "section-#{i}-#{s}.html")
    %x[grep -v "@@@" slides/#{s}/01_slide.md | redcarpet > #{fn}]
    %x[cp #{s}/*.png html 2>/dev/null]
    File.open(File.join("slides", "html", "index.html"), "a+") do |h|
      h.puts html_list_item s, fn
    end
  end
end

desc "Count the number of slides per section"
task :slide_counter do
  sections = showoff_sections
  puts "** Number of slides in each section:"
  sections.each do |section|
    puts "- #{section}: #{line_count(File.join("slides", section, "01_slide.md"))}"
  end
end

def line_count(slides)
  count = 0
  begin
    f = File.open(slides)
  rescue
    return "section does not exist, create with `mksection`"
  end
  f.each do |line|
    count += 1 if line =~ /^# /
  end
  count
end

def html_list_item(section, filename)
  %Q{<li><a href="../#{filename}">#{section}</a></li>}
end

def showoff_sections
  require 'json'
  JSON.parse(IO.read(File.join("slides", "showoff.json")))['sections']
end

def section_header(section = "welcome")
  lcase = %w{an of the a}
  parts = section.split(/[-_]/)
  header = Array(parts).flatten.map {|p| lcase.include?(p) ? p : p.capitalize }
  header.join(' ')
end

def copyright_header
  "\n.notes These course materials are Copyright © 2010-2012 Opscode, Inc. All rights reserved.
This work is licensed under a Creative Commons Attribution Share Alike 3.0 United States License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/us; or send a letter to Creative Commons, 171 2nd Street, Suite 300, San Francisco, California, 94105, USA.\n\n"
end

def exercise_guide(section = "welcome")
  "#{section_header(section)}\n======================\n\n## Objectives\n\n## Acceptance Criteria\n\n## Quesetions\n"
end
