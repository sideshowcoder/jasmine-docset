require "nokogiri"
require "pry"
require "rake/clean"

task :default => :generate

desc "Generate the docset for jasmine"
task :generate => [:download_docs, :setup_plist, :recreate_index]

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

task :recreate_index do
  db = "#{RESOURCES}/docSet.dsidx"
  html_index = "#{HTML}/index.html"
  rm db
  touch db

  SearchIndex.create_and_populate db, html_index
end

class SearchIndex
  IGNORED_SECTIONS = [ "Downloads", "Support", "Thanks" ]

  attr_reader :db
  def initialize db_file
    @db = File.open(db_file, "w")
  end

  def self.create_and_populate(db_file, html_file)
    search_index = new(db_file)
    search_index.create
    search_index.populate_with_content html_file
  end

  def create
    db.write "CREATE TABLE searchIndex(id INTEGER PRIMARY KEY, name TEXT, type TEXT, path TEXT);\n"
    db.write "CREATE UNIQUE INDEX anchor ON searchIndex (name, type, path);\n"
  end

  def populate_with_content html_file
    File.open(html_file, "r") { |f|
      doc = Nokogiri::HTML(f.read)
      populate_with doc
    }
  end

  private
  def populate_with doc
    doc.css(".docs h2").each { |e|
      next if IGNORED_SECTIONS.include? e.content
      name = e.content.split(":")[0]
      add_to_index name, "Section", "##{e.parent.parent["id"]}"
    }
  end

  def add_to_index name, type, path
    db.write "INSERT OR IGNORE INTO searchIndex(name, type, path) VALUES ('#{name}', '#{type}', '#{path}');\n"
  end

end

