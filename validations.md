## Model and Database Validations

#### Model 

Sequel takes care of our model validations and you can find detailed
instructions
[Here](http://sequel.jeremyevans.net/rdoc/files/doc/validations_rdoc.html).

A simple name validation would be as follows in a User model

				def validate 
			
				  super 

				  erros.add(:name, "name cannot be blank", if !name ||
				  name.empty)

				end

We are overwriting the Sequel method. This gives you the functional method of
`valid?`. Do not write over `#valid`, but instead conduct overrides in the
`#validate` method.

You can also use `validation_helpers` 

				class User

				  plugin :validation_helpers

				  def validate 
				   
					validate_presence :name 

				  end

#### Database 

For strict NOT NULL columns state the following in the migrations also handled
by Sequel and can be referenced
[Here](http://sequel.jeremyevans.net/rdoc/files/doc/migration_rdoc.html).

				Sequel.migration do 

				  change do 

					create_table :users do

					  String  :name, null: false	
					  
					end

				  end

				end
