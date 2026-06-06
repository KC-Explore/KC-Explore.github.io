source "https://rubygems.org"

# Modern, pinned toolchain — built by GitHub Actions (.github/workflows/pages.yml),
# NOT the legacy github-pages gemset. Jekyll 4 ships Dart Sass, so no Ruby-Sass quirks.
gem "jekyll", "~> 4.3"

group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.17"
  gem "jekyll-seo-tag", "~> 2.8"
end

# Pulled from Ruby stdlib in recent versions; declare explicitly for CI.
gem "webrick", "~> 1.9"

# Windows / JRuby timezone data (harmless elsewhere).
gem "tzinfo-data", platforms: [:mingw, :mswin, :x64_mingw, :jruby]
