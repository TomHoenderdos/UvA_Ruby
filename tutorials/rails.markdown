####Let’s install some Rails!

#####Windows:

Download [RailsInstaller][1] and run it. Click through the installer using the default options.  
Installation of Git and SSH-keys is required if you want to put your app online with Heroku.

#####OS X:

Download the RailsInstaller for your version of OS X:  
[10.7 and 10.8][2]  
[10.6][3]  

Double click the downloaded file and it will unpack it into the current directory.  
It will open a readme file with ‘Rails Installer OS X’ at the top. Please ignore the instructions in this file.

#####Linux:
For Ubuntu:  
```bash < <(curl -s https://raw.github.com/railsgirls/installation-scripts/master/rails-install-ubuntu.sh)```

For Fedora:  
```bash < <(curl -s https://raw.github.com/railsgirls/installation-scripts/master/rails-install-fedora.sh)```

####Or:

... if you already have Ruby installed:   
```ruby -v  ```

To install Rails, use the gem install command provided by RubyGems:  
```gem install rails  ```

To verify that you have everything installed correctly:  
```rails --version  ```

If it says something like “Rails 4.0.0”, you are ready to continue.

####Creating a Map

**Starting a new Rails project:**  
<pre>
mkdir projects  

cd projects  

rails new mapp  

cd mapp  

rails s  
</pre>

Open ```http://localhost:3000``` in browser.  
CTRL-C to exit the server in Terminal/Command Prompt.  


**Rails’ scaffolds generate a starting point that allows us to list, add, remove, edit, and view things.** 

```rails generate scaffold attendee name:string twitter_handle:string bio:text address:text picture:string  ```

```rake db:migrate ```

```rails s```

**We’ll use Twitter’s Bootstrap project to give us nicer default styles really easily.**  

Open ```app/views/layouts/application.html.erb``` and add on top of

```<%= stylesheet_link_tag "application" %>```

the line

```<link rel="stylesheet" href="http://railsgirls.com/assets/bootstrap.css">```

and swap

```<%= yield %>```

for
	
```<div class="container"><%= yield %></div>```

Let’s also add topbar and footer to the layout and style those and the 
attendees table. To the application.html.erb under ```<body>``` add:

```
<div class="navbar navbar-fixed-top">
    <div class="navbar-inner">
        <div class="container"><a class="brand" href="/">The mapp</a><ul class="nav">
              <li class="active"><a href="/attendees">attendees</a></li>
            </ul>
        </div>
    </div>
</div>
```

before ```</body>``` add:

```
<footer>
    <div class="container">5 talen in 5 dagen</div>
</footer>
```

Open ```app/assets/stylesheets/application.css``` and add to the bottom:
```
#logo {
    font-size: 20px;
    font-family: "Helvetica Neue",Helvetica,Arial,sans-serif;
    float: left;
    padding: 10px;
}
body { padding-top: 100px; }
footer { margin-top: 100px; }
table, td, th { vertical-align: middle !important; border: none !important; }
th { border-bottom: 1px solid #DDD !important; }
td.picture { width: 140px; }
td.picture img { width: 140px; }
```

**We need to install additional library to add image processing.** 
Open Gemfile in the project and add:

```gem 'carrierwave'```

under the line:

```gem 'sqlite3'```

In the Terminal/Command Prompt run bundle. Restart the Rails server process in the Terminal. Then run: 
rails generate uploader Picture

Open ```app/models/attendee.rb``` and add:

```mount_uploader :picture, PictureUploader```

under the line:
```class Attendee < ActiveRecord::Base```

Open ```app/views/attendees/_form.html.erb``` and change

```<%= f.text_field :picture %>```

to

```<%= f.file_field :picture %>```

and

```<%= form_for(@attendee) do |f| %>```

to

```<%= form_for(@attendee, :html => {:multipart => true}) do |f| %>```

The view doesn’t look nice, it only shows a path to the file, so let’s fix it.
Open app/views/attendees/show.html.erb and change

```<%= @attendee.picture %>```

to

```<%= image_tag(@attendee.picture_url, :width => 600) if @attendee.picture.present? %>```

**If you try to open http://localhost:3000 it still shows the default page.** 

On OS X and Linux, in the Terminal run:  

```rm public/index.html```

In Windows, in the Command Prompt run:  

```del public\index.html```

Then open ```config/routes.rb``` and add the following after the first line:
```root :to => redirect("/attendees")```

**Adding Geolocator and Google Maps**

Open Gemfile in the project and add  

```gem 'geocoder'```

and  

```gem 'gmaps4rails'```

under the line  

```gem 'carrierwave'```

in the Terminal/Command Prompt run  
```
bundle
```

and then  
```
rails generate migration AddLatitudeAndLongitudeToAttendee latitude:float longitude:float
rake db:migrate
```
Open ```app/models/attendee.rb``` and add  

```
geocoded_by :address
after_validation :geocode
```
after  

```mount_uploader :picture, PictureUploader``` 

In the Terminal/Command Prompt run  

```rails generate gmaps4rails:install```

Go back to ```app/models/attendee.rb``` and add  
```acts_as_gmappable :process_geocoding => false```

after  
```after_validation :geocode```

Open ```app/controllers/attendees_controller.rb``` and add

```@pins = @attendees.to_gmaps4rails```

in the index method, after  

```@attendees = Attendee.all```

Open ```app/views/attendees/index.html.erb``` and add  

```
<br/>  
<%= gmaps4rails(@pins) %>  
<%= yield :scripts %>
```

after the table.

####Putting it online with Heroku:

[Sign up for a Heroku account][4], if you don’t already have one.  
[Install the Heroku Toolbelt][5] for your development operating system.  
```heroku login```  
Press enter at the prompt to upload your existing ssh key or [create a new one][6], used for pushing code later on.  

Add the following in the Gemfile:  
```
group :development do  
  gem 'sqlite3'  
end  
```

Run:  
```bundle install --without production  ```

Version control:  
```
git init  
git add .  
git commit -m "initial commit"  
```
App creation:  
```
heroku create  
git push heroku master  
heroku run rake db:migrate  
heroku open
```


[1]: http://rubyforge.org/frs/download.php/76862/railsinstaller-2.2.1.exe
[2]: http://railsinstaller.s3.amazonaws.com/RailsInstaller-1.0.4-osx-10.7.app.tgz
[3]: http://railsinstaller.s3.amazonaws.com/RailsInstaller-1.0.4-osx-10.6.app.tgz
[4]: https://id.heroku.com/signup/devcenter
[5]: https://toolbelt.heroku.com/
[6]: https://devcenter.heroku.com/articles/keys

Rails 4:   
add ```<%= stylesheet_link_tag 'gmaps4rails' %>``` to your application layout 
