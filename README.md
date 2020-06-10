# Rails Travis CI Example

An example Ruby on Rails application configured to run Travis CI.

[![Build Status](https://travis-ci.org/stevepolitodesign/rails-travis-ci-example.svg?branch=master)](https://travis-ci.org/stevepolitodesign/rails-travis-ci-example)

## Configure Rails Application to run System Tests in Travis CI

Rails is configured by default to run [system tests](https://guides.rubyonrails.org/testing.html#system-testing) in Google Chrome. However, I ran into an issue with Travis CI when it came to running system tests using the default configuration. My solution was to update `test/application_system_test_case.rb` by declearing `:headless_chrome` instead of the default `:chrome` setting.

```ruby
# test/application_system_test_case.rb
require "test_helper"

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :headless_chrome, screen_size: [1400, 1400]
end
```

## Configure Travis CI

```
# .travis.yml
language: ruby
cache:
  - bundler
  - yarn
services:
  - postgresql
before_install:
  - nvm install --lts
before_script:
  - bundle install --jobs=3 --retry=3
  - yarn
  - bundle exec rake db:create
  - bundle exec rake db:schema:load
script:
  - bundle exec rake test
  - bundle exec rake test:system
```

|Key|Description|
|-------|-----------|
|[os](https://config.travis-ci.com/ref/os)|Sets the build's operating system. Note that we did **not** add an `os` key, and are using the [default environment](https://docs.travis-ci.com/user/reference/xenial/)|
|[language](https://config.travis-ci.com/ref/language)|Selects the language support used for the build. We select `ruby` since this is a Rails project|
|[cache](https://config.travis-ci.com/ref/job/cache)|Activates caching content that does not often change in order to speed up the build process. We add `bundler` and `yarn` since Rails uses [bundler](https://bundler.io/) and [yarn](https://yarnpkg.com/) to managage dependencies.|
|[services](https://config.travis-ci.com/ref/job/services)|Services to set up and start. We add `postgresql` since our database is postgresql. You could also add `redis`.|
|[before_install](https://config.travis-ci.com/)|Scripts to run before the install stage. We add `nvm install --lts` to use the latest stable version of Node. This will be needed when we run `yarn` later.|
|[before_script](https://config.travis-ci.com/)|Scripts to run before the script stage. This sets up our Rails application. Note that I do not seed the database, since we only care about the test environment. I run `bundle install --jobs=3 --retry=3` instead of `bundle` becuase that's what the [documentation](https://docs.travis-ci.com/user/languages/ruby#bundler) recommends.|
|[script](https://config.travis-ci.com/)|Scripts to run at the script stage. In our case, we just run our tests.|
