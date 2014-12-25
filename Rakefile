require 'nokogiri'
require 'sqlite3'

DOCUMENTATION_URL = 'http://pouchdb.com/'

task default: [
  :fetch_documentation,
  :build_index
]

task :fetch_documentation do
  target_directory = documents_path
  wget_command = <<-END_OF_COMMAND
    wget --mirror
         --convert-links
         --page-requisites
         -nH
         --directory-prefix=#{target_directory}
         #{DOCUMENTATION_URL}
    END_OF_COMMAND
  system wget_command.gsub(/\n/, "")
end

task :build_index do
  db_path = resources_path "/docSet.dsidx"
  db = SQLite3::Database.new db_path

  create_docset_table(db)
  parse_docset_into_db(db)
end

private
def create_docset_table(db)
  db.execute <<-SQL
    CREATE TABLE IF NOT EXISTS searchIndex(id INTEGER PRIMARY KEY, name TEXT, type TEXT, path TEXT);
    CREATE UNIQUE INDEX anchor ON searchIndex (name, type, path);
  SQL
end

def parse_docset_into_db(db)
  guides_document_path = documents_path "/guides/index.html"
  guides_file = File.open(guides_document_path)
  parsed_guides= Nokogiri::HTML(guides_file)
  guides_file.close

  parsed_guides.css('#sidebar ul.nav li a').each do |guide|
    name = guide.content
    path = "/guides/#{guide.attr('href')}"
    insert_statement = <<-SQL
      INSERT OR IGNORE INTO searchIndex(name, type, path) VALUES ('#{name}', 'Guide', '#{path}');
    SQL

    db.execute insert_statement
  end
end

def resources_path(path = "")
  "#{Dir.getwd}/PouchDB.docset/Contents/Resources#{path}"
end

def documents_path(path = "")
  "#{resources_path("/Documents")}#{path}"
end
