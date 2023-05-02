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

24. Go to google: https://console.cloud.google.com/getting-started?pli=1

25. Click APIs & Services > Credentials - Click new project, name it
* Make sure you are on the right project, click + Create Credentials select OAuth client ID, click Configure Consent Screen - choose Internal or External - for this we are choosing External - click create
* Name the app, add support email,  add email again under Developer contact information - hit save and continue - then hit save and continue again.
* add test users
There was something here about authorizing  https://youtu.be/CnZnwV38cjo?t=877
* you may have to click out and come back in to see what you need here.
* Authorized redirect URIs needs to be: http://localhost:3000/users/auth/google_oauth2/callback
* get the client ID and the client secret


26. take a look at app/config/initializers/devise.rb - notice the ENV stuff - type this into the console:
```
EDITOR="code --wait" bin/rails credentials:edit
```

27. add the client ID and the secret to that file:
```
google_oauth_client_id: xxxxxxxx-xxxxxxxxxxuxxxxxxx.apps.googleusercontent.com
google_oauth_client_secret: xxxxxxxx-Rxxxxxxxxxxxx1kxxxxxx
```
**Hit ctrl-s and then close the file to save**

If it worked correctly you will see:
```
File encrypted and saved.
```

28. Alter the config.omniauth stuff in devise.rb
```
config.omniauth :google_oauth2, 
    Rails.application.credentials.dig(:google_oauth_client_id), 
    Rails.application.credentials.dig(:google_oauth_client_secret)
```

29. Add to the home page:
```
<% if current_user%>
    <h2><%= current_user.email %></h2>
    <%= link_to "Edit Account", edit_user_registration_path %>
    <%= button_to "Logout", destroy_user_session_path, data: {turbo: "false"}, method: :delete %>
<% else %>
    <%= link_to "Login", new_user_session_path %>
    <%= link_to "Create Account", new_user_registration_path %>
<% end %>
```

30. generate the views so we can change some things:
```
bin/rails g devise:views
```

31. go to google and search "google login icon" go to the Branding Guidlines - download files - grab the ones within web>1x and add them to app/assets/images/

32. then go into app > views > Devise > Shared > Links and change the omniauthable one at the bottom:
```
<%- if devise_mapping.omniauthable? %>
  <%- resource_class.omniauth_providers.each do |provider| %>
    <%= form_for "Login",
      url: omniauth_authorize_path(resource_name, provider),
      method: :post,
      data: {turbo: "false"} do |f| %>
         <%= f.submit "Login", type: "image", src: url_for("/images/btn_google_signin_light_normal_web.png") %> 
    <% end %>
  <% end %>
<% end %>
```

Test and see if it is working. It is!!!!

33. added to registrations_controller.rb
```
def update_resource(resource, params)
    if resource.provider == 'google_oauth2'
      params.delete('current_password')
      resource.password = params['password']

      resource.update_without_password(params)
    else
      resource.update_with_password(params)
    end
  end
  ```

34. Everything working on the current build except for User#edit
