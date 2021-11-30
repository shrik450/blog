source "https://rubygems.org"

git_source :github do |repo| 
  "https://github.com/#{repo}.git"
end

gem "jekyll"

group :jekyll_plugins do
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-feed", "~> 0.12"
  gem "jekyll-org", github: "eggcaker/jekyll-org"
  gem "pygments.rb"
  gem "jekyll-postfiles"
end

# Required for Ruby 3+
gem "webrick"
