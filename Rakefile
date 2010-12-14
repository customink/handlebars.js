require "rubygems"
require "bundler/setup"

file "lib/handlebars/parser.js" => "src/handlebars.yy" do
  system "jison src/handlebars.yy"
  sh "mv handlebars.js lib/handlebars/parser.js"
end

task :compile => "lib/handlebars/parser.js"

desc "run the spec suite"
task :spec => [:release] do
  system "rspec -cfs spec"
end

task :default => [:compile, :test]

def remove_exports(string)
  match = string.match(%r{^// BEGIN\(BROWSER\)\n(.*)\n^// END\(BROWSER\)}m)
  match ? match[1] : string
end

minimal_deps = %w(parser compiler ast jison_ext handlebars_lexer runtime utils).map do |file|
  "lib/handlebars/#{file}.js"
end

debug_deps = %w(parser compiler ast jison_ext handlebars_lexer printer runtime utils).map do |file|
  "lib/handlebars/#{file}.js"
end

minimal_deps << "lib/handlebars.js"
debug_deps   << "lib/handlebars.js" << "lib/handlebars/debug.js"

minimal_deps.unshift "dist"
debug_deps.unshift   "dist"

def build_for_task(task)
  FileUtils.rm_rf("dist/*")

  contents = []
  task.prerequisites.each do |filename|
    next if filename == "dist"

    contents << "// #{filename}\n" + remove_exports(File.read(filename)) + ";"
  end

  File.open(task.name, "w") do |file|
    file.puts contents.join("\n")
  end
end

file "dist/handlebars.js" => minimal_deps do |task|
  build_for_task(task)
end

file "dist/handlebars.debug.js" => debug_deps do |task|
  build_for_task(task)
end

directory "dist" do
  FileUtils.mkdir_p("dist")
end

task :build => [:compile, "dist", "dist/handlebars.js"]
task :debug => [:compile, "dist", "dist/handlebars.debug.js"]

desc "build the build and debug versions of handlebars"
task :release => [:build, :debug]
