# == Dependencies ==============================================================
require 'rubygems'
require 'rake'
require 'yaml'
require 'fileutils'
require 'rbconfig'
#require 'flickraw'

# == Configuration =============================================================

# Set "rake watch" as default task
task :default => :watch

# Load the configuration file
CONFIG = YAML.load_file("_configs.yml")

# Get and parse the date
DATE = Time.now.strftime("%Y-%m-%d")

# Directories
POSTS = "_posts"
DRAFTS = "_drafts"

# == Helpers ===================================================================

# Execute a system command
def execute(command)
  system "#{command}"
end

# Chech the title
def check_title(title)
  if title.nil? or title.empty?
    raise "Please add a title to your file."
  end
end

# Transform the filename and date to a slug
def transform_to_slug(title, extension)
  characters = /("|'|!|\?|:|\s\z)/
  whitespace = /\s/
  "#{title.gsub(characters,"").gsub(whitespace,"-").downcase}.#{extension}"
end

# Read the template file
def read_file(template)
  File.read(template)
end

# Save the file with the title in the YAML front matter
def write_file(content, title, directory, filename)
  #parsed_content = "#{content.sub("title:", "title: \"#{title}\"")}"

  File.write("#{directory}/#{filename}", content)
  puts "#{filename} was created in '#{directory}'."
end


def template(name, category)
  t = Time.now
  contents = "" # otherwise using it below will be badly scoped
  File.open("_posts/yyyy-mm-dd-template.markdown", "rb") do |f|
    contents = f.read
  end

  postdir = "_posts/" + category # put the post in the category subdir
  Dir.mkdir(postdir) unless File.exists?(postdir) # make the dir unless it exists
  
  #make the image dir corresponding to this post
  Dir.mkdir("images/" + category) unless File.exists?("images/" + category) # make the dir unless it exists
  imgdir = "images/" + category + "/" + t.strftime("%Y-%m-%d-") + name.downcase.gsub( /[^a-zA-Z0-9_\.]/, '-')
  Dir.mkdir(imgdir) unless File.exists?(imgdir) # make the dir unless it exists

  # update variables in the yaml frontmatter for the post
  contents = contents.sub("%date%", t.strftime("%Y-%m-%d %H:%M:%S %z")).sub("%title%", name)
  contents = contents.sub("%category%", category)

  filename = postdir + "/" + t.strftime("%Y-%m-%d-") + name.downcase.gsub( /[^a-zA-Z0-9_\.]/, '-') + '.markdown'
  if File.exists? filename then
    puts "Post already exists: #{filename}"
    return
  end
  File.open(filename, "wb") do |f|
    f.write contents
  end
  puts "created #{filename}"
  execute("vim -g #{filename}") #open macvim with the file
end

# Create the file with the slug and open the default editor
def create_file(directory, filename, content, title, editor)
  if File.exists?("#{directory}/#{filename}")
    raise "The file already exists."
  else
    write_file(content, title, directory, filename)
    if editor && !editor.nil?
      sleep 1
      execute("#{editor} #{directory}/#{filename}")
    end
  end
end

# Get the "open" command
def open_command
  if RbConfig::CONFIG["host_os"] =~ /mswin|mingw/
    "start"
  elsif RbConfig::CONFIG["host_os"] =~ /darwin/
    "open"
  else
    "xdg-open"
  end
end

# == Tasks =====================================================================

# rake post["Title"]
desc "Create a post in _posts"
task :post, :title do |t, args|
  title = args[:title]
  template = CONFIG["post"]["template"]
  extension = CONFIG["post"]["extension"]
  editor = CONFIG["editor"]
  check_title(title)
  filename = "#{DATE}-#{transform_to_slug(title, extension)}"
  content = read_file(template)
  create_file(POSTS, filename, content, title, editor)
end


def move_with_path(src, dst)
  FileUtils.mkdir_p(File.dirname(dst))
  FileUtils.mv(src, dst)
end


# rake draft["Title","Category"]
desc "Create a post in _drafts"
task :draft, :title, :category do |t, args|
  title = args[:title]
  category = args[:category]
  template = CONFIG["post"]["template"]
  extension = CONFIG["post"]["extension"]
  editor = CONFIG["editor"]
  check_title(title)
  filename = transform_to_slug(title, extension)
  filename = "#{category}/#{DATE}-#{filename}"


  #fix template
  content = read_file(template)
  content = content.sub("%date%", Time.now.strftime("%Y-%m-%d %H:%M:%S %z")).sub("%title%", title)
  content = content.sub("%category%", category).sub("%layout%", category)


  #make sure category folder exists for this 

  puts "Checking if #{DRAFTS}/#{category} exists"
  FileUtils.mkdir_p("#{DRAFTS}/#{category}") unless File.directory? "#{DRAFTS}/#{category}"

  create_file(DRAFTS, filename, content, title, editor)

#make the image dir corresponding to this post
  Dir.mkdir("images/" + category) unless File.exists?("images/" + category)   
  imgdir = "images/#{category}/#{DATE}-#{title.downcase.gsub( /[^a-zA-Z0-9_\.]/, '-')}"
  Dir.mkdir(imgdir) unless File.exists?(imgdir) # make the dir unless it exists

end




# rake publish
desc "Move a post from _drafts to _posts"
task :publish do
  extension = CONFIG["post"]["extension"]
  #files = Dir["#{DRAFTS}/*.#{extension}"]
  Dir.glob("#{DRAFTS}/**/*.#{extension}").each_with_index do |file, index|
  #files.each_with_index do |file, index|
    puts "#{index + 1}: #{file}".sub("#{DRAFTS}/", "")
  end
  print "> "
  number = $stdin.gets
  if number =~ /\D/
    filename = files[number.to_i - 1].sub("#{DRAFTS}/", "")
    FileUtils.mv("#{DRAFTS}/#{filename}", "#{POSTS}/#{DATE}-#{filename}")
    puts "#{filename} was moved to '#{POSTS}'."
  else
    puts "Please choose a draft by the assigned number."
  end
end


def move_with_path(src, dst)
  FileUtils.mkdir_p(File.dirname(dst))
  FileUtils.mv(src, dst)
end


# rake publisher
desc "Move a post from _drafts to _posts"
task :publishers do
  extension = CONFIG["post"]["extension"]
  #files = Dir["#{DRAFTS}/*.#{extension}"]
  files = Dir.glob("#{DRAFTS}/**/*.#{extension}")
  #files = Dir.glob("#{DRAFTS}/**/*.#{extension}").each_with_index do |file, index|
  files.each_with_index do |file, index|
    puts "#{index + 1}: #{file}".sub("#{DRAFTS}/", "")
  end
  print "> "
  number = $stdin.gets
  if number =~ /\D/
    filename = File.basename(files[number.to_i - 1])
    #filename = File.basename(files[number.to_i - 1].sub("#{DRAFTS}/", "")
    filepath = files[number.to_i - 1]
    category = File.dirname(filepath).sub("#{DRAFTS}/", "")
    
    #puts  "post/data-filename: #{POSTS}/#{DATE}-#{filename}"
    #puts "filepath "  + filepath
    #puts "filename "  + filename
    #puts "category: #{category}"
    #puts  "post/category/date-filename: #{POSTS}/#{category}/#{DATE}-#{filename}"

    move_with_path("#{filepath}", "#{POSTS}/#{category}/#{filename}")
    #FileUtils.mv("#{DRAFTS}/#{filename}", "#{POSTS}/#{DATE}-#{filename}")
    puts "#{filename} was moved to '#{POSTS}'."
  else
    puts "Please choose a draft by the assigned number."
  end
end


# rake flickr
desc "download a photo from flickr"
task :flickr do

FlickRaw.api_key="b48f6a2cd40e4efef2f8f5b84122fac2"
FlickRaw.shared_secret="180e408215fb67eb"

list   = flickr.photos.getRecent

id     = list[0].id
secret = list[0].secret
info = flickr.photos.getInfo :photo_id => id, :secret => secret

puts info.title           # => "PICT986"
puts info.dates.taken     # => "2006-07-06 15:16:18"

sizes = flickr.photos.getSizes :photo_id => id

original = sizes.find {|s| s.label == 'Original' }
puts original.width       # => "800" -- may fail if they have no original marked image


end


# rake img 
desc "insert img tags into corresponding post"
task :img do
  extension = CONFIG["post"]["extension"]
  #files = Dir["#{DRAFTS}/*.#{extension}"]
  files = Dir.glob("#{DRAFTS}/**/*.#{extension}")
  files.each_with_index do |file, index|
    puts "#{index + 1}: #{file}".sub("#{DRAFTS}/", "")
  end
  print "> "
  number = $stdin.gets
  if number =~ /\D/
    filename = files[number.to_i - 1]
    basefilename =  File.basename(filename, '.*')    
    open(filename, 'a') do |f|

    onlyfilename = File.basename(files[number.to_i - 1])
    #filename = File.basename(files[number.to_i - 1].sub("#{DRAFTS}/", "")
    filepath = files[number.to_i - 1]
    category = File.dirname(filepath).sub("#{DRAFTS}/", "")

    puts "basefilename " + basefilename
    puts "filename " + filename
    puts "onlyfilename " + onlyfilename
    puts "filepath: " + filepath
    puts "category: " + category

    web_path = "http://moontoad.net/images/#{category}/"

    puts "Inserting image tags into #{filepath}"    
      Dir["images/#{category}/#{basefilename}/*"].each do |i|

        f << "![Title](/#{i} \"Alt Text\")\n" 
        puts "added: #{i}"

end
      end


  else
    puts "Please choose an image dir by assigned number"
  end
end



# rake imger 
desc "Move a post from _drafts to _posts"
task :imger do
  extension = CONFIG["post"]["extension"]
  #files = Dir["#{DRAFTS}/*.#{extension}"]
  files = Dir.glob("#{DRAFTS}/**/*.#{extension}")
  files.each_with_index do |file, index|
    puts "#{index + 1}: #{file}".sub("#{DRAFTS}/", "")
  end
  print "> "
  number = $stdin.gets
  if number =~ /\D/
    filename = files[number.to_i - 1]
    basefilename =  File.basename(filename, '.*')    
    open(filename, 'a') do |f|

    onlyfilename = File.basename(files[number.to_i - 1])
    #filename = File.basename(files[number.to_i - 1].sub("#{DRAFTS}/", "")
    filepath = files[number.to_i - 1]
    category = File.dirname(filepath).sub("#{DRAFTS}/", "")

    puts "basefilename " + basefilename
    puts "filename " + filename
    puts "onlyfilename " + onlyfilename
    puts "filepath: " + filepath
    puts "category: " + category







    puts "Inserting image tags into #{filepath}"    
      Dir["images/#{category}/#{basefilename}/*"].each do |i|

#only do this for new files
unless (File.basename(i) =~ /web/)
        image = Magick::Image.read(i).first

        webized_path = "images/#{category}/#{basefilename}/#{File.basename(i, '.*')}-web#{File.extname(i)}"
        web_path = "http://moontoad.net/images/#{category}/#{basefilename}/#{File.basename(i, '.*')}-web#{File.extname(i)}"


image.change_geometry!("680x") { |cols, rows, img|
    newimg = img.resize(cols, rows)
    newimg.write(webized_path)
    puts "resized image to:  #{webized_path}"
}


    
        f << "![Title](#{web_path} \"Alt Text\")\n" 
        puts "added: #{i}"
        puts "added: #{webized_path}"

end
      end
    
    end


  else
    puts "Please choose a draft by the assigned number."
  end
end



# rake page["Title"]
# rake page["Title","Path/to/folder"]
desc "Create a page (optional filepath)"
task :page, :title, :path do |t, args|
  title = args[:title]
  template = CONFIG["page"]["template"]
  extension = CONFIG["page"]["extension"]
  editor = CONFIG["editor"]
  directory = args[:path]
  if directory.nil? or directory.empty?
    directory = "./"
  else
    FileUtils.mkdir_p("#{directory}")
  end
  check_title(title)
  filename = transform_to_slug(title, extension)
  content = read_file(template)
  create_file(directory, filename, content, title, editor)
end

# rake build
desc "Build the site"
task :build do
  execute("jekyll build")
end

# rake watch
# rake watch[number]
# rake watch["drafts"]
desc "Serve and watch the site (with post limit or drafts)"
task :watch, :option do |t, args|
  option = args[:option]
  if option.nil? or option.empty?
    execute("jekyll serve --watch")
  else
    if option == "drafts"
      execute("jekyll serve --watch --drafts")
    else
      execute("jekyll serve --watch --limit_posts #{option}")
    end
  end
end

# rake preview
desc "Launch a preview of the site in the browser"
task :preview do
  port = CONFIG["port"]
  if port.nil? or port.empty?
    port = 4000
  end
  Thread.new do
    puts "Launching browser for preview..."
    sleep 1
    execute("#{open_command} http://localhost:#{port}/")
  end
  Rake::Task[:watch].invoke
end

# rake deploy["Commit message"]
desc "Deploy the site to a remote git repo"
task :deploy, :message do |t, args|
  message = args[:message]
  branch = CONFIG["git"]["branch"]
  if message.nil? or message.empty?
    raise "Please add a commit message."
  end
  if branch.nil? or branch.empty?
    raise "Please add a branch."
  else
    Rake::Task[:build].invoke
    execute("git add .")
    execute("git commit -m \"#{message}\"")
    execute("git push origin #{branch}")
  end
end

# rake transfer
desc "Transfer the site (remote server or a local git repo)"
task :transfer do
  command = CONFIG["transfer"]["command"]
  source = CONFIG["transfer"]["source"]
  destination = CONFIG["transfer"]["destination"]
  settings = CONFIG["transfer"]["settings"]
  if command.nil? or command.empty?
    raise "Please choose a file transfer command."
  elsif command == "robocopy"
    Rake::Task[:build].invoke
    execute("robocopy #{source} #{destination} #{settings}")
    puts "Your site was transfered."
  elsif command == "rsync"
    Rake::Task[:build].invoke
    execute("rsync #{settings} #{source} #{destination}")
    puts "Your site was transfered."
  else
    raise "#{command} isn't a valid file transfer command."
  end

end


#rake optimize
desc "Optimize all images in images"
task :optimize do 
    RakeFileUtils.verbose(false)
    start_time = Time.now

    file_list = FileList.new 'images/**/*.{gif,jpeg,jpg,png}'

    last_optimized_path = 'images/.last_optimized'
    if File.exists? last_optimized_path
      last_optimized = File.new last_optimized_path
      file_list.exclude do |f|
        File.new(f).mtime < last_optimized.mtime
      end
    end

    puts "\nOptimizing #{ file_list.size } image files."

    proc_cnt = 0
    skip_cnt = 0
    savings = {:old => Array.new, :new => Array.new}

    file_list.each_with_index do |f, cnt|
      puts "Processing: #{cnt+1}/#{file_list.size} #{f.to_s}"

      extension = File.extname(f).delete('.').gsub(/jpeg/,'jpg')
      ext_check = `file -b #{f} | awk '{print $1}'`.strip.downcase
      ext_check.gsub!(/jpeg/,'jpg')
      if ext_check != extension
        puts "\t#{f.to_s} is a: '#{ext_check}' not: '#{extension}' ..skipping"
        skip_cnt = skip_cnt + 1
        next
      end

      case extension
      when 'gif'
        `gifsicle -O2 #{f} > #{f}.n`
      when 'png'
        `pngcrush -q -rem alla -reduce -brute #{f} #{f}.n`
      when 'jpg'
        `jpegtran -copy none -optimize -perfect -progressive #{f} > #{f}.p`
        prog_size = File.size?("#{f}.p")

        `jpegtran -copy none -optimize -perfect #{f} > #{f}.np`
        nonprog_size = File.size?("#{f}.np")

        if prog_size < nonprog_size
          File.delete("#{f}.np")
          File.rename("#{f}.p", "#{f}.n")
        else
          File.delete("#{f}.p")
          File.rename("#{f}.np", "#{f}.n")
        end
      else
        skip_cnt = skip_cnt + 1
        next
      end

      old_size = File.size?(f).to_f
      new_size = File.size?("#{f}.n").to_f

      if new_size < old_size
        File.delete(f)
        File.rename("#{f}.n", f)
      else
        new_size = old_size
        File.delete("#{f}.n")
      end

      savings[:old] << old_size
      savings[:new] << new_size

      reduction = 100.0 - (new_size/old_size*100.0)

      puts "Output: #{sprintf "%0.2f", reduction}% | #{old_size.to_i} -> #{new_size.to_i}"
      proc_cnt = proc_cnt + 1
    end

    total_old = savings[:old].inject(0){|sum,item| sum + item}
    total_new = savings[:new].inject(0){|sum,item| sum + item}
    total_reduction = total_old > 0 ? (100.0 - (total_new/total_old*100.0)) : 0

    minutes, seconds = (Time.now - start_time).divmod 60
    puts "\nTotal run time: #{minutes}m #{seconds.round}s"

    puts "Files: #{file_list.size}\tProcessed: #{proc_cnt}\tSkipped: #{skip_cnt}"
    puts "\nTotal savings:\t#{sprintf "%0.2f", total_reduction}% | #{total_old.to_i} -> #{total_new.to_i} (#{total_old.to_i - total_new.to_i})"

    FileUtils.touch last_optimized_path
end

