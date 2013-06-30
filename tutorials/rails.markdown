####Let’s install some Rails!

#####Windows:

Download [RailsInstaller][1] and run it. Click through the installer using the default options.
Installation of Git and SSH-keys is required if you want to put your app online with Heroku.

#####OS X:

Download the RailsInstaller for your version of OS X:
[10.7 and 10.8][2] 
[10.6][3] 

Double click the downloaded file and it will unpack it into the current 
directory. It will open a readme file with ‘Rails Installer OS X’ at the top. Please ignore the instructions in this file.

#####Linux:
For Ubuntu:
bash < <(curl -s https://raw.github.com/railsgirls/installation-scripts/master/rails-install-ubuntu.sh)

For Fedora:
bash < <(curl -s https://raw.github.com/railsgirls/installation-scripts/master/rails-install-fedora.sh)

####Or:

... if you already have Ruby installed:
ruby -v 

To install Rails, use the gem install command provided by RubyGems:
gem install rails

To verify that you have everything installed correctly:

rails --version

If it says something like “Rails 4.0.0”, you are ready to continue.

####Creating a Map

**Starting a new Rails project:**
mkdir projects
cd projects
rails new mapp
cd mapp
rails s

Open http://localhost:3000 in browser.
CTRL-C to exit the server in Terminal/Command Prompt.  


**Rails’ scaffolds generate a starting point that allows us to list, add, remove, edit, and view things.** 

rails generate scaffold attendee name:string twitter_handle:string bio:text address:text picture:string
rake db:migrate
rails s

**We’ll use Twitter’s Bootstrap project to give us nicer default styles really easily.** 
Open app/views/layouts/application.html.erb and add on top of

<pre><code><%= stylesheet_link_tag “application” %></pre></code>

the line

<pre><code><link rel="stylesheet" href="http://railsgirls.com/assets/bootstrap.css"></pre></code>

and swap

<pre><code><%= yield %></pre></code>

for
	
<pre><code><div class="container"><%= yield %></div></pre></code>

Let’s also add topbar and footer to the layout and style those and the 
attendees table. To the application.html.erb under <pre><code><body></pre></code> add:

<pre><code><div class="navbar navbar-fixed-top">
    <div class="navbar-inner">
        <div class="container"><a class="brand" href=“/”>The mapp</a><ul class="nav">
              <li class="active"><a href="/attendees">attendees</a></li>
            </ul>
        </div>
    </div>
</div></pre></code>

before <pre><code></body></pre></code> add:

<pre><code><footer>
    <div class=“container”>5 talen in 5 dagen</div>
</footer></pre></code>

Open app/assets/stylesheets/application.css and add to the bottom:

<pre><code>#logo {
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
td.picture img { width: 140px; }</pre></code>

**We need to install additional library to add image processing.** 
Open Gemfile in the project and add:

<pre><code>gem 'carrierwave'</pre></code>

under the line:

<pre><code>gem 'sqlite3'</pre></code>

In the Terminal/Command Prompt run bundle. Restart the Rails server process in the Terminal. Then run: 
rails generate uploader Picture

Open app/models/attendee.rb and add:

<pre><code>mount_uploader :picture, PictureUploader</pre></code>

under the line:
<pre><code>class Attendee < ActiveRecord::Base</pre></code>

Open app/views/attendees/_form.html.erb and change

<pre><code><%= f.text_field :picture %></pre></code>

to

<pre><code><%= f.file_field :picture %></pre></code>

and

<pre><code><%= form_for(@attendee) do |f| %></pre></code>

to

<pre><code><%= form_for(@attendee, :html => {:multipart => true}) do |f| %></pre></code>

The view doesn’t look nice, it only shows a path to the file, so let’s fix it.
Open app/views/attendees/show.html.erb and change

<pre><code><%= @attendee.picture %></pre></code>

to

<pre><code><%= image_tag(@attendee.picture_url, :width => 600) if @attendee.picture.present? %></pre></code>

**If you try to open http://localhost:3000 it still shows the default page.** 

On OS X and Linux, in the Terminal run:

rm public/index.html

In Windows, in the Command Prompt run:

del public\index.html

Then open config/routes.rb and add the following after the first line:
<pre><code>root :to => redirect("/attendees")</pre></code>

**Adding Geolocator and Google Maps**

Open Gemfile in the project and add

<pre><code>gem 'geocoder'</pre></code>

and

<pre><code>gem 'gmaps4rails'</pre></code>

under the line

<pre><code>gem 'carrierwave'</pre></code>

in the Terminal/Command Prompt run
bundle

and then

rails generate migration AddLatitudeAndLongitudeToAttendee latitude:float longitude:float
rake db:migrate

Open app/models/attendee.rb and add

<pre><code>geocoded_by :address</pre></code>
<pre><code>after_validation :geocode</pre></code>

after

<pre><code>mount_uploader :picture, PictureUploader</pre></code>

In the Terminal/Command Prompt run

<pre><code>rails generate gmaps4rails:install</pre></code>

Go back to app/models/attendee.rb and add
<pre><code>acts_as_gmappable :process_geocoding => false</pre></code>

after
<pre><code>after_validation :geocode</pre></code>

Open app/controllers/attendees_controller.rb and add

<pre><code>@pins = @attendees.to_gmaps4rails</pre></code>

in the index method, after

<pre><code>@attendees = Attendee.all</pre></code>

Open app/views/attendees/index.html.erb and add

<pre><code><br/>
<%= gmaps4rails(@pins) %>
<%= yield :scripts %></pre></code>

after the table.

####Putting it online with Heroku:

[Sign up for a Heroku account][4], if you don’t already have one.
[Install the Heroku Toolbelt][5] for your development operating system.
heroku login
Press enter at the prompt to upload your existing ssh key or create a new one, used for pushing code later on.

<pre><code>Add the following in the Gemfile:
group :development do
  gem ‘sqlite3’
end</pre></code>

Run:
bundle install --without production

Version control:
git init
git add .
git commit -m "initial commit"

App creation:
heroku create
git push heroku master
heroku run rake db:migrate
heroku open




[1]: http://rubyforge.org/frs/download.php/76862/railsinstaller-2.2.1.exe
[2]: http://railsinstaller.s3.amazonaws.com/RailsInstaller-1.0.4-osx-10.7.app.tgz
[3]: http://railsinstaller.s3.amazonaws.com/RailsInstaller-1.0.4-osx-10.6.app.tgz
[4]: https://id.heroku.com/signup/devcenter
[5]: https://toolbelt.heroku.com/
