# Asking for title
def ask message
print message
STDIN.gets.chomp
end
title = ask('Title: ')

#Create new a post
desc "Default 'rake' command creates a new post"
task :default do
  filename = "#{Time.now.strftime('%Y-%m-%d')}-#{title.gsub(/\s/, '_').downcase}.markdown"
  path = File.join("_posts", filename)
  if File.exist? path; raise RuntimeError.new("File exists #{path}"); end
  File.open(path, 'w') do |file|
  	file.write <<-EOS
---
layout: post
title: #{title}
date: #{Time.now.strftime('%Y-%m-%d %k:%M:%S')}
categories:
---
EOS
end

# invoke Textmate to edit file
# sh "mate #{path}"
end
