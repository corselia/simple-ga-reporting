[![Gem Version](https://badge.fury.io/rb/simple_ga_reporting.svg)](https://badge.fury.io/rb/simple_ga_reporting) [![CircleCI](https://circleci.com/gh/nikukyugamer/simple-ga-reporting.svg?style=svg)](https://circleci.com/gh/nikukyugamer/simple-ga-reporting)

# Japanese Document
- Japanese Document (my blog's article) is following
    - [Google Analytics の API を用いて簡単にデータを取得できる gem を作りました](https://obel.hatenablog.jp/entry/20180409/1523282148)

# What's this?
This gem helps you to obtain `Google Analytics` reporting data with ease.

First of all, you create configuration YAML file as Google Analytics API. Then execute the simple Ruby code, and you obtain Google Analytics reporting data.

Yeah, very simple :)

## Wrapper gem of Legato
This gem is a wrapper of [Legato gem](https://github.com/tpitale/legato). `Legato` is very coooooooool and full-stack gem, so complicated as a beginner (me!).

Therefore I developed this wrapper gem, `simple_ga_reporting`.

# Get Started

## 1. Installation
```bash
$ gem install simple_ga_reporting
```

## 2. Set `private_key` and `client_email`
- There are two ways to set `private_key` and `client_email`
- How to get above two Key-Value sets is written later

### 2-1. The first way: Create API configuration file
- Create `./config/key_and_email.yml`
- This YAML file must contain two Key-Value sets
    - `private_key` and its value
    - `client_email` and its value

### 2-2. The second way: Set values as arguments
- Set arguments of `SimpleGaReports.filtered_results` method
- For example:
    - `SimpleGaReports.filtered_results(private_key: 'FOO', client_email: 'BAR')`
- Convenient when you set by environment variables

## 3. Create Google Analytics reporting configuration file
- You must create the YAML file which contains Google Analytics reporting configuration such as `start-date`, `end-date`, `metrics`, `demensions`, `filters` and etc.
- filepath is `./config/ga_reporting_config.yml`
- This YAML file format is shown later

## 4. Create filters file (Optional)
- If you wanna use filters function, please create `./config/filters.rb`
- the filters configuration on GUI is below

![filter configuration on GUI](images/config_filter_by_gui.png)

## 5. Write your app
- Write your application used this gem
- The sample code is below
    - Use only two class methods
        - `SimpleGaReports.configure`
        - `SimpleGaReports.filtered_results`
    - Only three lines :-)
    - Note
        - If you don't use filters, use `SimpleGaReports.raw_results` too
        - when no filter is in configure file, `SimpleGaReports.raw_results` is the same as `SimpleGaReports.filtered_results`

```ruby
require 'simple_ga_reporting'

SimpleGaReports.configure
SimpleGaReports.filtered_results #=> Legato::Query
```

# Detail of configuration

## 1. Configuration of API account
- You must set Google Analytics API configuration, `private_key` and `client_email`
- There are two ways to set it
    - The one is by arguments
        - Set arguments of `SimpleGaReports.filtered_results` method
    - The other is by YAML file
        - Default filepath is `./config/key_and_email.yml`
        - You can change filepath by argument
- The sample of YAML file is below

```yaml
private_key: "-----BEGIN PRIVATE KEY-----\nMIIE..........Eqw==\n-----END PRIVATE KEY-----\n"
client_email: "your_account_name@foobar.iam.gserviceaccount.com"
```

## 2. Google Analytics reporting configuration
- You must create the YAML file which contains Google Analytics reporting configuration
- Default filepath is `./config/ga_reporting_config.yml`
    - You can change filepath by argument in `SimpleGaReports.configure` method
- The sample of YAML file is below

```yaml
profile_name: my_profile_name
start_date: 2018-04-01
end_date: 2018-04-05
metrics:
  - users
  - pageviews
dimensions:
  - pagePath
  - pageTitle
sort:
  - -pageviews
  - -users
filters:
  - my_awesome_filter
  - his_awesome_filter
  - her_awesome_filter
limit: 20
```

- Please refer the official documents as Key-Value sets
    - https://developers.google.com/analytics/devguides/reporting/core/v3/reference

![ga_query_parametes_summary.png](images/ga_query_parametes_summary.png)

- Though above official documents show many Key-Value sets, you can config only several sets as below
    - start-date
    - end-date
    - metrics
    - dimensions
    - sort
    - filters
    - max-results
    - samplingLevel
- Some Key names in configure file are different from ones in official document
    - `start_date` in configure file (not `start-date`)
    - `end_date` in configure file (not `end-date`)
    - `limit` in configure file (instead of `max-results`)
    - `sampling_level` in configure file (not `samplingLevel`)
- The type of `metrics`, `dimensions`, `sort` and `filters` is `Array`
    - The order of elements doesn't matter

### 2-1. Key-Value detail

#### profile_name
- REQUIRED
- It was called `view (view name)` before
    - but in Japanese page, still `ビュー名`

![View or Profile](images/ga_profile(view).png)

#### start_date
- REQUIRED
- Original Key name is `start-date`
- Value examples
    - `2018-04-01`
    - `today` or `yesterday`
    - `14daysAgo`
        - use `daysAgo` phrase even if `1day`

#### end_date
- REQUIRED
- Original Key name is `end-date`
- Value examples are the same as `start-date`

#### metrics
- REQUIRED
- `指標` in Japanese
- Official documents are below (very useful!)
    - [Dimensions & Metrics Explorer](https://developers.google.com/analytics/devguides/reporting/core/dimsmets)
- Value examples
    - `users`
    - `uniquePageviews`

![Dimensions & Metrics Explorer](images/dimensions_and_metrics_explorer.png)

#### dimensions
- optional
- Official documents are below (very useful!)
    - [Dimensions & Metrics Explorer](https://developers.google.com/analytics/devguides/reporting/core/dimsmets)
- Value examples
    - `pagePath`
    - `sessionCount`

#### sort
- optional
- prefix
    - `-` sign means descending
    - no sign means ascending
        - Don't use `+` sign
- Value examples
  - `-pageviews`
  - `users`

#### filters
- optional
- Value is method name you named in `filters.rb`

#### limit
- optional
- Original Key name is `max-results`
- Value is max amount of returned rows
- Default value is `100`

#### sampling_level
- optional
- Original Key name is `samplingLevel`
- Default value is `HIGHER_PRECISION`

## 3. filters file
- You use filters function using `filters.rb` file
    - the filename `filters.rb` is determined
        - only filepath is optional
    - the module name `Filters` is determined
    - Default filepath is `./config/filters.rb`
        - You can change it by argument with `configure` method
- Examples

```ruby
module Filters
  def chrome_or_fx
    filter :chrome_or_fx, &lambda { contains(:browser, 'Chrome|Firefox') }
  end

  def awesome_page
    filter :awesome_page, &lambda { contains(:pagePath, '\A.*awesome.*\z') }
  end
end
```

- `chrome_or_fx` method filters the results
    - pick up data which contains 'Google Chrome' or 'Firefox' as browser
- the convention of filter method is below
    - define method name
    - define filter name by symbol which is the same as method name
    - define filter behavior by block object
- Usage of multiple filters
    - If you want to apply AND-chain, only specify each filter in Google Analytics reporting configuration file
    - If you want to apply OR-chain, use regexp in block object as `chrome_or_fx` method
- Adding refer [Legato's Documents](https://github.com/tpitale/legato#google-analytics-supported-filtering-methods)

# Arguments of method

## Note
**YOU MUST SPECIFY ARGUMENTS AS RELATIVE FILEPATH SO YOU MUST ADD './' (dot slash) AT BEGINNING**

- Set arguments of `SimpleGaReports.filtered_results` method
- For example:
    - `SimpleGaReports.filtered_results(private_key: 'FOO', client_email: 'BAR')
- Convenient when you set by environment variables

## `SimpleGaReports.filtered_results` method
- If you don't use filters, use `SimpleGaReports.raw_results` method instead of `SimpleGaReports.filtered_results` method
    - when no filter is in configure file, `SimpleGaReports.raw_results` is the same as `SimpleGaReports.filtered_results`
- take three arguments
    - All arguments are keyword ones
    - You must set (`key_and_email_file`) or (`private_key` and `client_email`)

1. `key_and_email_file`
    - default is `./config/key_and_email.yml`

1. `private_key`
    - default is `nil`

1. `client_email`
    - default is `nil`

## `SimpleGaReports.configure` method
- take three arguments
    - All arguments are keyword ones

1. `report_config`
    - first argument
    - YAML file
    - default is `./config/ga_reporting_config.yml`

2. `filters`
    - second argument
    - filter file
    - default is `./config/filters.rb`

3. `**options`
    - third argument
    - default is nil
    - if you set this argument, you can overwrite values in Google Analytics reporting configuration file
    - example
        - `start_date: '90daysAgo', end_date: '60daysAgo', limit: 100`

# How to get `private_key` and `client_email`

#### 1. Access to [Google Cloud Platform Console](https://console.cloud.google.com/)

#### 2. Create new project
- name new project as you like

#### 3. Create service account for Google Analytics API and select `API and Service` from menu

![Google Cloud Platform API menu](images/gcp_api_menu.png)

#### 4. Create authentication information
- Select `Service Account Key`

![Google Cloud Platform API Auth Info](images/gcp_api_auth_info.png)

#### 5. Select Service Account and download JSON Key

![Google Cloud Platform JSON Key](images/gcp_download_json_key.png)

#### 6. Open JSON Key file by editor and pick up `"private_key"` and `"client_email"`

![Google Cloud Platform private_key and client_email](images/gcp_private_key_and_client_email.png)

#### 7. They are the very Key-Value sets you want
- Deal with them carefully

#### 8. They are the very Key-Value sets you want
- Deal with them carefully

#### 9. Activate Analytics API
- NOT `Google Analytics Reporting API` BUT `Analytics API`

![GCP Activate API 01](images/gcp_activate_api_01.png)

![GCP Activate API 02](images/gcp_activate_api_02.png)

![GCP Activate API 03](images/gcp_activate_api_03.png)

#### 10. Add API user to Google Analytics
- In Google Analytics, add mail address of API user
- Only `Display and Analytics` authority needs

![ga_add_api_user.png](images/ga_add_api_user.png)

# Sample App

## Note
**YOU MUST SPECIFY ARGUMENTS AS RELATIVE FILEPATH SO YOU MUST ADD './' (dot slash) AT BEGINNING**

## Prepare files

1. `./my_sample_app.rb`
1. `./foo/my_key_and_email.yml`
1. `./bar/my_ga_reporting_config.yml`
1. `./foobar/filters.rb`

#### 1. `./my_sample_app.rb`
```ruby
require 'simple_ga_reporting'

SimpleGaReports.configure(report_config: './bar/my_ga_reporting_config.yml', filters: './foobar/filters.rb', start_date: '2daysAgo', limit: 100)
results = SimpleGaReports.filtered_results(key_and_email: './foo/my_key_and_email.yml') # or use arguments of 'private_key:' and 'client_email:'

results.each do |result|
  puts '==================================='
  puts result['pagePath']
  puts result['pageTitle']
  puts result['pageviews']
  puts result['users']
end
```

#### 2. `./foo/my_key_and_email.yml`
```yaml
private_key: "-----BEGIN PRIVATE KEY-----\nMIIE..........Eqw==\n-----END PRIVATE KEY-----\n"
client_email: "your_account_name@foobar.iam.gserviceaccount.com"
```

- You can set two key and value by not only YAML file but also arguments of `SimpleGaReports.filtered_results` method

#### 3. `./bar/my_ga_reporting_config.yml`
```yaml
profile_name: my_profile_name
start_date: 2018-04-01
end_date: 2018-04-05
metrics:
  - users
  - pageviews
dimensions:
  - pagePath
  - pageTitle
sort:
  - -pageviews
  - -users
filters:
  - chrome_or_fx
  - happy_page
limit: 20
```

#### 4. `./foobar/filters.rb`
```ruby
module Filters
  def chrome_or_fx
    filter :chrome_or_fx, &lambda { contains(:browser, 'Chrome|Firefox') }
  end

  def happy_page
    filter :happy_page, &lambda { contains(:pagePath, '\A.*happy.*\z') }
  end
end
```

## execute script
```bash
$ ruby ./my_sample_app.rb
===================================
/i_am_happy.html
I am HAPPY!
10000
1000
===================================
/happy/index.html
HAPPY TOP
9000
900
===================================
/path/to/happy.html
Are You Happy?
8000
800
===================================
/happy.php
HAPPY CHECK!
7000
700
===================================
......
```

# Rails
When you use this gem on Rails, it's simple :)

1. add this gem to `Gemfile`
2. `$ bundle install`
3. after it, the same way above

# Very Awesome Reference Book (Recommended)
- [わかばちゃんと学ぶ Googleアナリティクス (Let's study Google Analytics with Wakaba-Chan)](http://www.c-r.com/book/detail/1217)

![わかばちゃんと学ぶ Googleアナリティクス](http://www.c-r.com/book/images/x/86354-232-7_x.jpg)

- Created by [Ai Minatogawa](https://llminatoll.github.io/)
    - She is an energetic creator!
- Japanese version only
    - English version will...?

# Official documents
## Core Reporting API - Reference Guide
- https://developers.google.com/analytics/devguides/reporting/core/v3/reference

## Dimensions & Metrics Explorer
- https://developers.google.com/analytics/devguides/reporting/core/dimsmets

## Query Explorer
- https://ga-dev-tools.appspot.com/query-explorer/

## API Rate Limits
- https://developers.google.com/analytics/devguides/reporting/core/v3/limits-quotas

# TODO
- can input not only `profile_name` but also `profile id (view id)`
- implement following query parametes
    - offset
    - quota_user
    - segment_id
        - ~~`segment` is the same as conbination of filters, so you can use 'filters' as alternative way~~
        - `segment` is the first filter for initial population and it can't substitute

# Development
After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

# Contributing
Bug reports and pull requests are welcome on GitHub at https://github.com/corselia/simple-ga-reporting. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

# License
The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

# Code of Conduct
Everyone interacting in the SimpleGaReporting project’s codebases, issue trackers, chat rooms and mailing lists is
expected to follow the [code of conduct](https://github.com/corselia/simple-ga-reporting/blob/master/CODE_OF_CONDUCT.md).
