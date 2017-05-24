require 'rubygems'
require 'rake'
require 'rdoc'
require 'date'
require 'yaml'
require 'tmpdir'
require 'jekyll'

desc "Generate blog files"
task :generate do
  Jekyll::Site.new(Jekyll.configuration({
    "source"      => ".",
    "destination" => "_site"
  })).process
end


desc "Generate and publish blog to gh-pages"
task :publish => [:generate] do
  Dir.mktmpdir do |tmp|
    system "mv _site/* #{tmp}"
    system "git checkout -B gh-pages"
    system "rm -rf *"
    system "mv #{tmp}/* ."
    message = "Site updated at #{Time.now.utc}"
    system "git add ."
    system "git commit -am #{message.shellescape}"
    system "git push origin gh-pages --force"
    system "git checkout master"
    system "echo yolo"
  end
end

task :default => :publish

task :images_to_post, :tags do  |t,args|
  t = Time.now
   postdir = "_paintings/automagick" # put the post in the category subdir
  Dir.mkdir(postdir) unless File.exists?(postdir) # make the dir unless it exists
  
  
  Dir["allTheJpg/*.jpg"].each do |image|
    contents = "" # otherwise using it below will be badly scoped
    File.open("_drafts/yyyy-mm-dd-title.md", "rb") do |f|
      contents = f.read
    end

    imagename = File.basename(image)
  
    contents = contents.sub("%date%", t.strftime("%Y-%m-%d %H:%M:%S %z")).sub("%title%", imagename)
    contents = contents.gsub("%category%", "painting")
    contents = contents.sub("%image%", "/allTheJpg/"+imagename)
    contents = contents.sub("%thumbnail%", "/allTheJpg/"+imagename)
    contents = contents.sub("%tags%", args[:tags].gsub("\ ", "\,\ "))
    contents = contents.sub("%body%", "![A painting titled \""+imagename+"\" by painter Kyle Cunningham](/allTheJpg/"+imagename+ ")")

    filename = postdir + "/" + t.strftime("%Y-%m-%d-") + imagename.downcase.gsub( /[^a-zA-Z0-9_\.]/, '-').sub("\.jpg", '') + '.md'
    
    if File.exists? filename then
      puts "Post already exists: #{filename}"
      return
    end
    File.open(filename, "wb") do |f|
      f.write contents
    end
    puts "created #{filename}"


  end
end
