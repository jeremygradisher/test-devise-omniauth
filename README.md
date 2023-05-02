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

Everything is set for tutorial here:<br>
https://youtu.be/CnZnwV38cjo

8. 