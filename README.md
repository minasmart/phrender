# Phrender

A rack app and api to render javascript based websites using PhantomJS.

## Installation

Add this line to your application's Gemfile:

    gem 'phrender'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install phrender

Then install PhantomJS:

    $ brew install phantomjs

## Usage

*NOTE*: Ember users, there is a driver for ember. See the [Ember Driver](#ember-driver)
section.

### For completely static javascript sites:

In your rackup file:

```ruby
require 'phrender'

# The asset root should reflect where your app is stored, as well as all images,
# fonts, css, and other static files.
asset_root = "./"

# The index file should link to no javascript on its own, or include any script
# tags.
index_file = 'phrender.html'

# This is the main application. It may or may not also start up the app.
javascript_files = [ '/assets/application.js' ]

# This is just raw javascript. Typically the app boot code.
javascript = [ "MyApp.boot()" ]

# Javascript files and raw javascript get concatenated. Files first, then raw
# code in array order. This code is then run against the index file using
# PhantomJS

# Rackup!
run Phrender::RackStatic.new({
  :asset_root => asset_root,
  :index_file => index_file,
  :javascript_files => javascript_files,
  :javascript => javascript
})
# Additionally, you map pass the option :timeout, which gets passed on to
# PhantomJS.
```

### For partially dynamic javascript sites:

Use this method if some other rack-based application is serving up your assets,
or your index.html file. This can be useful for development, and even in some
production cases.

In your rackup file:

```ruby
require 'phrender'
require 'my_rack_app'

# These options are the same as above, but instead of referencing actual files,
# the paths are the request paths to send to the upstream application server.
index_file = 'phrender.html'
javascript_files = [ '/assets/application.js' ]
javascript = [ "MyApp.boot()" ]

use Phrender::RackMiddleware, {
  :index_file => index_file,
  :javascript_files => javascript_files,
  :javascript => javascript
}
# Additional options are timeout and ssl, if, for some reason, your rack app is
# serving up SSL encrypted pages. Run `phantomjs --help` to see available
# options, or use something falsey to disable. Both options are passed to
# phantom.

# Include other middlewarn
use Rack::Static, :urls => "/assets", :root => "./assets"

# Rackup!
run MyRackApp.new
```

## Signalling Render Completion

Phrender will, after `timeout` milliseconds serve up whatever has been rendered.
However, performance is better if you signal to phrender that rendering is
complete, and content should be served up. To do this, your application needs to
log to the console:

```
console.log('-- PHRENDER COMPLETE --');
```

If you are using ember, there is already a mechanism to do this, using the ember
driver.

## Ember Driver

There is a driver provided for ember users that overrides `Ember.Route` to log
out to phrender, without harming your application. The driver is stored in
`lib/phrender/support/ember_driver.js`. This is how you include it:

```
phrender.add_javascript_file '/assets/application.js'
phrender.add_javascript_file :ember_driver
phrender.add_javascript "MyApp.boot()"
```

It's important to include the driver *after* ember and *before* you boot your
application.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
