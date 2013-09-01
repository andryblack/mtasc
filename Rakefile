WGET_VERSION = "1.11.4"
PCRE_VERSION = "8.33"
GODI_VERSION = "20091222"

desc "Remove cruft generated by compilation"
task :clean do
  files_to_clean = File.read('.gitignore').split("\n").map { |i| "**/#{i}"}
  p files_to_clean
  Dir.glob(files_to_clean).each do |file|
    rm file if !File.directory? file
  end
  system('rm -Rf tmp')
end

desc "Install all required tools for compilation"
task :setup do
  mkdir_p "tmp"
  Dir.chdir("tmp") do
    # wget
    if File.exist? "/usr/local/bin/wget"
      puts "wget is already installed"
    else
      if !File.exist? "wget-#{WGET_VERSION}.tar.bz2"
        system("curl -a -O http://ftp.gnu.org/gnu/wget/wget-#{WGET_VERSION}.tar.bz2")
      end
      system("bunzip2 wget-#{WGET_VERSION}.tar.bz2")
      system("tar xf wget-#{WGET_VERSION}.tar")
      Dir.chdir("wget-#{WGET_VERSION}") do
        system("./configure")
        system("make install")
      end
    end

    # pcre
    if File.exist? "/usr/local/bin/pcregrep"
      puts "pcre is already installed"
    else
      if !File.exist? "pcre-#{PCRE_VERSION}.tar.bz2"
        system("curl -a -O ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-#{PCRE_VERSION}.tar.bz2")
      end
      system("bunzip2 pcre-#{PCRE_VERSION}.tar.bz2")
      system("tar xf pcre-#{PCRE_VERSION}.tar")
      Dir.chdir("pcre-#{PCRE_VERSION}") do
        system("./configure")
        system("make install")
      end
    end

    # godi
    if File.exist? "/usr/local/godi/bin/ocaml"
      puts "GODI is already installed"
    else
      if !File.exist? "godi-rocketboost-#{GODI_VERSION}.tar.gz"
        system("curl -a -O http://download.camlcity.org/download/godi-rocketboost-#{GODI_VERSION}.tar.gz")
      end
      system("tar xvzf godi-rocketboost-#{GODI_VERSION}.tar.gz")
      mkdir_p "godi"
      Dir.chdir("godi-rocketboost-#{GODI_VERSION}") do
        system("./bootstrap --prefix /usr/local/godi --localbase /usr/local/godi")
        system("PATH=/usr/local/godi/bin:/usr/local/godi/sbin:$PATH && ./bootstrap_stage2")
      end
    end
  end
end

def version
  File.read("VERSION").chomp
end

desc "Compile MTASC"
task :compile => :setup do
  system("PATH=/usr/local/godi/bin:/usr/local/godi/sbin:$PATH && ocaml install.ml")
end

desc "Package MTASC for release"
task :package => :compile do
  system("rm -Rf pkg/#{version}") if File.exist?("pkg/#{version}")
  mkdir_p "pkg/#{version}"
  system("cp -R bin/mtasc* pkg/#{version}/")
  system("cp LICENSE pkg/#{version}/")
  system("cp LICENSE pkg/#{version}/")
  system("cp -R src/mtasc/std* pkg/#{version}/")
end

desc "Install MTASC to /usr/local"
task :install => :compile do
  system("rm /usr/local/bin/mtasc")
  system("cp bin/mtasc /usr/local/bin/mtasc-#{version}")
  system("ln -s /usr/local/bin/mtasc-#{version} /usr/local/bin/mtasc")
  system("cp -R src/mtasc/std* /usr/local/bin/")
end

task :default => :compile