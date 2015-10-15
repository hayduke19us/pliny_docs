## Rake Tasks 

#### Rakefile 

*$ROOT/Rakefile.rb*

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

* database_url
	
* Pliny::DbSupport

* Sequel 

* Pliny::Utils
	* self#parse.env

**Connecting to the Database**

				Sequel.connect(database_url)

				or

				database_urls.cycle { |url| Sequel.connect(url) }

A way to do this while keeping things production ready can be found in Pliny's
*lib/pliny/db.rake#137*. The method `#databse_urls` here first looks for your
ENV['DATABASE_URL'] and if not found parses your *.env* and *.env.test* files.
It returns an enumerable of [database_urls]. They can be iterated thru as
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

				database_urls.cycle { |url| Sequel.connect url }

				or

				Sequel.connect('database_url')

				or 

				Sequel.connect(database_urls.last)

				or

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
looking for ENV['RACK_ENV'] is just checking if it is launched on heroku where
you have access to `heroku config`, the ENV.  Or you can get an individual env
like below.


				def get_env env
				
					if ENV[env]

					  ENV[VARIABLE] 
					  
					else

					  %(.env .env.test).map {
					
						env_path = "./{env_path}" 
			  
						If File.exists?(env_path)

						  Pliny::Utils.parse_env(env_path)["ENV_VARIABLE"] 

						else 
						
						  nil 
						
						 end

					   }  
					  
					end
					
				  end
