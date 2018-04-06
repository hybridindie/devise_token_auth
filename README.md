# Devise Token Auth

[![Gem Version](https://badge.fury.io/rb/devise_token_auth.svg)](http://badge.fury.io/rb/devise_token_auth)
[![Build Status](https://travis-ci.org/lynndylanhurley/devise_token_auth.svg?branch=master)](https://travis-ci.org/lynndylanhurley/devise_token_auth)
[![Code Climate](https://codeclimate.com/github/lynndylanhurley/devise_token_auth/badges/gpa.svg)](https://codeclimate.com/github/lynndylanhurley/devise_token_auth)
[![Test Coverage](https://codeclimate.com/github/lynndylanhurley/devise_token_auth/badges/coverage.svg)](https://codeclimate.com/github/lynndylanhurley/devise_token_auth/coverage)
[![Dependency Status](https://gemnasium.com/lynndylanhurley/devise_token_auth.svg)](https://gemnasium.com/lynndylanhurley/devise_token_auth)
[![Downloads](https://img.shields.io/gem/dt/devise_token_auth.svg)](https://rubygems.org/gems/devise_token_auth)
[![Backers on Open Collective](https://opencollective.com/devise_token_auth/backers/badge.svg)](#backers)
[![Sponsors on Open Collective](https://opencollective.com/devise_token_auth/sponsors/badge.svg)](#sponsors)
[![Join the chat at https://gitter.im/lynndylanhurley/devise_token_auth](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/lynndylanhurley/devise_token_auth?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Simple, multi-client and secure token-based authentication for Rails.

If you're building SPA or a mobile app, and you want authentication, you need tokens, not cookies.
This gem refreshes the tokens on each request, and expires them in a short time, so the app is secure.
Also, it maintains a session for each client/device, so you can have as many sessions as you want.

## Main features

* Seamless integration with:
  * [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth) for [AngularJS](https://github.com/angular/angular.js)
  * [Angular2-Token](https://github.com/neroniaky/angular2-token) for [Angular2](https://github.com/angular/angular)
  * [redux-token-auth](https://github.com/kylecorbelli/redux-token-auth) for [React with Redux](https://github.com/reactjs/react-redux)
  * [jToker](https://github.com/lynndylanhurley/j-toker) for [jQuery](https://jquery.com/)
* Oauth2 authentication using [OmniAuth](https://github.com/intridea/omniauth).
* Email authentication using [Devise](https://github.com/plataformatec/devise), including:
  * User registration, update and deletion
  * Login and logout
  * Password reset, account confirmation
* Support for [multiple user models](./docs/usage/multiple_models.md).
* It is [secure](docs/security.md).

This project leverages the following gems:

* [Devise](https://github.com/plataformatec/devise)
* [OmniAuth](https://github.com/intridea/omniauth)

## Installation

Add the following to your `Gemfile`:

~~~ruby
gem 'devise_token_auth'
~~~

Then install the gem using bundle:

~~~bash
bundle install
~~~

## [Docs](https://maicolben.gitbooks.io/devise-token-auth/content/docs/config/)
=======
# Configuration TL;DR

You will need to create a [user model](#model-concerns), [define routes](#mounting-routes), [include concerns](#controller-methods), and you may want to alter some of the [default settings](#initializer-settings) for this gem. Run the following command for an easy one-step installation:

~~~bash
rails g devise_token_auth:install [USER_CLASS] [MOUNT_PATH] [ORM]
~~~

**Example**:

~~~bash
rails g devise_token_auth:install User auth
~~~

This generator accepts the following optional arguments:

| Argument | Default | Description |
|---|---|---|
| USER_CLASS | `User` | The name of the class to use for user authentication. |
| MOUNT_PATH | `auth` | The path at which to mount the authentication routes. [Read more](#usage-tldr). |
| ORM | `active_record` | Using active_record or mongoid|

The following events will take place when using the install generator:

* An initializer will be created at `config/initializers/devise_token_auth.rb`. [Read more](#initializer-settings).

* A model will be created in the `app/models` directory. If the model already exists, a concern will be included at the top of the file. [Read more](#model-concerns).

* Routes will be appended to file at `config/routes.rb`. [Read more](#mounting-routes).

* A concern will be included by your application controller at `app/controllers/application_controller.rb`. [Read more](#controller-methods).

* A migration file will be created in the `db/migrate` directory. Inspect the migrations file, add additional columns if necessary, and then run the migration:

  ~~~bash
  rake db:migrate
  ~~~

You may also need to configure the following items:

* **OmniAuth providers** when using 3rd party oauth2 authentication. [Read more](#omniauth-authentication).
* **Cross Origin Request Settings** when using cross-domain clients. [Read more](#cors).
* **Email** when using email registration. [Read more](#email-authentication).
* **Multiple model support** may require additional steps. [Read more](#using-multiple-models).

[Jump here](#configuration-cont) for more configuration information.

# Usage TL;DR

The following routes are available for use by your client. These routes live relative to the path at which this engine is mounted (`auth` by default). These routes correspond to the defaults used by the [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth) module for [AngularJS](https://angularjs.org/) and the [jToker](https://github.com/lynndylanhurley/j-toker) plugin for [jQuery](https://jquery.com/).

| path | method | purpose |
|:-----|:-------|:--------|
| /    | POST   | Email registration. Requires **`email`**, **`password`**, **`password_confirmation`**, and **`confirm_success_url`** params (this last one can be omitted if you have set `config.default_confirm_success_url` in `config/initializers/devise_token_auth.rb`). A verification email will be sent to the email address provided. Upon clicking the link in the confirmation email, the API will redirect to the URL specified in **`confirm_success_url`**. Accepted params can be customized using the [`devise_parameter_sanitizer`](https://github.com/plataformatec/devise#strong-parameters) system. |
| / | DELETE | Account deletion. This route will destroy users identified by their **`uid`**, **`access-token`** and **`client`** headers. |
| / | PUT | Account updates. This route will update an existing user's account settings. The default accepted params are **`password`** and **`password_confirmation`**, but this can be customized using the [`devise_parameter_sanitizer`](https://github.com/plataformatec/devise#strong-parameters) system. If **`config.check_current_password_before_update`** is set to `:attributes` the **`current_password`** param is checked before any update, if it is set to `:password` the **`current_password`** param is checked only if the request updates user password. |
| /sign_in | POST | Email authentication. Requires **`email`** and **`password`** as params. This route will return a JSON representation of the `User` model on successful login along with the `access-token` and `client` in the header of the response. |
| /sign_out | DELETE | Use this route to end the user's current session. This route will invalidate the user's authentication token. You must pass in **`uid`**, **`client`**, and **`access-token`** in the request headers. |
| /:provider | GET | Set this route as the destination for client authentication. Ideally this will happen in an external window or popup. [Read more](#omniauth-authentication). |
| /:provider/callback | GET/POST | Destination for the oauth2 provider's callback uri. `postMessage` events containing the authenticated user's data will be sent back to the main client window from this page. [Read more](#omniauth-authentication). |
| /validate_token | GET | Use this route to validate tokens on return visits to the client. Requires **`uid`**, **`client`**, and **`access-token`** as params. These values should correspond to the columns in your `User` table of the same names. |
| /password | POST | Use this route to send a password reset confirmation email to users that registered by email. Accepts **`email`** and **`redirect_url`** as params. The user matching the `email` param will be sent instructions on how to reset their password. `redirect_url` is the url to which the user will be redirected after visiting the link contained in the email. |
| /password | PUT | Use this route to change users' passwords. Requires **`password`** and **`password_confirmation`** as params. This route is only valid for users that registered by email (OAuth2 users will receive an error). It also checks **`current_password`** if **`config.check_current_password_before_update`** is not set `false` (disabled by default). |
| /password/edit | GET | Verify user by password reset token. This route is the destination URL for password reset confirmation. This route must contain **`reset_password_token`** and **`redirect_url`** params. These values will be set automatically by the confirmation email that is generated by the password reset request. |

[Jump here](#usage-cont) for more usage information.

# Configuration cont.

## Initializer settings

The following settings are available for configuration in `config/initializers/devise_token_auth.rb`:

| Name | Default | Description|
|---|---|---|
| **`change_headers_on_each_request`** | `true` | By default the access-token header will change after each request. The client is responsible for keeping track of the changing tokens. Both [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth) and [jToker](https://github.com/lynndylanhurley/j-toker) do this out of the box. While this implementation is more secure, it can be difficult to manage. Set this to false to prevent the `access-token` header from changing after each request. [Read more](#about-token-management). |
| **`token_lifespan`** | `2.weeks` | Set the length of your tokens' lifespans. Users will need to re-authenticate after this duration of time has passed since their last login. |
| **`batch_request_buffer_throttle`** | `5.seconds` | Sometimes it's necessary to make several requests to the API at the same time. In this case, each request in the batch will need to share the same auth token. This setting determines how far apart the requests can be while still using the same auth token. [Read more](#about-batch-requests). |
| **`omniauth_prefix`** | `"/omniauth"` | This route will be the prefix for all oauth2 redirect callbacks. For example, using the default '/omniauth' setting, the github oauth2 provider will redirect successful authentications to '/omniauth/github/callback'. [Read more](#omniauth-provider-settings). |
| **`default_confirm_success_url`** | `nil` | By default this value is expected to be sent by the client so that the API knows where to redirect users after successful email confirmation. If this param is set, the API will redirect to this value when no value is provided by the client. |
| **`default_password_reset_url`** | `nil` | By default this value is expected to be sent by the client so that the API knows where to redirect users after successful password resets. If this param is set, the API will redirect to this value when no value is provided by the client. |
| **`redirect_whitelist`** | `nil` | As an added security measure, you can limit the URLs to which the API will redirect after email token validation (password reset, email confirmation, etc.). This value should be an array containing matches to the client URLs to be visited after validation. Wildcards are supported. |
| **`enable_standard_devise_support`** | `false` | By default, only Bearer Token authentication is implemented out of the box. If, however, you wish to integrate with legacy Devise authentication, you can do so by enabling this flag. NOTE: This feature is highly experimental! |
| **`remove_tokens_after_password_reset`** | `false` | By default, old tokens are not invalidated when password is changed. Enable this option if you want to make passwords updates to logout other devices. |
| **`default_callbacks`** | `true` | By default User model will include the `DeviseTokenAuth::Concerns::UserOmniauthCallbacks` concern, which has `email`, `uid` validations & `uid` synchronization callbacks. |
| **`bypass_sign_in`** | `true` | By default DeviseTokenAuth will not check user's `#active_for_authentication?` which includes confirmation check on each call (it will do it only on sign in). If you want it to be validated on each request (for example, to be able to deactivate logged in users on the fly), set it to false. |


Additionally, you can configure other aspects of devise by manually creating the traditional devise.rb file at `config/initializers/devise.rb`. Here are some examples of what you can do in this file:

~~~ruby
Devise.setup do |config|
  # The e-mail address that mail will appear to be sent from
  # If absent, mail is sent from "please-change-me-at-config-initializers-devise@example.com"
  config.mailer_sender = "support@myapp.com"

  # If using rails-api, you may want to tell devise to not use ActionDispatch::Flash
  # middleware b/c rails-api does not include it.
  # See: http://stackoverflow.com/q/19600905/806956
  config.navigational_formats = [:json]
end
~~~

## OmniAuth authentication

If you wish to use omniauth authentication, add all of your desired authentication provider gems to your `Gemfile`.

**OmniAuth example using github, facebook, and google**:
~~~ruby
gem 'omniauth-github'
gem 'omniauth-facebook'
gem 'omniauth-google-oauth2'
~~~

Then run `bundle install`.

[List of oauth2 providers](https://github.com/intridea/omniauth/wiki/List-of-Strategies)

## OmniAuth provider settings

In `config/initializers/omniauth.rb`, add the settings for each of your providers.

These settings must be obtained from the providers themselves.

**Example using github, facebook, and google**:
~~~ruby
# config/initializers/omniauth.rb
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :github,        ENV['GITHUB_KEY'],   ENV['GITHUB_SECRET'],   scope: 'email,profile'
  provider :facebook,      ENV['FACEBOOK_KEY'], ENV['FACEBOOK_SECRET']
  provider :google_oauth2, ENV['GOOGLE_KEY'],   ENV['GOOGLE_SECRET']
end
~~~

The above example assumes that your provider keys and secrets are stored in environmental variables. Use the [figaro](https://github.com/laserlemon/figaro) gem (or [dotenv](https://github.com/bkeepers/dotenv) or [secrets.yml](https://github.com/rails/rails/blob/v4.1.0/railties/lib/rails/generators/rails/app/templates/config/secrets.yml) or equivalent) to accomplish this.

#### OmniAuth callback settings

The "Callback URL" setting that you set with your provider must correspond to the [omniauth prefix](#initializer-settings) setting defined by this app. **This will be different than the omniauth route that is used by your client application**.

For example, the demo app uses the default `omniauth_prefix` setting `/omniauth`, so the "Authorization callback URL" for github must be set to "http://devise-token-auth-demo.herokuapp.com**/omniauth**/github/callback".

**Github example for the demo site**:
![password reset flow](https://github.com/lynndylanhurley/devise_token_auth/raw/master/test/dummy/app/assets/images/omniauth-provider-settings.png)

The url for github authentication will be different for the client. The client should visit the API at `/[MOUNT_PATH]/:provider` for omniauth authentication.

For example, given that the app is mounted using the following settings:

~~~ruby
# config/routes.rb
mount_devise_token_auth_for 'User', at: 'auth'
~~~

The client configuration for github should look like this:

**Angular.js setting for authenticating using github**:
~~~javascript
angular.module('myApp', ['ng-token-auth'])
  .config(function($authProvider) {
    $authProvider.configure({
      apiUrl: 'http://api.example.com'
      authProviderPaths: {
        github: '/auth/github' // <-- note that this is different than what was set with github
      }
    });
  });
~~~

**jToker settings for github should look like this:

~~~javascript
$.auth.configure({
  apiUrl: 'http://api.example.com',
  authProviderPaths: {
    github: '/auth/github' // <-- note that this is different than what was set with github
  }
});
~~~

This incongruence is necessary to support multiple user classes and mounting points.

#### Note for [pow](http://pow.cx/) and [xip.io](http://xip.io) users

If you receive `redirect-uri-mismatch` errors from your provider when using pow or xip.io urls, set the following in your development config:

~~~ruby
# config/environments/development.rb

# when using pow
OmniAuth.config.full_host = "http://app-name.dev"

# when using xip.io
OmniAuth.config.full_host = "http://xxx.xxx.xxx.app-name.xip.io"
~~~

## Email authentication
If you wish to use email authentication, you must configure your Rails application to send email. [Read here](http://guides.rubyonrails.org/action_mailer_basics.html) for more information.

I recommend using [mailcatcher](http://mailcatcher.me/) for development.

##### mailcatcher development example configuration:
~~~ruby
# config/environments/development.rb
Rails.application.configure do
  config.action_mailer.default_url_options = { :host => 'your-dev-host.dev' }
  config.action_mailer.delivery_method = :smtp
  config.action_mailer.smtp_settings = { :address => 'your-dev-host.dev', :port => 1025 }
end
~~~

If you wish to send custom e-mails instead of using the default devise templates, you can [do that too](#email-template-overrides).

## Customizing Devise Verbiage
Devise Token Auth ships with intelligent default wording for everything you need. But that doesn't mean you can't make it more awesome. You can override the [devise defaults](https://github.com/plataformatec/devise/blob/master/config/locales/en.yml) by creating a YAML file at `config/locales/devise.en.yml` and assigning whatever custom values you want. For example, to customize the subject line of your devise e-mails, you could do this:

~~~yaml
en:
  devise:
    mailer:
      confirmation_instructions:
        subject: "Please confirm your e-mail address"
      reset_password_instructions:
        subject: "Reset password request"
~~~

## CORS

If your API and client live on different domains, you will need to configure your Rails API to allow [cross origin requests](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing). The [rack-cors](https://github.com/cyu/rack-cors) gem can be used to accomplish this.

The following **dangerous** example will allow cross domain requests from **any** domain. Make sure to whitelist only the needed domains.

##### Example rack-cors configuration:
~~~ruby
# gemfile
gem 'rack-cors', :require => 'rack/cors'

# config/application.rb
module YourApp
  class Application < Rails::Application
    config.middleware.use Rack::Cors do
      allow do
        origins '*'
        resource '*',
          :headers => :any,
          :expose  => ['access-token', 'expiry', 'token-type', 'uid', 'client'],
          :methods => [:get, :post, :options, :delete, :put]
      end
    end
  end
end
~~~

Make extra sure that the `Access-Control-Expose-Headers` includes `access-token`, `expiry`, `token-type`, `uid`, and `client` (as is set in the example above by the`:expose` param). If your client experiences erroneous 401 responses, this is likely the cause.

CORS may not be possible with older browsers (IE8, IE9). I usually set up a proxy for those browsers. See the [ng-token-auth readme](https://github.com/lynndylanhurley/ng-token-auth) or the [jToker readme](https://github.com/lynndylanhurley/j-toker) for more information.

# Usage cont.

## Mounting Routes

The authentication routes must be mounted to your project. This gem includes a route helper for this purpose:

**`mount_devise_token_auth_for`** - similar to `devise_for`, this method is used to append the routes necessary for user authentication. This method accepts the following arguments:

| Argument | Type | Default | Description |
|---|---|---|---|
|`class_name`| string | 'User' | The name of the class to use for authentication. This class must include the [model concern described here](#model-concerns). |
| `options` | object | {at: 'auth'} | The [routes to be used for authentication](#usage) will be prefixed by the path specified in the `at` param of this object. |

**Example**:
~~~ruby
# config/routes.rb
mount_devise_token_auth_for 'User', at: 'auth'
~~~

Any model class can be used, but the class will need to include [`DeviseTokenAuth::Concerns::User`](#model-concerns) for authentication to work properly.

You can mount this engine to any route that you like. `/auth` is used by default to conform with the defaults of the [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth) module and the [jToker](https://github.com/lynndylanhurley/j-toker) plugin.


## Controller Methods

### Concerns

This gem includes a [Rails concern](http://api.rubyonrails.org/classes/ActiveSupport/Concern.html) called `DeviseTokenAuth::Concerns::SetUserByToken`. Include this concern to provide access to [controller methods](#controller-methods) such as [`authenticate_user!`](#authenticate-user), [`user_signed_in?`](#user-signed-in), etc.

The concern also runs an [after_action](http://guides.rubyonrails.org/action_controller_overview.html#filters) that changes the auth token after each request.

It is recommended to include the concern in your base `ApplicationController` so that all children of that controller include the concern as well.

##### Concern example:

~~~ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  include DeviseTokenAuth::Concerns::SetUserByToken
end
~~~

### Methods

This gem provides access to all of the following [devise helpers](https://github.com/plataformatec/devise#controller-filters-and-helpers):

| Method | Description |
|---|---|
| **`before_action :authenticate_user!`** | Returns a 401 error unless a `User` is signed-in. |
| **`current_user`** | Returns the currently signed-in `User`, or `nil` if unavailable. |
| **`user_signed_in?`** | Returns `true` if a `User` is signed in, otherwise `false`. |
| **`devise_token_auth_group`** | Operate on multiple user classes as a group. [Read more](#group-access) |

Note that if the model that you're trying to access isn't called `User`, the helper method names will change. For example, if the user model is called `Admin`, the methods would look like this:

* `before_action :authenticate_admin!`
* `admin_signed_in?`
* `current_admin`


##### Example: limit access to authenticated users
~~~ruby
# app/controllers/test_controller.rb
class TestController < ApplicationController
  before_action :authenticate_user!

  def members_only
    render json: {
      data: {
        message: "Welcome #{current_user.name}",
        user: current_user
      }
    }, status: 200
  end
end
~~~

### Token Header Format

The authentication information should be included by the client in the headers of each request. The headers follow the [RFC 6750 Bearer Token](http://tools.ietf.org/html/rfc6750) format:

##### Authentication headers example:
~~~
"access-token": "wwwww",
"token-type":   "Bearer",
"client":       "xxxxx",
"expiry":       "yyyyy",
"uid":          "zzzzz"
~~~

The authentication headers (each one is a seperate header) consists of the following params:

| param | description |
|---|---|
| **`access-token`** | This serves as the user's password for each request. A hashed version of this value is stored in the database for later comparison. This value should be changed on each request. |
| **`client`** | This enables the use of multiple simultaneous sessions on different clients. (For example, a user may want to be authenticated on both their phone and their laptop at the same time.) |
| **`expiry`** | The date at which the current session will expire. This can be used by clients to invalidate expired tokens without the need for an API request. |
| **`uid`** | A unique value that is used to identify the user. This is necessary because searching the DB for users by their access token will make the API susceptible to [timing attacks](http://codahale.com/a-lesson-in-timing-attacks/). |

## Need help?

Please use [StackOverflow](https://stackoverflow.com/questions/tagged/devise-token-auth) for help requests and how-to questions.

Please open GitHub issues for bugs and enhancements only, not general help requests. Please search previous issues (and Google and StackOverflow) before creating a new issue.

Please read the [ISSUE_TEMPLATE.md] before posting issues.

## [FAQ](docs/faq.md)

## Contributors wanted!

See our [Contribution Guidelines](https://github.com/lynndylanhurley/devise_token_auth/blob/master/.github/CONTRIBUTING.md). Feel free to submit pull requests, review pull requests, or review open issues. If you'd like to get in contact, [Zach Feldman](https://github.com/zachfeldman) has been wrangling this effort, you can reach him with his name @gmail. Further discussion of this in [this issue](https://github.com/lynndylanhurley/devise_token_auth/issues/969).

## Live Demos

[Here is a demo](http://ng-token-auth-demo.herokuapp.com/) of this app running with the [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth) module and [AngularJS](https://github.com/angular/angular.js).

[Here is a demo](https://angular2-token.herokuapp.com) of this app running with the [Angular2-Token](https://github.com/neroniaky/angular2-token) service and [Angular2](https://github.com/angular/angular).

[Here is a demo](https://j-toker-demo.herokuapp.com/) of this app using the [jToker](https://github.com/lynndylanhurley/j-toker) plugin and [React](http://facebook.github.io/react/).

The fully configured api used in these demos can be found [here](https://github.com/lynndylanhurley/devise_token_auth_demo).


## Contributors

<a href="graphs/contributors"><img src="https://opencollective.com/devise_token_auth/contributors.svg?width=890&button=false" /></a>

## Backers

Thank you to all our backers! 🙏 [[Become a backer](https://opencollective.com/devise_token_auth#backer)]

[![](https://opencollective.com/devise_token_auth/backers.svg?width=890)](https://opencollective.com/devise_token_auth#backers)


## Sponsors

Support this project by becoming a sponsor. Your logo will show up here with a link to your website. [[Become a sponsor](https://opencollective.com/devise_token_auth#sponsor)]

[![](https://opencollective.com/devise_token_auth/sponsor/0/avatar.svg)](https://opencollective.com/devise_token_auth/sponsor/0/website) [![](https://opencollective.com/devise_token_auth/sponsor/1/avatar.svg)](https://opencollective.com/devise_token_auth/sponsor/1/website) [![](https://opencollective.com/devise_token_auth/sponsor/2/avatar.svg)](https://opencollective.com/devise_token_auth/sponsor/2/website) [![](https://opencollective.com/devise_token_auth/sponsor/3/avatar.svg)](https://opencollective.com/devise_token_auth/sponsor/3/website) [![](https://opencollective.com/devise_token_auth/sponsor/4/avatar.svg)](https://opencollective.com/devise_token_auth/sponsor/4/website) [![](https://opencollective.com/devise_token_auth/sponsor/5/avatar.svg)](https://opencollective.com/devise_token_auth/sponsor/5/website) [![](https://opencollective.com/devise_token_auth/sponsor/6/avatar.svg)](https://opencollective.com/devise_token_auth/sponsor/6/website) [![](https://opencollective.com/devise_token_auth/sponsor/7/avatar.svg)](https://opencollective.com/devise_token_auth/sponsor/7/website) [![](https://opencollective.com/devise_token_auth/sponsor/8/avatar.svg)](https://opencollective.com/devise_token_auth/sponsor/8/website) [![](https://opencollective.com/devise_token_auth/sponsor/9/avatar.svg)](https://opencollective.com/devise_token_auth/sponsor/9/website)

## License
This project uses the WTFPL
