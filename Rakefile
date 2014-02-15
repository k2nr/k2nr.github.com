task :post, [:title] do |task, args|
  t = Time.new
  date_s = sprintf("%04d-%02d-%02d", t.year, t.month, t.day)
  filepath = sprintf("source/blog/%s-%s.md", date_s, args.title)
  template = <<-EOS
---
title: #{args.title}
date: #{date_s}
tags:
---
  EOS

  `echo "#{template}" > #{filepath}`
  puts "file created: #{filepath}"
end
