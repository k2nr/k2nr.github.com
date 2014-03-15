task :post, [:title] do |task, args|
  t = Time.new
  date_s = sprintf("%04d-%02d-%02d %02d:%02d:%02d", t.year, t.month, t.day, t.hour, t.min, t.sec)
  template = <<-EOS
---
title: "#{args.title}"
date: "#{date_s}"
---
  EOS

  filepath = sprintf("source/blog/%02d-%02d-%02d-%s.md", t.year, t.month, t.day, args.title)
  `echo "#{template}" > #{filepath}`
  puts "file created: #{filepath}"
end
