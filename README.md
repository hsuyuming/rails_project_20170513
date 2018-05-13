# Create table relationship

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


```ruby
#rails c
# all will return a list (ActiveRecord::Relation)
Book.all
```
```sql
#<ActiveRecord::Relation [#<Book id: 1, title: "The Force", sales: 500, created_at: "2018-05-13 09:52:16", updated_at: "2018-05-13 09:52:16", author_id: 2, genre_id: 2>, #<Book id: 2, title: "Britney Speares: An Anthology", sales: 950, created_at: "2018-05-13 09:52:16", updated_at: "2018-05-13 09:52:16", author_id: 1, genre_id: 3>, #<Book id: 3, title: "Only One Direction", sales: 45, created_at: "2018-05-13 09:52:16", updated_at: "2018-05-13 09:52:16", author_id: 1, genre_id: 3>, #<Book id: 4, title: "DIY Deathstar", sales: 1200, created_at: "2018-05-13 09:52:16", updated_at: "2018-05-13 09:52:16", author_id: 1, genre_id: 2>]>
```
```ruby
# if you want, you can use find_by_sql replace to use all
Book.all
Book.find_by_sql("SELECT books.* FROM books")
```

### Compare where & find_by_(param)

> where => will return a collection no matter just one element or not.
> find_by_(param) => always return one single result.

```ruby
Book.where(title:"The Force")
Book.where(title:"The Force").author
```
```sql
NoMethodError (undefined method `author' for #<Book::ActiveRecord_Relation:0x00005592fe020878>)
Book.where(title:"The Force").first.author
```

```ruby
Book.find_by_title("The Force")
```

```sql
  Book Load (3.0ms)  SELECT  "books".* FROM "books" WHERE "books"."title" = $1 LIMIT $2  [["title", "The Force"], ["LIMIT", 1]]
=> #<Book id: 1, title: "The Force", sales: 500, created_at: "2018-05-13 09:52:16", updated_at: "2018-05-13 09:52:16", author_id: 2, genre_id: 2>
```

```ruby
Book.where(title:"The Force").class
#=> Book::ActiveRecord_Relation
Book.find_by_title("The Force").class
#=> Book(id: integer, title: string, sales: integer, created_at: datetime, updated_at: datetime, author_id: integer, genre_id: integer)
```

### Check any record exist or not 
> use any
```ruby
leia = Author.find_by_name("Leia")
leia.book.any?
=>Book Exists (1.9ms)  SELECT  1 AS one FROM "books" WHERE "books"."author_id" = $1 LIMIT $2  [["author_id", 3], ["LIMIT", 1]] return false

vader = Author.find_by_name("Vader")
vader.book.any?
Book Exists (2.2ms)  SELECT  1 AS one FROM "books" WHERE "books"."author_id" = $1 LIMIT $2  [["author_id", 1], ["LIMIT", 1]] return true
```
>Book.count
```sql
SELECT COUNT(*) FROM "books"
```

# How many sales author made
> Book.where(name:"Vader")
```sql
SELECT COUNT(*) FROM "books"
```

>使用連環計
```ruby
#sum(:欄位)
Author.find_by_name("Vader").book.sum(:sales)
```
#result
```sql
SELECT  "authors".* FROM "authors" WHERE "authors"."name" = $1 LIMIT $2  [["name", "Vader"], ["LIMIT", 1]]
   (2.1ms)  SELECT SUM("books"."sales") FROM "books" WHERE "books"."author_id" = $1  [["author_id", 1]]
=>2195
```

```ruby
#average
Book.average(:sales) 
=>0.67375e3
Book.average(:sales).to_f
=>673.75
```

```ruby
#order => return collection
Book.order('sales DESC')
=> #<ActiveRecord::Relation [#<Book id: 4, title: "DIY Deathstar", sales: 1200, created_at: "2018-05-13 09:52:16", updated_at: "2018-05-13 09:52:16", author_id: 1, genre_id: 2>, #<Book id: 2, title: "Britney Speares: An Anthology", sales: 950, created_at: "2018-05-13 09:52:16", updated_at: "2018-05-13 09:52:16", author_id: 1, genre_id: 3>, #<Book id: 1, title: "The Force", sales: 500, created_at: "2018-05-13 09:52:16", updated_at: "2018-05-13 09:52:16", author_id: 2, genre_id: 2>, #<Book id: 3, title: "Only One Direction", sales: 45, created_at: "2018-05-13 09:52:16", updated_at: "2018-05-13 09:52:16", author_id: 1, genre_id: 3>]>
Book.order('sales DESC').first

Book.order('sales DESC').first.author.name
=>Book Load (2.5ms)  SELECT  "books".* FROM "books" ORDER BY sales DESC LIMIT $1  [["LIMIT", 1]]
  Author Load (1.3ms)  SELECT  "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]] Return "Vader"
```

#Ability to run multiple queries and be able to connect tables
> Question: Which genres that each one of our authors has written?

```ruby
#model:Author.rb
class Author < ApplicationRecord
    has_many :books
    has_many :genres, through: :books
end
#model:Gerne.rb
class Genre < ApplicationRecord
    has_many :books
    has_many :authors, through: :books
end

#rails c
reload!
Author.find_by_name("Vader").genres
  Author Load (2.3ms)  SELECT  "authors".* FROM "authors" WHERE "authors"."name" = $1 LIMIT $2  [["name", "Vader"], ["LIMIT", 1]]
  Genre Load (3.3ms)  SELECT  "genres".* FROM "genres" INNER JOIN "books" ON "genres"."id" = "books"."genre_id" WHERE "books"."author_id" = $1 LIMIT $2  [["author_id", 1], ["LIMIT", 11]]
#<ActiveRecord::Associations::CollectionProxy [#<Genre id: 2, name: "Non-Fiction", created_at: "2018-05-13 09:52:16", updated_at: "2018-05-13 09:52:16">, #<Genre id: 3, name: "Biographies", created_at: "2018-05-13 09:52:16", updated_at: "2018-05-13 09:52:16">, #<Genre id: 3, name: "Biographies", created_at: "2018-05-13 09:52:16", updated_at: "2018-05-13 09:52:16">]>
```

# Plunk
> It comes to being able to select pick and choose the kind of items that you want.
> can allow you don't need to iterator each object.

```ruby
Book.all => foreach get name
Book.pluck(:title)

Author.ids #shortcut and handy way
=> [1, 2, 3]
```
```sql
SELECT "books"."title" FROM "books"
=>["The Force", "Britney Speares: An Anthology", "Only One Direction", "DIY Deathstar"]
```


# Performace tunning

> contoller 

```ruby
def index
  @books = Book.all
end

#change to 
def index
  @books = Book.includes(:author, :genre)
end

web_1  |   Book Load (4.6ms)  SELECT "books".* FROM "books"
web_1  |   Author Load (1.4ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" IN (2, 1)
web_1  |   Genre Load (2.2ms)  SELECT "genres".* FROM "genres" WHERE "genres"."id" IN (2, 3)
```
reduces hit db's count




