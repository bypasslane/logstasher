# Logstasher [![Gem Version](https://badge.fury.io/rb/logstasher.png)](http://badge.fury.io/rb/logstasher) [![Build Status](https://secure.travis-ci.org/shadabahmed/logstasher.png)](https://secure.travis-ci.org/shadabahmed/logstasher)
### Awesome Logging for Rails !!

This gem is heavily inspired from [lograge](https://github.com/roidrage/lograge), but it's focused on one thing and one thing only. That's making your logs awesome like this:

[![Awesome Logs](https://f.cloud.github.com/assets/830679/2407078/dcde03e8-aa82-11e3-85ac-8c5b3a86676e.png)](https://f.cloud.github.com/assets/830679/2407078/dcde03e8-aa82-11e3-85ac-8c5b3a86676e.png)

How it's done ?

By, using these awesome tools:
* [Logstash](http://logstash.net) - Store and index your logs
* [Kibana](http://kibana.org/) - for awesome visualization. This is optional though, and you can use any other visualizer

Update: Logstash now includes Kibana build in, so no need to separately install. Logstasher has been test with **logstash version 1.3.3**

See [quickstart](#quick-setup-for-logstash) for quickly setting up logstash

## About logstasher

This gem purely focuses on how to generate logstash compatible logs i.e. *logstash json event format*,  without any overhead. Infact, logstasher logs to a separate log file named `logstash_<environment>.log`.
The reason for this separation:
 * To have a pure json log file
 * Prevent any logger messages(e.g. info) getting into our pure json logs

Before **logstasher** :

```
Started GET "/login" for 10.109.10.135 at 2013-04-30 08:59:01 -0400
Processing by SessionsController#new as HTML
  Rendered sessions/new.html.haml within layouts/application (4.3ms)
  Rendered shared/_javascript.html.haml (0.6ms)
  Rendered shared/_flashes.html.haml (0.2ms)
  Rendered shared/_header.html.haml (52.9ms)
  Rendered shared/_title.html.haml (0.2ms)
  Rendered shared/_footer.html.haml (0.2ms)
Completed 200 OK in 532ms (Views: 62.4ms | ActiveRecord: 0.0ms | ND API: 0.0ms)
```

After **logstasher**:

```
{"@source":"unknown","@tags":["request"],"@fields":{"method":"GET","path":"/","format":"html","controller":"file_servers"
,"action":"index","status":200,"duration":28.34,"view":25.96,"db":0.88,"ip":"127.0.0.1","route":"file_servers#index",
"parameters":"","ndapi_time":null,"uuid":"e81ecd178ed3b591099f4d489760dfb6","user":"shadab_ahmed@abc.com",
"site":"internal"},"@timestamp":"2013-04-30T13:00:46.354500+00:00"}
```

By default, the older format rails request logs are disabled, though you can enable them.

## Installation

In your Gemfile:

    gem 'logstasher'

### Configure your `<environment>.rb` e.g. `development.rb`

    # Enable the logstasher logs for the current environment
    config.logstasher.enabled = true

    # This line is optional if you do not want to suppress app logs in your <environment>.log
    config.logstasher.suppress_app_log = false

    # This line is optional, it allows you to set a custom value for the @source field of the log event
    config.logstasher.source = 'your.arbitrary.source'

## Logging params hash

Logstasher can be configured to log the contents of the params hash.  When enabled, the contents of the params hash (minus the ActionController internal params)
will be added to the log as a deep hash.  This can cause conflicts within the Elasticsearch mappings though, so should be enabled with care.  Conflicts will occur
if different actions (or even different applications logging to the same Elasticsearch cluster) use the same params key, but with a different data type (e.g. a
string vs. a hash).  This can lead to lost log entries.  Enabling this can also significantly increase the size of the Elasticsearch indexes.

To enable this, add the following to your `<environment>.rb`

    # Enable logging of controller params
    config.logstasher.log_controller_parameters = true

## Adding custom fields to the log

Since some fields are very specific to your application for e.g. *user_name*, so it is left upto you, to add them. Here's how to add those fields to the logs:

    # Create a file - config/initializers/logstasher.rb

    if LogStasher.enabled
      LogStasher.add_custom_fields do |fields|
        # This block is run in application_controller context,
        # so you have access to all controller methods
        fields[:user] = current_user && current_user.mail
        fields[:site] = request.path =~ /^\/api/ ? 'api' : 'user'

        # If you are using custom instrumentation, just add it to logstasher custom fields
        LogStasher.custom_fields << :myapi_runtime
      end
    end

## Ignore Actions
This was taken from Lograge.  It add the ability to ignore certain actions.
To further clean up your logging, you can also tell Logstasher to skip log messages 
meeting given criteria.  You can skip log messages generated from certain controller
actions, or you can write a custom handler to skip messages based on data in the log event:

```
# config/environments/production.rb
MyApp::Application.configure do
  config.lograge.enabled = true

  config.lograge.ignore_actions = ['home#index', 'aController#anAction']
  config.lograge.ignore_custom = lambda do |event|
    # return true here if you want to ignore based on the event
  end
end
```
## Quick Setup for Logstash

* Download logstash from [logstash.net](http://www.logstash.net/)
* Use this sample config file: [quickstart.conf](https://github.com/shadabahmed/logstasher/raw/master/sample_logstash_configurations/quickstart.conf) 
* Start logstash with the following command:
```
java -jar logstash-1.3.3-flatjar.jar agent -f quickstart.conf -- web
```
* Visit http://localhost:9292/ to see the Kibana interface and your parsed logs
* For advanced options see the latest logstash documentation at [logstash.net](http://www.logstash.net/) or visit my blog at (shadabahmed.com)[http://shadabahmed.com/blog/2013/04/30/logstasher-for-awesome-rails-logging] (slightly outdated but will sure give you ideas for distributed setup etc.)

## Versions
All versions require Rails 3.0.x and higher and Ruby 1.9.2+. Tested on Rails 4 and Ruby 2.0

## Development
 - Run tests - `rake`
 - Generate test coverage report - `rake coverage`. Coverage report path - coverage/index.html

## Copyright

Copyright (c) 2013 Shadab Ahmed, released under the MIT license
