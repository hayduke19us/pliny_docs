## Commands

#### Generators

				pliny-generate [model, migration, mediator, serializer]

The pliny-generate commands can be found in */lib/pliny/commands/generator.rb*.
They are methods in a Thor subclass which can be investigated
[Here](https://github.com/erikhuda/thor). Thor is a command line tool builder.

*Models* 

				pliny-generate 'user'


This creates 3 new files

1. *./lib/models/user.rb* 

2. *./db/migration/#{Time.now}_create_users*

3. *./spec/models/user_spec.rb*

A Sequel::Model subclass named by the argument at generation or by file and
class. More about Sequel::Model [ here ]( https://github.com/jeremyevans/sequel )

*./lib/models/user.rb*

				class User < Sequel::Model 
				
				  plugins: :timestamps, update_on_create: true
				
				end

A Sequel#migration block. More on Sequel migrations can be found
[here](http://sequel.jeremyevans.net/rdoc/files/doc/migration_rdoc.html).

*./db/migration/#{Time.now}_create_users*

				Sequel.migration do 
				
				  change do

					create_table 'user' do 
					
						uuid :id, default: Sequel.function(:uuid_generate_v4), primary_key: true
						timestamptz  :created_at, default: Sequel.function(:now), null: false
						timestamptz  :updated_at, default: Sequel.function(:now), null: false 
						
					end 
					
				  end
				  
				end

Last but not least a spec. 

*./spec/models/user_spec.rb* 

				require 'spec_helper'

				describe User do

				  before do

					@u = User.create

				  end

				  it 'can be created' do 
				  
					assert User.find id: @u.id
					
				  end

				end
				

#### Rake

As always you can see a list of all your rake task with `rake -T` which looks in
your *Rakefile* for task that have been created. In pliny's vanilla set up your
Rake file is loading each rake task in *./lib/tasks*. All of the Database
transactions are performed by `Sequel`. More info about sequels database actions
can be found
[here](http://sequel.jeremyevans.net/rdoc/files/doc/opening_databases_rdoc.html)
Here is a brief overview of the rake task that come with Pliny.

`rake db:create`

It gets the database urls looking first in `ENV['DATABSE_URL']`then if nil looks
in *.env* and *.env.test*. Then it connects to the database assuming it is at
"postgres:/postgres" and creates the urls found in the env files by running raw
sequel. 

`rake db:drop` 

Drops the databases in env files if the databases exist.

`rake db:rollback`

Looks for all of the :schema_migrations in your database and finds the target
for rollback. Uses `Sequel::Migrator.apply()`. More on the Migrator can be found
[here](http://sequel.jeremyevans.net/rdoc/classes/Sequel/Migrator.html).

`rake db:migrate`

Runs all of your migrations in *./db/migrate*

`rake db:schema:load` 

Loads the *./db/schema.sql* into the connected database if there is a schema.

`rake db:schema:dump`

This dumps all of the databases `:schema_migrations` into the *schema.sql*. The
COMMENT ON's are all gsubed with whitespace. 

`rake db:schema merge`

Runs `rake db:setup` and `rake db:dump` dumping all of your migrations in the
*./db/schema.sql.* file and then deletes all of the migrations. 

`rake db:seed`

Connects to all databases defined in env and loads the *./db/seeds/rb* file.

`rake db:setup` 

Intended to be used in development settings. Runs the following rake tasks in
succession.

* `drop`

* `create`

* `schema:load`

* `migrate`

* `seed`

`rake schema`

This attempts to rebuild your json schema with `gem prmd`.

		prmd combine --meta meta.json schemata/ > schema.json 

Make sure you have `pliny-generate schema users` if you intend on validating
with a schema. 

