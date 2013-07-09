require 'octokit'
require 'nokogiri'
require 'versionomy'

desc "generate api docs"
task :generate_api_docs do
  unless File.directory?("leapjs")
    system("git clone https://github.com/leapmotion/leapjs.git") or raise
  end
  Dir.chdir('leapjs') do
    system("git reset --hard") or raise
    system("git checkout master") or raise
    system("git pull") or raise
    # iterate through all the tags, doing a checkout
    tags = Octokit.tags("leapmotion/leapjs")
    tag_names = tags.map(&:name).sort_by{|v| Versionomy.parse(v)}.reverse

    tag_names.to_a[0...1].each do |tag|
      output_dir = "./api-docs/#{tag}"
      system("git reset --hard") or raise
      system("git checkout #{tag}") or raise
      system("npm install") or raise
      system("mkdir -p #{output_dir}") or raise
      system("cp ../INDEX.md README.md") or raise
      system("./node_modules/jsdoc/jsdoc -c jsdoc_conf.json lib README.md -d #{output_dir}") or raise
    end
  end
end

desc "compile api docs"
task :compile_api_docs do
  Dir['leapjs/api-docs/*'].each do |dir|
    tag = File.basename(dir)
    output_dir = "./leapjs/api-docs/#{tag}"
    # put them all together!
    files = Dir[output_dir + "/*.html"].sort_by{|f| File.size(f) }.map {|f| File.basename(f) }
    # we'll treat these specially
    special_files = ['index.html', 'Leap.html']
    special_files.each {|f| files.delete(f)}
    parts = (special_files + files).map do |f|
      doc = Nokogiri::HTML(File.read("#{output_dir}/#{f}"))
      main_node = doc.css("#main").first
      main_node['id'] = File.basename(f, ".*").downcase.gsub('.', '-')
      main_node['class'] = "doc-section"
      5.downto(1) do |h|
        main_node.css("h#{h}").to_a.each do |node|
          node.name = "h#{h.succ}"
        end
      end
      if special_files.first != f
        main_node.css('a').each do |a|
          new_node = doc.create_element "span"
          new_node.inner_html = a.inner_html
          a.replace new_node
        end
      end
      main_node.to_s
    end

    File.open("./views/api/#{tag}-docs.erb", "w") do |f|
      f << parts.join("\n")
    end
  end
end