###Create table relationship

```ruby
#cmd
# xxx has many yyy , yyy belong_to xxx
docker-compose run web rails g migration add_xxx_reference_to_yyy xxx:references
docker-compose run web

rails db:migrate
```

>In the project's xxx and yyy model file

```ruby
#In xxx model file
add =>  has_many :yyy
#In yyy model file
add =>  belongs_to :xxx
```