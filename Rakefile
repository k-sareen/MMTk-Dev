# Rakefile
task default: [:hello]

$command_finished = true
at_exit { $command_finished || sleep(1) }

def 🔵(command, cwd: '.')
    puts "🔵 #{command}"
    $command_finished = false
    res = system("cd #{cwd} && #{command}")
    $command_finished = true
    res || raise('❌')
end

ENV["RUST_BACKTRACE"] = "1"
ENV['MMTK_PLAN'] = ENV['gc'] || 'NoGC'
if ENV.has_key?("threads")
    ENV['MMTK_THREADS'] = ENV["threads"]
end

namespace "v8" do
    profile = ENV["profile"] || 'optdebug-mmtk'
    v8 = "./v8"
    mmtk = "./mmtk-v8/mmtk"
    no_max_failures = true

    task :build do
        🔵 "cargo build", cwd:mmtk
        🔵 "./tools/dev/gm.py x64.#{profile}.all", cwd:v8
    end

    task :test => :build do
        no_max_failures = no_max_failures ? "--exit-after-n-failures=0" : ""
        🔵 "./tools/dev/gm.py x64.#{profile}.checkall #{no_max_failures}", cwd:v8
    end

    task :gdb do
        cmd = ARGV[(ARGV.index("--") + 1)..-1].join(" ")
        cmd.gsub! /x64\.(release|optdebug)/, 'x64.debug'
        profile = cmd.match(/x64\.(?<name>[\w\-_\d]+)/)[:name]
        Rake::Task["v8:build"].invoke
        🔵 "gdb -ex='set confirm on' -ex r -ex q --args #{cmd}", cwd:v8
        exit 0
    end
end

namespace "jdk" do
    profile = ENV["profile"] || 'fastdebug'
    heap = ENV["heap"] || '100M'
    benchmark = ENV["bench"] || 'xalan'
    n = ENV["n"] || '1'

    vm_args = "-XX:MetaspaceSize=1G"
    heap_args = -> { "-Xms#{heap} -Xmx#{heap}" }
    mmtk_args = "-XX:+UseThirdPartyHeap -Dprobes=RustMMTk"
    probes = "$PWD/evaluation/probes"
    dacapo_9_12 = "-Djava.library.path=#{probes} -cp #{probes}:#{probes}/probes.jar:/usr/share/benchmarks/dacapo/dacapo-9.12-bach.jar Harness"
    bm_args = "#{dacapo_9_12} -n #{n} -c probe.DacapoBachCallback #{benchmark}"
    jdk = "./mmtk-openjdk/repos/openjdk"
    mmtk = "./mmtk-openjdk/mmtk"
    conf = -> { "linux-x86_64-normal-server-#{profile}" }
    java = -> { "#{jdk}/build/#{conf.()}/jdk/bin/java" }

    task :config do
        🔵 "sh configure --disable-warnings-as-errors --with-debug-level=#{profile} --with-target-bits=64 --disable-zip-debug-info", cwd:jdk
    end

    task :build do
        🔵 "make --no-print-directory CONF=#{conf.()} THIRD_PARTY_HEAP=$PWD/../../openjdk", cwd:jdk
    end

    task :test => :build do
        🔵 "#{java.()} #{vm_args} #{heap_args.()} #{mmtk_args} #{bm_args}"
    end

    task :gdb => :build do
        🔵 "gdb --args #{java.()} #{vm_args} #{heap_args.()} #{mmtk_args} #{bm_args}"
    end
end