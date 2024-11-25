desc 'Creates a snap of the application for deployment'
task :snap do
  env = {
    'BUNDLE_CACHE_ALL'           => '1',
    'BUNDLE_CACHE_ALL_PLATFORMS' => '1',
    'BUNDLE_NO_INSTALL'          => '1',
  }

  sh  'rm -rf vendor/cache'
  sh env, 'bundle check'
  sh env, 'bundle package'

  # sh "tar --sort=name --owner=root:0 --group=root:0 --mtime='UTC 2022-12-19' -c #{<<~FILES.split.join(' ')} | gzip -n > app_new.tar.gz"
  #   bin
  # FILES

  # sh 'diff -q app.tar.gz app_new.tar.gz' do |ok, _res|
  #   if ok
  #     puts "App and configs haven't changed. Leaving app.tar.gz as is."
  #     sh 'rm -f app_new.tar.gz'
  #   else
  #     puts "App and configs have changed. Updating app.tar.gz ..."
  #     sh 'mv app_new.tar.gz app.tar.gz'
  #   end
  # end

  sh 'tar --sort=name --owner=root:0 --group=root:0 --mtime="UTC 2022-12-19" -c Gemfile Gemfile.lock vendor/cache | gzip -n > bundle_new.tar.gz'

  sh 'diff -q bundle.tar.gz bundle_new.tar.gz' do |ok, _res|
    if ok
      puts "Bundle hasn't changed. Leaving bundle.tar.gz as is."
      sh 'rm -f bundle_new.tar.gz'
    else
      puts "Bundle has changed. Updating bundle.tar.gz ..."
      sh 'mv bundle_new.tar.gz bundle.tar.gz'
    end
  end

  sh 'snapcraft --use-lxd --verbose --debug'
end
