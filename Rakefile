# [x] create the folders
# [x] download the html documentation in this folder
# [x] add plist
# [ ] create sqlite index
# [ ] setup feed with archived docset
# [ ] link to docset feed
# [ ] add icon
# [x] enable javascript

require "nokogiri"
require "rake/clean"
task :default => :generate

desc "Generate the docset for jasmine"
task :generate => [:download_docs, :setup_plist, :create_index]

DOCSET = "jasmine.docset"
CONTENTS = "#{DOCSET}/Contents"
RESOURCES = "#{CONTENTS}/Resources"
HTML = "#{RESOURCES}/Documents"
CLOBBER.include DOCSET

task :download_docs do
  mkdir_p HTML
  `wget --progress=dot --convert-links -p http://pivotal.github.io/jasmine/`
  cp_r Dir["pivotal.github.io/jasmine/*"], HTML
end
CLEAN.include "pivotal.github.io"

task :setup_plist do
  cp "templates/info.plist", CONTENTS
end

task :create_index do
  file = "#{RESOURCES}/docSet.dsidx"
  html_index = "#{HTML}/index.html"
  touch file
  populate_index file, html_index
end

def populate_index db, to_index
  puts :foo
end
