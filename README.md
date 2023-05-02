# Testing Devise w Google Omniauth in Rails 7

1. Create new rails app:
```
rails new test-devise-omniauth --css=bootstrap --javascript=esbuild --database=postgresql
```

2. We can now run bundle install to install the correct version of the gem.
```
bundle install
```

3. Now that our application is ready, let's type the bin/setup command to install the dependencies and create the database:
```
bin/setup
```

4. We can now run the rails server, and the scripts that precompile the CSS and the JavaScript code with the bin/dev command:
```
bin/dev
```

5. We can now go to http://localhost:3000, and we should see the Rails boot screen.

6. Create a new controller named "welcome" by running the following command in your terminal:
```
rails generate controller Welcome index
```
This command will generate a new controller named "Welcome" and a view named "index". The view is located in the app/views/welcome directory.

7. Open the config/routes.rb file and add the following line:
```
root 'welcome#index'
```

Everything is set for tutorial here: https://youtu.be/CnZnwV38cjo

8. Add the gems we need:
```
gem 'devise'
gem 'omniauth'
gem 'omniauth-google-oauth2'
gem 'omniauth-rails_csrf_protection'
```
Then run ```bundle install```

9. Then do the Devise install:
```
bin/rails g devise:install
```

10. add to Devise initializer devise.rb
```
config.omniauth :google_oauth2, ENV['google_oauth_client_id'], ENV['google_oauth_client_secret']
```

11. Create User Model:
```
bin/rails g devise user
```

12. add to Devise model - app/models/user.rb
```
:omniauthable, omniauth_providers: [:google_oauth2]
```

13. Jump into the migration and add the following:
```
## our additons:
t.string :first_name
t.string :last_name
t.string :uid
t.string :avatar_url
t.string :provider
```

14. Run ```bin/rails db:migrate```

* this is at 4:34 of 26:39: https://youtu.be/CnZnwV38cjo?t=274

15. create page:
```
bin/rails g controller pages home
```

16. adjust devise_for :users in routes.rb
```
devise_for :users, controllers: {
    registrations: 'user/registrations',
    sessions: 'users/sessions',
    omniauth_callbacks: 'users/omniauth_callbacks'
  }
```

17. generate Devise controller users:
```
bin/rails g devise:controllers users
```

18. add methods to app/controllers/users/sessions_controller.rb:
```
def after_sign_out_path_for(_resource_or_scope)
    new_user_session_path
  end

  def after_sign_in_path_for(resource_or_scope)
    stored_location_for(resource_or_scope) || root_path
  end
```
We are here: https://youtu.be/CnZnwV38cjo?t=411

19. add to app/users/omniauth_callbacks_controller.rb
```
def google_oauth2
    user = User.from_omniauth(auth)
end
```

20. add to User.rb
```
def self.from_omniauth(auth)
    where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
end
```

21. add to omniauth_callbacks_controller.rb
```
private

  def auth
    @auth ||= request.env['omniauth.auth']
  end
```

22. make another adjustment to user.rb:
```
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable,
         :omniauthable, omniauth_providers: [:google_oauth2]


  def self.from_omniauth(auth)
    where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
      user.email = auth.info.email
      user.password = Devise.friendly_token[0,20]
      #user.full_name = auth.info.name
      user.first_name = auth.info.name
      user.avatar_url = auth.info.image
      #if you are using confirmable and the provider(s) you use validate emails,
      #uncomment the line below to skip confirmation emails:
      # user.skip_confirmation!
    end
  end
end
```

23. go back to adjusting omniauth_callbacks_controller.rb
```
  def google_oauth2
    user = User.from_omniauth(auth)

    if user.present?
      sign_out_all_scopes
      flash[:success] = t 'devise.omniauth_callbacks.success', kind: 'Google'
      sign_in_and_redirect user, event: :authentication
    else
      flash[:alert] =
      t 'devise.omniauth_callbacks.failure', kind: 'Google', reason: "#{auth.info.email} is not authorized."
      redirect_to new_user_session_path
    end
  end
```

24. 