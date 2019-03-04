require 'rake/packagetask'
require_relative 'config/application'

Rails.application.load_tasks


Rake::PackageTask.new("rake", "1.2.3") do |p|
	  path = ENV["ARCHIVE_PATH"] || Dir.pwd
	    exclude = ENV["EXCLUDE"] || ""

	      p.name = path.split("/").last
	        p.need_tar_gz = true
		  p.package_files.include("#{path}/*")
		    p.package_files.exclude(exclude.split(","))
		      p.package_dir = "build"
end

desc "Archives a folder into a tar.gz file"
task archive: [:package]
