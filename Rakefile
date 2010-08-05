begin
  require 'jeweler'
rescue LoadError
  puts "Jeweler not available. Install it with: sudo gem install technicalpickles-jeweler -s http://gems.github.com"
  exit 1
end
require 'rubygems'
require 'activesupport'
require 'rake/testtask'
require 'rake/rdoctask'
require 'rcov/rcovtask'

Jeweler::Tasks.new do |s|
  s.name              = "graticule"
  s.rubyforge_project = "graticule"
  s.author            = 'Brandon Keepers'
  s.email             = 'brandon@opensoul.org'
  s.summary           = "API for using all the popular geocoding services."
  s.description       = 'Graticule is a geocoding API that provides a common interface to all the popular services, including Google, Yahoo, Geocoder.us, and MetaCarta.'
  s.homepage          = "http://github.com/collectiveidea/graticule"
  s.add_dependency    "activesupport"
  s.add_dependency    'happymapper', '>=0.3.0'
  s.has_rdoc          = true
  s.extra_rdoc_files  = ["README.txt"]
  s.rdoc_options      = ["--main", "README.rdoc", "--inline-source", "--line-numbers"]
  s.test_files        = Dir['test/**/*']
end

desc 'Default: run unit tests.'
task :default => :test

desc 'Run the unit tests'
Rake::TestTask.new(:test) do |t|
  t.libs << 'lib' << 'test'
  t.pattern = 'test/**/*_test.rb'
  t.verbose = true
end

desc 'Generate documentatio'
Rake::RDocTask.new(:rdoc) do |rdoc|
  rdoc.rdoc_dir = 'rdoc'
  rdoc.title    = 'Graticule'
  rdoc.options << '--line-numbers' << '--inline-source'
  rdoc.rdoc_files.include('README.txt')
  rdoc.rdoc_files.include('lib/**/*.rb')
end

namespace :test do
  desc "just rcov minus html output"
  Rcov::RcovTask.new(:coverage) do |t|
    # t.libs << 'test'
    t.test_files = FileList['test/**/*_test.rb']
    t.output_dir = 'coverage'
    t.verbose = true
    t.rcov_opts = %w(--exclude test,/usr/lib/ruby,/Library/Ruby,$HOME/.gem --sort coverage)
  end
end

require 'rake/contrib/sshpublisher'
namespace :rubyforge do

  desc "Release gem and RDoc documentation to RubyForge"
  task :release => ["rubyforge:release:gem", "rubyforge:release:docs"]

  namespace :release do
    desc "Publish RDoc to RubyForge."
    task :docs => [:rdoc] do
      config = YAML.load(
          File.read(File.expand_path('~/.rubyforge/user-config.yml'))
      )

      host = "#{config['username']}@rubyforge.org"
      remote_dir = "/var/www/gforge-projects/the-perfect-gem/"
      local_dir = 'rdoc'

      Rake::SshDirPublisher.new(host, remote_dir, local_dir).upload
    end
  end
end

require 'active_support'
require 'net/http'
require 'uri'
RESPONSES_PATH = File.dirname(__FILE__) + '/test/fixtures/responses'

def cache_responses(service)
  test_config[service.to_s]['responses'].each do |file,url|
    File.open("#{RESPONSES_PATH}/#{service}/#{file}", 'w') do |f|
      f.puts Net::HTTP.get(URI.parse(url))
    end
  end
end

def test_config
  file = File.dirname(__FILE__) + '/test/config.yml'
  raise "Copy config.yml.default to config.yml and set the API keys" unless File.exists?(file)
  @test_config ||= YAML.load(File.read(file)).tap do |config|
    config.each do |service,values|
      values['responses'].each {|file,url| update_placeholders!(values, url) }
    end
  end
end

def update_placeholders!(config, thing)
  config.each do |option, value|
    thing.gsub!(":#{option}", value) if value.is_a?(String)
  end
end

namespace :test do
  namespace :cache do
    desc 'Cache test responses from all the geocoders'
    task :all => test_config.keys

    test_config.keys.each do |service|
      desc "Cache test responses for #{service}"
      task service do
        cache_responses(service)
      end
    end
  end
end

