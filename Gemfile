# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll-theme-chirpy", "7.2.4"

gem "html-proofer", "~> 5.0", group: :test

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.2.0", :platforms => [:mingw, :x64_mingw, :mswin]
