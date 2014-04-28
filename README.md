# Infrataster

[![Gem Version](https://badge.fury.io/rb/infrataster.png)](http://badge.fury.io/rb/infrataster)

Infrastructure Behavior Testing Framework.

## Usage with Vagrant

First, create `Gemfile`:

```ruby
source 'https://rubygems.org'

gem 'infrataster'
```

Install gems:

```
$ bundle install
```

Install Vagrant: [Official Docs](http://docs.vagrantup.com/v2/installation/index.html)

Create Vagrantfile:

```ruby
# Vagrantfile
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "hashicorp/precise64"

  config.vm.define :proxy do |c|
    c.vm.network "private_network", ip: "192.168.33.10"
    c.vm.network "private_network", ip: "171.16.33.10", virtualbox__intnet: "infrataster-example"
  end

  config.vm.define :app do |c|
    c.vm.network "private_network", ip: "171.16.33.11", virtualbox__intnet: "infrataster-example"
  end
end
```

Start VMs:

```
$ vagrant up
```

Initialize rspec directory:

```
$ rspec --init
  create   spec/spec_helper.rb
  create   .rspec
```

`require 'infrataster/rspec'` and define target servers for testing in `spec/spec_helper.rb`:

```ruby
# spec/spec_helper.rb
require 'infrataster/rspec'

Infrataster::Server.define(
  :proxy,          # name
  '192.168.33.10', # ip address
  vagrant: true    # for vagrant VM
)
Infrataster::Server.define(
  :app,            # name
  '172.16.33.11',  # ip address
  vagrant: true,   # for vagrant VM
  from: :proxy     # access to this machine via SSH port forwarding from proxy
)

# Code generated by `rspec --init` is following...
```

If you use `capybara`, you should download and extract [BrowserMob Proxy](http://bmp.lightbody.net/) and set `Infrataster::BrowsermobProxy.bin_path` to binary path in `spec/spec_helper.rb`:

```ruby
# spec/spec_helper.rb
Infrataster::BrowsermobProxy.bin_path = '/path/to/browsermob/bin/browsermob'
```

Then, you can write spec files:

```ruby
# spec/example_spec.rb
require 'spec_helper'

describe server(:app) do
  describe http('http://app') do
    it "responds content including 'Hello Sinatra'" do
      expect(response.body).to include('Hello Sinatra')
    end
    it "responds as 'text/html'" do
      expect(response.header.content_type).to eq('text/html')
    end
  end
end
```

Run tests:

```
$ bundle exec rspec
2 examples, 2 failures
```

Currently, the tests failed because the VM doesn't respond to HTTP request.

It's time to write provisioning instruction like Chef's cookbooks or Puppet's manifests!

## Resources

"Resource" is what you test by Infrataster. For instance, the following code describes `http` resource.

```ruby
describe server(:app) do
  describe http('http://example.com') do
    it "responds content including 'Hello Sinatra'" do
      expect(response.body).to include('Hello Sinatra')
    end
  end
end
```

### `http` resource

### `capybara` resource

### `mysql` resource

## Example

* [example](example)
* [spec/integration](spec/integration)

## Tests

### Unit Tests

Unit tests are under `spec/unit` directory.

```
$ bundle exec rake spec:unit
```

### Integration Tests

Integration tests are under `spec/integration` directory.

```
$ bundle exec rake spec:integration:prepare
$ bundle exec rake spec:integration
```

## Presentations

* https://speakerdeck.com/ryotarai/infrataster-infra-behavior-testing-framework-number-oedo04

## Changelog

[Changelog](CHANGELOG.md)

## Contributing

1. Fork it ( http://github.com/ryotarai/infrataster/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
