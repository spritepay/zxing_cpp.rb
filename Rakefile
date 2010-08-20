# -*- mode:ruby -*-

java = RUBY_PLATFORM =~ /java/

gem "hoe", "~> 2.5"
require 'hoe'

Hoe.plugin :debugging, :doofus, :git
Hoe.plugins.delete :rubyforge

HOE = Hoe.spec 'zxing' do
  developer "Steven Parkes", "smparkes@smparkes.net"

  self.extra_rdoc_files         = FileList["*.rdoc"]
  self.history_file             = "CHANGELOG.rdoc"
  self.readme_file              = "README.rdoc"

  self.spec_extras[:extensions] = %w(Rakefile)

  if java
    clean_globs << "vendor/zxing/core/core.jar"
    clean_globs << "vendor/zxing/javase/javase.jar"
  else
    clean_globs << "**/*.a"
    clean_globs << "**/*.so"
    clean_globs << "**/*.bundle"
    clean_globs << "**/*.dylib"
    clean_globs << "**/*.pyc"
    clean_globs << "lib/zxing/Makefile"
    clean_globs << "lib/zxing/zxing.o"
    clean_globs << "**/.sconsign.dblite"
  end
  clean_globs << "**/build"

end

shared_ext = ".so"
Dir["lib/zxing/zxing.*"].each do |file|
  case file
  when %r{\.so$}; shared_ext = ".so"
  when %r{\.dylib$}; shared_ext = ".dylib"
  when %r{\.bundle$}; shared_ext = ".bundle"
  end
end

file "vendor/zxing" do
  sh "git submodule update --init"
end

task :compile => "vendor/zxing"

Hoe.plugin :debugging, :doofus, :git
Hoe.plugins.delete :rubyforge

task :clean do
  if File.exist? "vendor/zxing/cpp/build" 
    Dir.chdir "vendor/zxing/cpp" do
      sh "python scons/scons.py -c"
    end
  end
  if java
    if File.exist? "vendor/zxing/core/build"
      Dir.chdir "vendor/zxing/core" do
        sh "ant clean"
      end
    end
    if File.exist? "vendor/zxing/javase/build" 
      Dir.chdir "vendor/zxing/javase" do
        sh "ant clean"
      end
    end
  end
end

task(:test).clear

zxing = "vendor/zxing"

subdirs = [ :qrcode, :datamatrix, :negative, :oned ]
if java
  subdirs << :pdf417
end

if java
  if File.exists? "#{zxing}/core"
    file "#{zxing}/core/core.jar" do
      sh "ant -f #{zxing}/core/build.xml compile"
    end
    file "lib/zxing/core.jar" => "#{zxing}/core/core.jar" do
      cp "#{zxing}/core/core.jar", "lib/zxing/core.jar"
    end
  end

  if File.exists? "#{zxing}/javase"
    file "#{zxing}/javase/javase.jar" => "#{zxing}/core/core.jar" do
      sh "ant -f #{zxing}/javase/build.xml build"
    end
    file "lib/zxing/javase.jar" => "#{zxing}/javase/javase.jar" do
      cp "#{zxing}/javase/javase.jar", "lib/zxing/javase.jar"
    end
  end

  namespace :compile do
    task :core do
      sh "ant -f #{zxing}/core/build.xml compile && cp #{zxing}/core/core.jar lib/zxing/core.jar"
    end
    task :javase do
      sh "ant -f #{zxing}/javase/build.xml build && cp #{zxing}/javase/javase.jar lib/zxing/javase.jar"
    end
  end

  task :compile => [ "lib/zxing/core.jar", "lib/zxing/javase.jar" ]
  task :recompile => [ "compile:core", "compile:javase" ]
end

if !java
  file "vendor/zxing/cpp/build/libzxing.a" do
    Dir.chdir "vendor/zxing/cpp" do
      sh "python scons/scons.py PIC=yes lib"
    end
  end
  file "lib/zxing/Makefile" => [ "lib/zxing/extconf.rb",
                                 "vendor/zxing/cpp/build/libzxing.a" ] do
    Dir.chdir "lib/zxing" do
      ruby "extconf.rb"
    end
  end
  file "lib/zxing/zxing#{shared_ext}" => [ "lib/zxing/Makefile",
                                           "lib/zxing/zxing.cc",
                                           "vendor/zxing/cpp/build/libzxing.a" ] do
    sh "cd lib/zxing && make"
  end
  task :recompile do
    file("vendor/zxing/cpp/build/libzxing.a").execute
    rm "lib/zxing/zxing#{shared_ext}"
    file("lib/zxing/zxing#{shared_ext}").execute
  end
  task :compile => "lib/zxing/zxing#{shared_ext}"
end

namespace :test do
  subdirs.each do |subdir|
    namespace subdir do
      desc "run #{subdir} tests"
      task :run do
        args = [
                # "ruby", "-Ilib",
                "test/runner.rb" ] +
          Dir["**/#{subdir}/*BlackBox*TestCase.java"]
        args.unshift "valgrind" if ENV["valgrind"]
        sh args.join(" ")
      end
    end
    desc "compile and run #{subdir} tests"
    task subdir => [ :compile, "test:#{subdir}:run" ]
  end
  task :run => subdirs.map { |subdir| "test:#{subdir}:run" }
end

task :test, [ :pattern ] => :compile do |t, args|
  if args[:pattern]
    args = [ "ruby", "-Ilib", "test/runner.rb", args[:pattern] ]
    args.unshift "valgrind" if ENV["valgrind"]
    sh args.join(" ")
  else
    subdirs.each { |subdir| task("test:#{subdir}:run").execute }
  end
end

task :default => :test

if java
  task :gem => [ "lib/zxing/core.jar", "lib/zxing/javase.jar" ]
end

# manifest stuff:
# egrep -v '(/actionscript/|/zxingorg/|/zxing.appspot|symbian|/rim/|javame|zxing/jruby|/iphone/|csharp/|/test/|/bug/|/android|.git|.valgrind|.xcodeproj|vendor/zxing/core)' Manifest.txt | less