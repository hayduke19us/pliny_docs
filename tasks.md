## Rake Tasks 

#### Rakefile 

*$ROOT/Rakefile*

The Rakefile by default has two jobs.

1. `require 'pliny/tasks'`

2. Loads each tasks created in *lib/tasks/anything.rake* 

3. Set a default task `:spec` which of course runs your tests.

#### Custom Tasks

To create a custom tasks create a file in *lib/tasks* with an extension of
.rake. Follow the task guide lines [ Here ](https://github.com/ruby/rake). Make
sure you keep your *Rakefile* pristine and `require pliny/tasks` must always
precede your code. When the tasks are required we get necessary classes and
methods.

* database_urls
	
* Pliny::DbSupport

* Sequel 

* Pliny::Utils
	* self#parse_env

**Connecting to the Database**

				Sequel.connect(database_url)

				or

				database_urls.cycle(1) { |url| Sequel.connect(url) }

A good example of how these methods are used can be found in
*lib/pliny/tasks/db.rake#131*. The method `#databse_urls` first calls
`ENV['DATABASE_URL']` and if not found parses your *.env* and *.env.test* files.
It returns a mapped enumerable of [database_urls]. They can be iterated thru as
follows:

				database_urls.each do |database_url| 

				  Pliny::DbSupport.run(database_url, $stdout) do |helper|  
				  
					users = helper[:users] 

					puts users.count
				  
				  end
				  
				end

This block uses *pliny/lib/db_support.rb* which is a class named
Pliny::DbSupport. The `#run` method connects to the database by creating a new
instance with `#new`. It `yields` the instance and then sends the instance
`#disconnect` when the block is completed. The details can be seen in the file.
The summary is `Db::Support` has a lot of helpful instance methods.

				db_support = Db::Support.new(database_url)

				  .exist?('name_of_database') 

					=> boolean

				  .create(name) 
				
					=> < new database >

				  .disconnect

				  .db 

					 => < database connection >

You can also connect to any database you please using Sequel with
`Sequel.connect(database_url)`. And then of course you have full control.

				db = Sequel.connect(database_url) 
				
				users = db[:users]

				  => <users dataset 'SELECT * FROM users'>

				users.all

				  ==>
	

To Choose the last database 

				Sequel::Databases.last 

**Accessing Models**

The important thing is to make sure your database is connected before sending a
message to a model. If not it will throw a verbose error.

				database_urls.cycle(1) { |url| Sequel.connect url }

				or

				Sequel.connect('database_url')

				or 

				Sequel.connect(database_urls.last)


				require './lib/models/user'

				User.create(name: 'Frank', age: 60, employed: false)


**Getting Environment Variables**

I suggest wrapping the parsing of  *.env* and *.env.test* files in a method or
grabbing all of them from the get go.

				def parse_env 
				
				  File.exist?(file) ? Pliny::Utils.parse_env(file) : nil
			    	
				end

				def env type='development' 
				
				  env = lambda { 
				    
					case type

					  when 'develoment' 
					  
						parse_env(.env)
					  
					  when 'test' 
					  
						parse_env(.env.test)

					end
				  
				  }
					
					ENV['PLINY_ENV'] ? ENV : env.call
				
				end

Then you can use the `env['SOMETHING']` any time you want. The conditional
looking for ENV['RACK_ENV'] is just checking if it is launched on Heroku where
you have access to `heroku config`, the ENV.  Or you can get an individual env
like below.


				def get_env env
				
				  ENV[env] ? ENV[env] : parse_env.compact

				end 

				def parse_env 

				  %(.env .env.test).map do |file|
                     
					path = './' + file

					if File.exist? path

					  Pliny::Utils.parse_env(path)["env"]

					else 
					
					  nil 
					
					end 
					
				  end
					
				end  
				  

This does the same thing as `database_urls`. Here it is refactored into two
methods. The first method looking for the environment returning that variable if
found. The second method is a composite for the first and parses the environment
files. 

