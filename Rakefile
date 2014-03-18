require 'rake/packagetask'
require 'octokit'

GIT_REPO = "MattesGroeger/vim-bookmarks"

task :default => [:release]

desc "Create release archive"
task :release do
  version = request_user_input("Which version do you want to release (0.1.0)?")
  file_path = create_zip(version)
  if request_user_input("Create git release now? (y/n)") == 'y'
    upload_release(version, file_path)
  end
end

def create_zip(version)
  file_path = "release/vim-bookmarks-#{version}.zip"
  `mkdir -p release`
  `zip -r #{file_path} . -i "doc/*" -i "plugin/*" -i LICENSE.txt`
  file_path
end

def upload_release(version, asset_path)
  asset_name = asset_path.split("/")[-1]

  # login to github
  client = Octokit::Client.new(:netrc => true)
  client.login

  # get all milestones
  milestones = client.milestones(GIT_REPO)
  milestone = milestones.select { |m|
    m.title == version
  }.first
  return puts "Unable to find milestone for version #{version}. Aborted!" if !milestone

  # abort if there are still open issues
  return puts "Found #{milestone.open_issues} open issues for milestone #{version}. Close them first!" if milestone.open_issues > 0

  # get changes via issues
  issues = client.issues(GIT_REPO, :milestone => milestone.number, :state => :closed)
  changes = issues.map { |i|
    labels = i.labels.map { |l| "[#{l.name}]" }.join(", ")
    " * #{labels} #{i.title}\n"
  }

  # compile changelist of issues, get it confirmed by user (y/n)
  puts "> Changelog:\n #{changes.join}\n"
  return puts "Aborted!" if request_user_input("Do you want to create release #{version} with the above changelog? (y/n)", "n").downcase != "y"

  # create release
  release = client.create_release(GIT_REPO, version, :name => "vim-bookmarks-#{version}", :body => changes.join)
  puts "> Created release #{release.name} (id: #{release.id})\n\n"

  # upload asset
  asset = client.upload_asset(release.url, asset_path, :content_type => 'application/zip', :name => asset_name)
  puts "> Asset #{asset.name} uploaded (id: #{asset.id})\n\n"

  # close milestone
  client.update_milestone(GIT_REPO, milestone.number, :state => :closed)
  puts "> Closed milestone #{version}. Done!"
end

def request_user_input(message, fallback = "")
  STDOUT.puts message
  input = STDIN.gets.strip.to_s
  if input.empty?
    if fallback.empty?
      request_user_input(message) # try again
    else
      fallback.to_s
    end
  else
    input.to_s
  end
end
