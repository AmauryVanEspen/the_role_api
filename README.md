# gem 'the_role' (alpha v0.1)

| Bye bye CanCan, I got The Role! | Description |
|:------------- |:-------------|
| ![Bye bye CanCan, I got The Role!](https://github.com/the-teacher/the_role/raw/master/Bye_bye_CanCan_I_got_the_Role.png) | TheRole is an authorization library for Ruby on Rails which restricts what resources a given user is allowed to access. All permissions are defined in with 2-level-hash, and store in database with JSON.<br><br>TheRole - Semantic, lightweight role system with an administrative interface.<br><br>Role is a two-level hash, consisting of the **sections** and nested **rules**.<br><br>**Section** may be associated with **controller** name.<br><br>**Rule** may be associated with **action** name.<br><br>Section can have many rules.<br><br>Rule can have **true** or **false** value<br><br>**Sections** and nested **Rules** provide **ACL** (**Access Control List**)<br><br>Role **stored in the database as JSON** string.<br><br>Using of hashes, makes role system extremely easy to configure and use.<br> |

### GUI

| TheRole management web interface |
|:-------------:|
|![TheRole](https://github.com/the-teacher/the_role/raw/master/pic.png)|

### rubygems page

http://rubygems.org/gems/the_role

### TheRole and Devise 2

[Integration with Devise2](https://github.com/the-teacher/the_role/wiki/Integration-with-Devise2)

### TheRole and Sorcery

[Integration with Sorcery](https://github.com/the-teacher/the_role/wiki/Integration-with-Sorcery)

### Want to improve this gem?

[Want to improve this gem?](https://github.com/the-teacher/the_role/wiki/Want-to-improve-this-gem%3F)

### Rspec for TheRole

[Specs with Devise 2](https://github.com/the-teacher/devise2_on_the_role/tree/master/spec)

Read **Want to improve this gem?** manual for running specs

## What does it mean semantic?

Semantic - the science of meaning. Human should fast to understand what is happening in a role system.

Look at hash. If you can understand access rules - this role system is semantically.

``` ruby
role = {
  'pages' => {
    'index'   => true,
    'show'    => true,
    'new'     => false,
    'edit'    => false,
    'update'  => false,
    'destroy' => false
  },
  'articles' => {
    'index'  => true,
    'show'   => true
  },
  'twitter'  => {
    'button' => true,
    'follow' => false
  }
}
```

### Virtual sections and rules

Usually, we use real names of controllers and actions for names of sections and rules:

``` ruby
current_user.has_role?(:pages, :show)
```

But, also, you can use virtual names of sections, and virtual names of section's rules.

``` ruby
current_user.has_role?(:twitter, :button)
current_user.has_role?(:facebook, :like)
```

And you can use them as well as other access rules.

# Install

``` ruby
  gem 'the_role'
```

``` ruby
  bundle
```

### Migrate

Add **role_id:integer** to User Model Migration

``` ruby
rake the_role_engine:install:migrations
  >> Copied migration 20111028145956_create_roles.rb from the_role_engine
```

``` ruby
rails g model role --migration=false
```

``` ruby
rake db:create && rake db:migrate
```

### Fake roles for test (not required)

Creating roles for test 

``` ruby
rake db:roles:test
  >> Administrator, Moderator of pages, User, Demo
```

### Change your ApplicationController

**Example for Devise2**

``` ruby
class ApplicationController < ActionController::Base
  protect_from_forgery

  def access_denied
    render :text => 'access_denied: requires an role' and return
  end

  alias_method :login_required,     :authenticate_user!
  alias_method :role_access_denied, :access_denied

end
```

Define aliases method for correctly work TheRole's controllers

**authenticate_user!** or any other method from your auth system

**access_denied** or any other method for processing access denied situation


### Using with any controller

``` ruby
class PagesController < ApplicationController
  # Devise2 and TheRole before_filters
  before_filter :login_required, :except => [:index, :show]
  before_filter :role_required,  :except => [:index, :show]

  before_filter :find_page,      :only   => [:edit, :update, :destroy]
  before_filter :owner_required, :only   => [:edit, :update, :destroy]

  private

  def find_page
    @page = Page.find params[:id]
    @ownership_checking_object = @page
  end
end
```

**owner_required** method require **@ownership_checking_object** variable, with cheked object.

### Who is Administrator?

Administrator it's a user who can access any section and the rules of your application.

Administrator is the owner of any objects in your application.

Administrator it's a user, which has virtual section **system** and rule **administrator** in the role-hash.


``` ruby
admin_role_fragment = {
  :system => {
    :administrator => true
  }
}
```

### Who is Moderator?

Moderator it's a user, which has access to any actions of some section(s).

Moderator is's owner of any objects of some class.

Moderator it's a user, which has a virtual section **moderator**, with **section name** as rule name.

There is Moderator of Pages (controller) and Twitter (virtual section)

``` ruby
moderator_role_fragment = {
  :moderator => {
    :pages   => true,
    :blogs   => false,
    :twitter => true
  }
}
```

### Who is Owner?

Administrator is owner of any object in system.

Moderator of pages is owner of any page.

User is owner of object, when **Object#user_id == User#id**.

# User Model methods

Has a user an access to **rule** of **section** (action of controller)?

``` ruby
current_user.has_role?(:pages,    :show)  => true | false
current_user.has_role?(:blogs,    :new)   => true | false
current_user.has_role?(:articles, :edit)  => true | false
```

Is it Moderator?

``` ruby
current_user.moderator?(:pages)           => true | false
current_user.moderator?(:blogs)           => true | false
current_user.moderator?(:articles)        => true | false
```

Is it Administrator?

``` ruby
current_user.admin?                       => true | false
```

Is it **Owner** of object?

``` ruby
current_user.owner?(@page)                => true | false
current_user.owner?(@blog)                => true | false
current_user.owner?(@article)             => true | false
```

# Base Role methods

``` ruby
# User's role
@role = current_user.role
```

``` ruby
# Find a Role by name
@role = Role.find_by_name(:user)
```

``` ruby
@role.has?(:pages, :show)       => true | false
@role.moderator?(:pages)        => true | false
@role.admin?                    => true | false
```

# CRUD API (for console users)

#### CREATE

``` ruby
# Create a section of rules
@role.create_section(:pages)
```

``` ruby
# Create rule in section (false value by default)
@role.create_rule(:pages, :index)
```

#### READ

``` ruby
@role.to_hash => Hash

# JSON string
@role.to_json => String

# JSON string
@role.to_s => String
```

#### UPDATE

``` ruby
# Incoming hash is true-mask-hash
# All rules of Role will be reset to false
# Only rules from true-mask-hash will be set on true
new_role_hash = {
  :pages => {
    :index => true,
    :show => true
  }
}

@role.update_role(new_role_hash)
```

``` ruby
# set this rule on true
@role.rule_on(:pages, :index)
```

``` ruby
# set this rule on false
@role.rule_off(:pages, :index)
```

### DELETE

``` ruby
# delete a section
@role.delete_section(:pages)

# delete rule in section
@role.delete_rule(:pages, :show)
```