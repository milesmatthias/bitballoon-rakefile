#!/usr/bin/env rake

require 'bitballoon'
require 'yaml'

# this is currently experimental. DONT USE

CONFIG_FILE = 'config/bitballoon.yaml'
BUILD_DIR = 'build'

namespace :bitballoon do

  def load_config
    $config = YAML::load_file(CONFIG_FILE)
  end

  def save_config
    puts 'saving config...'
    if $config.nil?
      puts 'config is nil so not doing anything'
    else
      puts $config.inspect
      File.open(CONFIG_FILE, 'w') { |f| f.write $config.to_yaml }
      puts 'config saved.'
    end
  end

  def auth
    load_config
    if $config['account']['access_token'].nil?
      puts 'using client_id and client_secret to authenticate'
      bitballoon = BitBalloon::Client.new(:client_id => $config['account']['client_id'], :client_secret => $config['account']['client_secret'])
      bitballoon.authorize_from_credentials!
      $config['account']['access_token'] = bitballoon.access_token
      save_config
    else
      puts 'using token to authenticate'
      bitballoon = BitBalloon::Client.new(:access_token => $config['account']['access_token'])
    end
    
    bitballoon
  end

  def notify_term(url)
    begin
      require 'terminal-notifier'
      system "terminal-notifier -title 'dojo4' -message 'Deploy completed successfully to #{ url }.' -sound default"
    rescue LoadError
    end
  end
  
  desc "deploy site to bitballoon"
  task :deploy do
    puts 'Deploying site...'

    # authenticate
    bitballoon = auth

    # actually deploy
    begin
      # if site exists, just update
      site = bitballoon.sites.get($config['site']['id'] || $config['site']['url'])
      puts 'the site exists, so simply updating...'
      site.update(:dir => BUILD_DIR)
    rescue
      # if site hasn't been created yet, create it
      puts 'the site does NOT exist, so creating it'
      site = bitballoon.sites.create(:dir => BUILD_DIR)
      $config['site']['id'] = site.id
      $config['site']['url'] = site.url
      save_config
    end

    # wait until the deploy is done...
    site.wait_for_ready

    # notify campfire and end.
    puts '...deploy finished!'
    puts('Site was deployed to: ' + $config['site']['url'])

    # use terminal notifier if it's available
    notify_term($config['site']['url'])
  end

  desc "list all sites on our account"
  task :list do
    bitballoon = auth
    bitballoon.sites.each do |s|
      puts('s.id = ' + s.id + '; s.url = ' + s.url)
    end
  end

  desc "destroy the site"
  task :destroy do
    bitballoon = auth
    begin
      # if site exists, just update
      site = bitballoon.sites.get($config['site']['id'] || $config['site']['url'])
      site.destroy!
    rescue
      puts "no site to destroy. so doing nothing."
    end
  end

  desc "destroy all of your sites DANGEROUS"
  task :destroyall do
    bitballoon = auth
    bitballoon.sites.each do |s|
      s.destroy!
    end
  end


end


