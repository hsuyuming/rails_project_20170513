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