# gprbuild-pkg/Rakefile

PKG_NAME     = "gprbuild-gcc14"
VERSION      = "25.0.0"
GPRBUILD     = "gprbuild-#{VERSION}"
GPRCONFIG_KB = "gprconfig_kb-#{VERSION}"
XMLADA       = "xmlada-#{VERSION}"

RAKE_ROOT    = File.expand_path(File.dirname(__FILE__))
STAGE_DIR    = File.join(RAKE_ROOT, 'stage')
PATH         = "#{RAKE_ROOT}/#{GPRBUILD}/bootstrap/bin:$PATH"
GPR_PROJECT_PATH = "#{RAKE_ROOT}/#{GPRBUILD}/bootstrap/share/gpr"

GCC      = "/usr/local/bin/gcc14"
GNATMAKE = "/usr/local/bin/gnatmake14"
GMAKE    = "/usr/local/bin/gmake"
TARGET   = `#{GCC} -dumpmachine`.strip

# 🔗 Create symbolic links (requires sudo)
puts "🔗 Creating gcc/gnatmake symlinks..."
puts "⚠️ At this step, sudo privileges are required. A password prompt may appear."
sh "sudo ln -sf #{GCC} /usr/local/bin/gcc"
sh "sudo ln -sf /usr/local/bin/gnatmake14 /usr/local/bin/gnatmake"
sh "sudo ln -sf /usr/local/bin/gnatls14   /usr/local/bin/gnatls"
sh "sudo ln -sf /usr/local/bin/gnatbind14 /usr/local/bin/gnatbind"
sh "sudo ln -sf /usr/local/bin/gnatlink14 /usr/local/bin/gnatlink"

FILES_TO_DOWNLOAD = [
  {
    name: "#{GPRBUILD}.tar.gz",
    url: "https://github.com/AdaCore/gprbuild/archive/refs/tags/v25.0.0.tar.gz",
    sha512: "eb2d7072194323cae90acd0c8683eeb6a806ef6ff2ed4d3496e8b94c5b63dae8a428ec428a3610b380df7e122d7a00d9e9634ef06b5369b165536c99209602ce"
  },
  {
    name: "#{GPRCONFIG_KB}.tar.gz",
    url: "https://github.com/AdaCore/gprconfig_kb/archive/refs/tags/v25.0.0.tar.gz",
    sha512: "afc1754efdf6e3cbff9752a182cd063f83965c6a13e53930a14f806a46e3cbfb0afed8f8e11b098986227f27a1a67b45d22369adaa39a5dc1f2a8cc494f789e8"
  },
  {
    name: "#{XMLADA}.tar.gz",
    url: "https://github.com/AdaCore/xmlada/archive/refs/tags/v25.0.0.tar.gz",
    sha512: "c57db78e3afd20862c3275d3d0874ada1748e98df06a76841cb3dca3686b29c7693835a591ca5789dca2d3d6ba9677c9082df94857e180e0758a5b77fafc40c0"
  }
]

file_list = FILES_TO_DOWNLOAD.map { |f| f[:name] }

task default: 'pkg'

