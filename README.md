[<img src="https://travis-ci.org/voormedia/flipflop.svg?branch=master" alt="Build Status">](https://travis-ci.org/voormedia/flipflop)

# Flipflop your features

**Flipflop** provides a declarative, layered way of enabling and disabling
application functionality at run-time. It is forked from [Flip](https://github.com/pda/flip).

This gem optimizes for:

* developer ease-of-use,
* visibility and control for other stakeholders (like marketing); and
* run-time performance

There are three layers of strategies per feature:

* default
* database, to flipflop features site-wide for all users
* cookie, to flipflop features for single users

There is also a configurable system-wide default - !Rails.env.production?` works nicely.

Flipflop has a dashboard UI that's easy to understand and use.

## Installation

Add the gem to your `Gemfile`:

```ruby
gem "flipflop"
```

Generate routes, feature model and databse migration:

```
rails g flipflop:install
```

Run the migration:

```
rake db:migrate
```

Explicitly include the Feature model, e.g. in `config/initializers/feature.rb`:
```
require "feature"
```

## Declaring Features

```ruby
# This is the model class generated by rails g flipflop:install
class Feature < ActiveRecord::Base
  include Flipflop::Declarable

  # The recommended Flipflop strategy stack.
  strategy Flipflop::CookieStrategy
  strategy Flipflop::DatabaseStrategy

  # A basic feature declaration.
  feature :shiny_things

  # Override the system-wide default.
  feature :world_domination, default: true
end
```

## Checking Features

`Flipflop.on?` or the dynamic predicate methods are used to check feature
state:

```ruby
Flipflop.on?(:world_domination)  # true
Flipflop.world_domination?       # true

Flipflop.on?(:shiny_things)      # false
Flipflop.shiny_things?           # false
```

This works everywhere, also in your views and controllers:

```erb
<div>
  <% if Flipflop.world_domination? %>
    <%= link_to "Dominate World", world_dominations_path %>
  <% end %>
</div>
```

## Feature Flipflopping Controllers

The `Flipflop::ControllerFilters` module is mixed into the base `ApplicationController` class.  The following controller will respond with 404 Page Not Found to all but the `index` action unless the `:something` feature is enabled:

```ruby
class SampleController < ApplicationController

  require_feature :something, :except => :index

  def show
  end

  def index
  end

end
```

## Dashboard

The dashboard provides visibility and control over the features.

The gem includes some basic styles:

```haml
= content_for :stylesheets_head do
  = stylesheet_link_tag "flipflop"
```

You probably don't want the dashboard to be public. Here's one way of
implementing access control.

app/controllers/admin/features_controller.rb:

```ruby
class Admin::FeaturesController < Flipflop::FeaturesController
  before_action :assert_authenticated_as_admin
end
```

app/controllers/admin/strategies_controller.rb:

```ruby
class Admin::StrategiesController < Flipflop::StrategiesController
  before_action :assert_authenticated_as_admin
end
```

routes.rb:

```ruby
namespace :admin do
  resources :features, only: [ :index ] do
    resources :strategies, only: [ :update, :destroy ]
  end
end

mount Flipflop::Engine => "/admin/features"
```

## License

This software is licensed under the MIT License. [View the license](LICENSE).
