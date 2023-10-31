* Several frameworks offer mass-assignment features to lessen the workload for developers. 
* The feature is often used without a whitelist for protecting the fields from users input
## Examples:
### Ruby on rails
* Web application framework that is vulnerable to this type of attack.
```ruby
class User < ActiveRecord::Base
  attr_accessible :username, :email
end
```
* This specifies that only the username and email attributes are allowed to be mass-assigned.
## Exploiting
* Attempt to fuzz on burp suite. 
