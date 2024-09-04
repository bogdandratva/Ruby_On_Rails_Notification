# Notifications

Mountable notifications for any Rails applications.

## Installation

```bash
$ bundle add notifications
```

You now have a notifications generator in your Rails application:

```bash
$ rails g notifications:install
```

You can generate views, controllers if you need to customize them:

```bash
$ rails g notifications:views
$ rails g notifications:controllers
```

## Usage

### Create a Notification

```ruby
class User
  def follow(user)
    Notification.create(notify_type: 'follow', actor: self, user: user)
  end
end

class Comment
  belongs_to :post
  belongs_to :user

  after_commit :create_notifications, on: [:create]
  def create_notifications
    Notification.create(
      notify_type: 'comment',
      actor: self.user,
      user: self.post.user,
      target: self)
  end
end
```

Get unread notifications count for a user:

```rb
# unread count
unread_count = Notification.unread_count(current_user)

# read count
read_count = Notification.read_count(current_user)

```

```rb initialize/**.rb
# for non-user class
Notifications.config.user_class = 'Member'

#or change

Notifications.configure do
  # Class name of you User model, default: 'User'
  self.user_class = 'User'

  # Method of user name in User model, default: 'name'
  # self.user_name_method = 'name'

  # Method of user avatar in User model, default: nil
  # self.user_avatar_url_method = nil

  # Method name of user profile page path, in User model, default: nil
  # self.user_profile_url_method = 'profile_url'

  # authenticate_user method in your Controller, default: nil
  # If you use Devise, authenticate_user! is correct
  # self.authenticate_user_method = 'authenticate_user!'

  # current_user method name in your Controller, default: 'current_user'
  # If you use Devise, current_user is correct
  # self.current_user_method = 'current_user'
end


```

### Write your custom Notification partial view for notify_types:

If you create a notify_type, you need to add a partial view in `app/views/notifications/` path, for example:

```rb
# There have two notify_type
Notification.create(notify_type: 'follow' ....)
Notification.create(notify_type: 'mention', target: @reply, second_target: @topic, ....)
```

Your app must have:

- app/views/notifications/_follow.html.erb
- app/views/notifications/_mention.html.erb

```erb
# app/views/notifications/_follow.html.erb
<div class="media-heading">
  <%= link_to notification.actor.title, main_app.user_path(notification.actor) %> just followed you.
</div>
```

```erb
# app/views/notifications/_mention.html.erb
<div class="media-heading">
  <%= link_to notification.actor.title, main_app.user_path(notification.actor) %> has mentioned you in
  <%= link_to notification.second_target.title, main_app.topic_path(notification.second_target) %>
</div>
<div class="media-content">
  <%= notification.target.body %>
</div>
```

> NOTE: When you want use Rails route path name in notification views, you must use [main_app](http://api.rubyonrails.org/classes/Rails/Engine.html#class-Rails::Engine-label-Using+Engine-27s+routes+outside+Engine) prefix. etc: `main_app.user_path(user)`

### About Notification template N+1 performance

It is recommended that you use [second_level_cache](https://github.com/hooopo/second_level_cache) for solving N+1 performance issues.

## Contributing

Testing for multiple Rails versions:

```bash
make test_51
# or test all
make test
```

## License
The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
