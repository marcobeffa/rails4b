source 'https://rubygems.org'

gem "jekyll", "~> 4.3.2" # installed by `gem jekyll`
# gem "webrick"        # required when using Ruby >= 3 and Jekyll <= 4.2.2

group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
  gem "jekyll-default-layout"
  gem "jekyll-github-metadata", ">= 2.15"

  gem "jekyll-include-cache"


end

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]

# Lock `http_parser.rb` gem to `v0.6.x` on JRuby builds since newer versions of the gem
# do not have a Java counterpart.
gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]

gem "just-the-docs"

gem "html-proofer", "~> 5.0", :group => :development
gem "faraday-retry"

gem 'jekyll-sitemap'