task 'pkg' => 'build-gprbuild' do
  puts "=> Creating stage directory: #{STAGE_DIR}"
  mkdir_p "#{STAGE_DIR}/usr/local/bin"
  mkdir_p "#{STAGE_DIR}/usr/local/libexec/gprbuild"

  sh "install -m755 #{GPRBUILD}/exe/production/gprbuild   #{STAGE_DIR}/usr/local/bin"
  sh "install -m755 #{GPRBUILD}/exe/production/gprclean   #{STAGE_DIR}/usr/local/bin"
  sh "install -m755 #{GPRBUILD}/exe/production/gprconfig  #{STAGE_DIR}/usr/local/bin"
  sh "install -m755 #{GPRBUILD}/exe/production/gprinstall #{STAGE_DIR}/usr/local/bin"
  sh "install -m755 #{GPRBUILD}/exe/production/gprls      #{STAGE_DIR}/usr/local/bin"
  sh "install -m755 #{GPRBUILD}/exe/production/gprname    #{STAGE_DIR}/usr/local/bin"
  sh "install -m755 #{GPRBUILD}/exe/production/gprslave   #{STAGE_DIR}/usr/local/bin"
  sh "install -m755 #{GPRBUILD}/exe/production/gprbind    #{STAGE_DIR}/usr/local/libexec/gprbuild"
  sh "install -m755 #{GPRBUILD}/exe/production/gprlib     #{STAGE_DIR}/usr/local/libexec/gprbuild"

  puts "=> Stripping executables in stage area..."
  sh "strip #{STAGE_DIR}/usr/local/bin/*"
  sh "strip #{STAGE_DIR}/usr/local/libexec/gprbuild/*"

  sh "cd #{GPRBUILD} && find share -type d -exec install -d #{STAGE_DIR}/usr/local/{} \\;"
  sh "cd #{GPRBUILD} && find share -type f -exec install -m 644 {} #{STAGE_DIR}/usr/local/{} \\;"

ver = `uname -r`.split(".")[0] # 14.3-RELEASE

manifest_content = <<~MANIFEST
  name: #{PKG_NAME}
  origin: local/#{PKG_NAME}
  version: "#{VERSION}"
  comment: "Multi-language build tool based on GPR project files"
  desc: "Gprbuild is a build tool that automates the construction of multi-language systems. It uses project files (.gpr) to describe the source files, dependencies, and compiler switches."
  maintainer: "Hodong Kim <hodong@nimfsoft.art>"
  www: "https://github.com/AdaCore/gprbuild"
  abi: "FreeBSD:#{ver}:amd64"
  arch: "freebsd:#{ver}:x86:64"
  prefix: /usr/local
  licenses: [GPLv3+]
  deps: {}
MANIFEST

  File.write("+MANIFEST", manifest_content)
  license_dir = "#{STAGE_DIR}/usr/local/share/licenses/gprbuild-#{VERSION}"
  mkdir_p license_dir
  sh "install -m644 gprbuild-#{VERSION}/COPYING3 #{license_dir}/GPLv3+"
  sh "install -m644 gprbuild-#{VERSION}/COPYING.RUNTIME #{license_dir}/RUNTIME_EXCEPTION"

  sh "find ./stage/usr/ -type f | sed 's/\\.\\/stage//g' >  plist"
  sh "find ./stage/usr/ -type l | sed 's/\\.\\/stage//g' >> plist"
  sh "pkg create -m . -r #{STAGE_DIR} -p plist -o ."
end

file 'build-gprbuild' => ['build-xmlada', 'setup'] do
  cd "#{GPRBUILD}" do
    sh "PATH=#{PATH} GPR_PROJECT_PATH=#{GPR_PROJECT_PATH} #{GMAKE} -f Makefile PROCESSORS=1 ENABLE_SHARED=yes all"
  end

  sh "touch build-gprbuild"
end

file 'build-xmlada' => 'bootstrap' do
  cd "#{XMLADA}" do
    sh "./configure --prefix=#{RAKE_ROOT}/#{GPRBUILD}/bootstrap --disable-shared --enable-build=Production"
    sh "PATH=#{PATH} #{GMAKE} -f Makefile PROCESSORS=1 all"
    sh "PATH=#{PATH} #{GMAKE} -f Makefile PROCESSORS=1 install"
  end

  sh "touch build-xmlada"
end

