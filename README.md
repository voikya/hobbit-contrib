# Hobbit::Contrib [![Build Status](https://travis-ci.org/patriciomacadden/hobbit-contrib.png?branch=master)](https://travis-ci.org/patriciomacadden/hobbit-contrib) [![Code Climate](https://codeclimate.com/github/patriciomacadden/hobbit-contrib.png)](https://codeclimate.com/github/patriciomacadden/hobbit-contrib) [![Coverage Status](https://coveralls.io/repos/patriciomacadden/hobbit-contrib/badge.png?branch=master)](https://coveralls.io/r/patriciomacadden/hobbit-contrib) [![Dependency Status](https://gemnasium.com/patriciomacadden/hobbit-contrib.png)](https://gemnasium.com/patriciomacadden/hobbit-contrib) [![Gem Version](https://badge.fury.io/rb/hobbit-contrib.png)](http://badge.fury.io/rb/hobbit-contrib)

Contributed Hobbit extensions.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'hobbit-contrib', require: 'hobbit/contrib'
# or this if you want to use master
# gem 'hobbit-contrib', github: 'patriciomacadden/hobbit-contrib', require: 'hobbit/contrib'
```

And then execute:

```bash
$ bundle
```

Or install it yourself as:

```bash
$ gem install hobbit-contrib
```

## Usage

Each extension may have its own usage. In general, including the module will be
enough.

```ruby
require 'hobbit'
require 'hobbit/contrib'

class App < Hobbit::Base
  # include hobbit session extension
  include Hobbit::Session

  # define your application
end
```

## Available extensions

### Hobbit::AssetTag

This module allows you to include images, javascripts and stylesheets in a easy
way. To use this extension just include the module:

In `config.ru`:

```ruby
require 'hobbit'
require 'hobbit/contrib'

class App < Hobbit::Base
  # Put your assets in:
  # * public/images
  # * public/javascripts
  # * public/stylesheets
  use Rack::Static, root: 'public', urls: ['/images', '/javascripts', '/stylesheets']
  include Hobbit::AssetTag
  include Hobbit::EnhancedRender # see below

  get '/' do
    render 'index', {}, layout: 'layout'
  end
end

run App.new
```

in `views/layouts/layout.erb`:

```ruby
<!DOCTYPE html>
<html>
  <head>
    <title>Hobbit::AssetTag</title>
    <!--
    becomes:
      <script src="http://code.jquery.com/jquery-2.0.0.min.js" type="text/javascript"></script>
      <script src="/javascripts/application.js" type="text/javascript"></script>
    -->
    <%= javascript 'http://code.jquery.com/jquery-2.0.0.min.js', 'application' %>
    <!--
    becomes:
      <link href="/stylesheets/application.css" rel="stylesheet"/>
    -->
    <%= stylesheet 'application' %>
  </head>
  <body>
    <%= yield %>
  </body>
</html>
```

and in `views/index.erb`:

```ruby
<h1>Hobbit::AssetTag</h1>
<!-- becomes: /images/some-hobbit.png -->
<img src="<%= image_path 'some-hobbit.png' %>"/>
```

#### Available methods

* `image_path`: Returns the path for a given image.
* `javascript`: Returns a list of one or more script tags (see the example
above).
* `javascript_path`: Returns the path for a given javascript file. If you pass
an url, it returns the given url. If you pass an arbitrary string, it returns
`/javascripts/#{url}.js`.
* `stylesheet`: Returns a list of one or more link tags (see the example
above).
* `stylesheet_path`: Returns the path for a given stylesheet file. If you pass
an url, it returns the given url. If you pass an arbitrary string, it returns
`/stylesheets/#{url}.css`.

### Hobbit::Environment

This extension allows you to control the application environment by using the
provided methods. To use this extension just include the module:

```ruby
require 'hobbit'
require 'hobbit/contrib'

class App < Hobbit::Base
  include Hobbit::Environment

  get '/' do
    "currently in #{environment}"
  end
end

run App.new
```

#### Available methods

* `environment`: Returns the current environment. By default is
`ENV['RACK_ENV']`.
* `environment=()`: Sets the environment.
* `development?`: Returns true if the current environment is `:development`.
* `production?`: Returns true if the current environment is `:production`.
* `test?`: Returns true if the current environment is `:test`.

### Hobbit::ErrorHandling

This extension provides a way of handling errors raised by your application. To
use this extension just include the module:

```ruby
require 'hobbit'
require 'hobbit/contrib'

class App < Hobbit::Base
  include Hobbit::ErrorHandling

  error Exception do |exception|
    exception.message
  end

  get '/' do
    raise Exception, 'Oops'
  end
end

run App.new
```

#### Available methods

* `errors`: Returns a hash with the exceptions being handled and its
corresponding handler.
* `error`: Sets a handler for a given exception.

**Note**: If you define more than one handler per exception the last one
defined will have precedence over the others.

### Hobbit::Filter

This extension provides a way of calling blocks before and after the
evaluation of a route (just like sinatra's filters). To use this extension just
include the module:

```ruby
require 'hobbit'
require 'hobbit/contrib'

class App < Hobbit::Base
  include Hobbit::Filter

  def authenticate_user!
    # ...
  end

  before do
    authenticate_user!
  end

  get '/' do
    # ...
  end
end

run App.new
```

#### Available methods

* `after`: Sets an after filter. Optionally, you can specify a route.
* `before`: Sets a before filter. Optionally, you can specify a route.

**Note**: It is recommended to include `Hobbit::Filter` before
`Hobbit::ErrorHandling` if you want to use both extensions.

### Hobbit::Render

This module provides rendering to your hobbit application. To use this
extension just include the module:

```ruby
require 'hobbit'
require 'hobbit/contrib'

class App < Hobbit::Base
  include Hobbit::Render

  get '/' do
    # will render views/index.erb using views/layouts/application.erb as layout
    render 'index'
  end
end
```

#### Available methods

* `default_layout`: Returns the name of the default layout (Override to use
another layout).
* `find_template`: Returns the path to a template (Override for multiple views
paths lookup).
* `layouts_path`: Returns the layouts path (Override to change the layouts
directory).
* `partial`: Renders the given template using tilt (without a layout).
* `render`: Renders the given template using tilt. You can pass a `layout`
option to change the layout (or set to `false` to not use one).
* `template_engine`: Returns the template engine beign used (Override to use
other template engine).
* `views_path`: Returns the views path (Override to change the views
directory).

### Hobbit::Session

This module provides helper methods for handling user sessions. To use this
extension just include the module:

```ruby
require 'hobbit'
require 'hobbit/contrib'

class App < Hobbit::Base
  include Hobbit::Session
  use Rack::Session::Cookie, secret: SecureRandom.hex(64)

  post '/' do
    session[:name] = 'hobbit'
  end

  get '/' do
    "Hello #{session[:name]}!"
  end
end
```

#### Available methods

* `session`: Returns the user's session.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## License

See the [LICENSE](https://github.com/patriciomacadden/hobbit-contrib/blob/master/LICENSE).
