rails s 做了啥？

```ruby
rails 在那？
/bin/rails
#!/usr/bin/env ruby.exe
APP_PATH = File.expand_path('../config/application', __dir__)
require_relative "../config/boot"
require "rails/commands"



# 项目目录 E:\rails\test-demo\config\boot.rb
        ENV['BUNDLE_GEMFILE'] ||= File.expand_path('../Gemfile', __dir__)
        require "bundler/setup" # Set up gems listed in the Gemfile.
        require "bootsnap/setup" # Speed up boot time by caching expensive operations.

# require "bundler/setup" # Set up gems listed in the Gemfile.
E:\rails\test-demo\vendor\gems\bundler-master\lib\bundler\setup.rb
        require_relative "shared_helpers"

        if Bundler::SharedHelpers.in_bundle?
          require_relative "../bundler"

          if STDOUT.tty? || ENV["BUNDLER_FORCE_TTY"]
            begin
              Bundler.ui.silence { Bundler.setup }
            rescue Bundler::BundlerError => e
              Bundler.ui.warn "\e[31m#{e.message}\e[0m"
              Bundler.ui.warn e.backtrace.join("\n") if ENV["DEBUG"]
              if e.is_a?(Bundler::GemNotFound)
                Bundler.ui.warn "\e[33mRun `bundle install` to install missing gems.\e[0m"
              end
              exit e.status_code
            end
          else
            Bundler.ui.silence { Bundler.setup }
          end

          # We might be in the middle of shelling out to rubygems
          # (RUBYOPT=-rbundler/setup), so we need to give rubygems the opportunity of
          # not being silent.
          Gem::DefaultUserInteraction.ui = nil
        end

# require "bootsnap/setup"
		require_relative('../bootsnap')
		Bootsnap.default_setup
# E:\rails\test-demo\vendor\gems\bootsnap-master\lib\bootsnap.rb
		  def self.default_setup
            env = ENV['RAILS_ENV'] || ENV['RACK_ENV'] || ENV['ENV']
            development_mode = ['', nil, 'development'].include?(env)

            unless ENV['DISABLE_BOOTSNAP']
              cache_dir = ENV['BOOTSNAP_CACHE_DIR']
              unless cache_dir
                config_dir_frame = caller.detect do |line|
                  line.include?('/config/')
                end

                unless config_dir_frame
                  $stderr.puts("[bootsnap/setup] couldn't infer cache directory! Either:")
                  $stderr.puts("[bootsnap/setup]   1. require bootsnap/setup from your application's config directory; or")
                  $stderr.puts("[bootsnap/setup]   2. Define the environment variable BOOTSNAP_CACHE_DIR")

                  raise("couldn't infer bootsnap cache directory")
                end

                path = config_dir_frame.split(/:\d+:/).first
                path = File.dirname(path) until File.basename(path) == 'config'
                app_root = File.dirname(path)

                cache_dir = File.join(app_root, 'tmp', 'cache')
              end


              setup(
                cache_dir:            cache_dir,
                development_mode:     development_mode,
                load_path_cache:      !ENV['DISABLE_BOOTSNAP_LOAD_PATH_CACHE'],
                compile_cache_iseq:   !ENV['DISABLE_BOOTSNAP_COMPILE_CACHE'] && iseq_cache_supported?,
                compile_cache_yaml:   !ENV['DISABLE_BOOTSNAP_COMPILE_CACHE'],
              )

              if ENV['BOOTSNAP_LOG']
                log!
              end
            end
          end


```