file 'setup' => 'bootstrap' do
  cd "#{GPRBUILD}" do

    command_parts = [
      "#{GMAKE}",
      "-f", "Makefile",
      "prefix=#{STAGE_DIR}/usr/local",
      "ENABLE_SHARED=yes",
      "BUILD=production",
      "PROCESSORS=1",
      "TARGET=#{TARGET}",
      "SOURCE_DIR=#{RAKE_ROOT}/gprbuild-25.0.0",
      "setup"
    ]

    sh command_parts.join(' ')

    command_parts = [
      "./gprconfig",
      "--batch",
      "--target=#{TARGET}",
      "--config=Ada,,default,/usr/local/bin,GNAT"
    ]

    sh command_parts.join(' ')
  end

  sh "touch setup"
end

file 'bootstrap' => [GPRBUILD,
                     XMLADA,
                     GPRCONFIG_KB,
                     'patch_xml'] do
  puts "🚀 Bootstrapping gprbuild..."
  cd "#{GPRBUILD}" do

    command_parts = [
      "./bootstrap.sh",
      "--with-xmlada=../xmlada-25.0.0",
      "--with-kb=../gprconfig_kb-25.0.0",
      "--prefix=./bootstrap"
    ]

    sh "GNATMAKE=#{GNATMAKE} CC=#{GCC} " + command_parts.join(' ')
  end
  sh "touch bootstrap"
  puts "👍 Bootstrap completed."
end

file 'patch_xml' => "#{GPRCONFIG_KB}/db/compilers.xml" do
  puts "✍️ Patching compilers.xml for FreeBSD..."
  cd "#{GPRCONFIG_KB}" do
    sh "patch -N -p1 < ../compilers-freebsd.patch"
  end
  sh "touch patch_xml"
  puts "👍 compilers.xml patched successfully."
end

directory GPRBUILD => "#{GPRBUILD}.tar.gz" do
  puts "📦 Unpacking #{GPRBUILD}.tar.gz..."
  sh "tar -xzf #{GPRBUILD}.tar.gz"
end

directory XMLADA => "#{XMLADA}.tar.gz" do
  puts "📦 Unpacking #{XMLADA}.tar.gz..."
  sh "tar -xzf #{XMLADA}.tar.gz"
end

file "#{GPRCONFIG_KB}/db/compilers.xml" => GPRCONFIG_KB

directory GPRCONFIG_KB => "#{GPRCONFIG_KB}.tar.gz" do
  puts "📦 Unpacking #{GPRCONFIG_KB}.tar.gz..."
  sh "tar -xzf #{GPRCONFIG_KB}.tar.gz"
end

FILES_TO_DOWNLOAD.each do |f|
  file f[:name] do
    puts "⬇️ Downloading #{f[:name]}..."
    sh "curl -L -O -J #{f[:url]}"

    puts "🔍 Verifying SHA512 checksum for #{f[:name]}..."

    sha_cmd = 'sha512sum'

    calculated_sum = `#{sha_cmd} #{f[:name]}`.split.first

    if calculated_sum == f[:sha512]
      puts "👍 Checksum OK for #{f[:name]}."
    else
      $stderr.puts "  Checksum MISMATCH for #{f[:name]}!"
      $stderr.puts "  Expected: #{f[:sha512]}"
      $stderr.puts "  Got:      #{calculated_sum}"
      rm f[:name]
      raise "Aborting due to checksum mismatch."
    end
  end
end

task :clean do
  puts "Cleaning up unpacked source directories..."
  dirs_to_clean = file_list.map { |f| f.gsub('.tar.gz', '') }
  rm_rf(file_list,                    verbose: true)
  rm_rf(dirs_to_clean,                verbose: true)
  rm_rf(STAGE_DIR,                    verbose: true)
  rm_rf('patch_xml',                  verbose: true)
  rm_rf('bootstrap',                  verbose: true)
  rm_rf('build-gprbuild',             verbose: true)
  rm_rf('build-xmlada',               verbose: true)
  rm_rf("#{PKG_NAME}-#{VERSION}.pkg", verbose: true)
  rm_rf('setup',                      verbose: true)
  rm_rf('+MANIFEST',                  verbose: true)
  rm_rf('plist',                      verbose: true)
end
