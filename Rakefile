require "nokogiri"
require "erb"
require "ostruct"
require "sqlite3"
require "rake/clean"
require "rake/packagetask"

task :default => :create_package

desc "Create a release"
task :release => [:create_package, :package, :create_release]

DOCSET = "jasmine.docset"
CONTENTS = "#{DOCSET}/Contents"
RESOURCES = "#{CONTENTS}/Resources"
HTML = "#{RESOURCES}/Documents"
PUBLIC = "public"
VERSION = "1.3.1"
CLOBBER.include DOCSET

# [ ] upload to github pages
Rake::PackageTask.new("jasmine-docset", "1.3.1") do |p|
  p.need_tar = true
  p.package_dir = PUBLIC
  p.package_files.include("#{DOCSET}/*")
  p.package_files.include("#{DOCSET}/**/*")
end

task :create_release do
  url = "http://sideshowcoder.github.io/jasmine-docset/public/jasmine-docset-#{VERSION}.tgz"
  feed_variables = OpenStruct.new(version: VERSION, url: url)
  feed_template = File.read("./templates/feed.xml.erb")
  feed = ERB.new(feed_template).result(feed_variables.instance_eval { binding })
  File.open("#{PUBLIC}/Jasmine.xml", "w") { |f| f.write feed }
end

task :publish do
  Dir.mktmpdir { |dir|
    cp "public/jasmine-docset-#{VERSION}.tgz", dir
    cp "public/Jasmine.xml", dir
    `git checkout gh-pages`
    cp "#{dir}/jasmine-docset-#{VERSION}.tgz", "public"
    cp "#{dir}/Jasmine.xml", "public"
    `git add -A`
    current_commit = `git rev-parse --short HEAD`
    `git commit -m 'updated to #{current_commit}'`
    `git push origin gh-pages`
    `git checkout master`
    `git stash pop`
  }
end

desc "Generate the docset package for jasmine"
task :create_package do
  mkdir_p HTML
  `wget --progress=dot --convert-links -p http://pivotal.github.io/jasmine/`
  cp_r Dir["pivotal.github.io/jasmine/*"], HTML
  cp "templates/Info.plist", CONTENTS
  cp "templates/icon.png", DOCSET
  db = "#{RESOURCES}/docSet.dsidx"
  html_index = "#{HTML}/index.html"
  touch db

  SearchIndex.create_and_populate db, html_index
end
CLEAN.include "pivotal.github.io"

class SearchIndex
  IGNORED_SECTIONS = [ "Downloads", "Support", "Thanks" ]

  def initialize db_file
    @db_file = db_file
  end

  def with_db
    begin
      db = SQLite3::Database.open @db_file
      yield db
    rescue SQLite3::Exception => e
      puts "[ERROR] #{e}"
    ensure
      db.close if db
    end
  end

  def self.create_and_populate(db_file, html_file)
    search_index = new(db_file)
    search_index.create
    search_index.populate_with_content html_file
  end

  def create
    with_db { |db|
      db.execute "CREATE TABLE searchIndex(id INTEGER PRIMARY KEY, name TEXT, type TEXT, path TEXT)"
      db.execute "CREATE UNIQUE INDEX anchor ON searchIndex (name, type, path)"
    }
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
      add_to_index name, "Section", "index.html##{e.parent.parent["id"]}"
    }
  end

  def add_to_index name, type, path
    with_db { |db|
      db.execute "INSERT OR IGNORE INTO searchIndex(name, type, path) VALUES ('#{name}', '#{type}', '#{path}');\n"
    }
  end

end

