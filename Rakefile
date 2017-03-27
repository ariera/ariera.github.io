require 'fileutils'
require "pry"
require 'active_support'
require 'active_support/all'
namespace :post do
  task :new, [:title] do  |t, args|
    date = Date.today.strftime("%Y-%m-%d")
    file_name = [date, args[:title]].join("-").parameterize
    file_name = "#{file_name}.markdown"
    file_path = File.join(File.dirname(__FILE__), "_posts", file_name)
    if File.exist?(file_path)
      puts "[SKIPPED] File already exists: #{file_path}"
    else
      # FileUtils.touch(file_path)
      File.open(file_path, "w") do |file|
        file.write(new_post_template(args[:title], DateTime.now))
      end
      puts "New post created: #{file_path}"
    end
  end
end


def new_post_template(title, date_time)
  if date_time.respond_to?(:strftime)
    date_time = date_time.strftime("%Y-%m-%d %H:%M:%S %z")
  end
  <<-TEMPLATE
---
layout: post
title:  "#{title}"
date:   #{date_time}
---

  TEMPLATE
end
