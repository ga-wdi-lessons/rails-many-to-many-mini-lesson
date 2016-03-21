# Many-to-many Relationships (using has_many :through) in Rails

## Learning Objectives

* Differentiate between a one-to-many and a many-to-many relationship
* Describe the role of a join table in a many-to-many relationship
* Create a Model to represent a join table
* Use `has_many through` to connect two models via a join model in Rails
* Use a many-to-many relationship to implement a feature in a Rails app

### Screencast of this lesson

[Screencast on Youtube](https://www.youtube.com/watch?v=JxW8lJzLhxI)

## Framing: Review One-to-Many Relationships (5 min)

So far in Class we have been working with One to Many Relationships:

* In Scribble, we had `Posts` and `Comments`.
* During our Hogwarts Lab, we had `Houses` and `Students`
* In Tunr, we had `Artists` and `Songs`

However, using the Tunr as an example, let's consider that a lot of music is composed collaboratively and we may have songs that belong to **multiple Artists**.

We need a way to represent this type of many to many relationship!

## Many-to-Many Relationships (5 min)

Many-to-many relationships are fairly common in apps. Examples might include:

* posts have many categories, categories have many posts
* groups have many photos, photos have many groups
* playlists have many songs, songs can be in many playlists

Unlike 1-to-many relationships, we can't just add a foreign key to one of the
two tables (the `belongs_to` table) to store these associations. We'd run into a
problem where the column would need to store multiple ids, rather than just the
one id in a one-to-many relationship.

Instead, we must create a new table, a *join table* to store these associations.

## Join Tables & Models (5 min)

### Join Tables

A `join table` is a separate table in our DB whose job is to store information
about the relationship between our two models of the many-to-many. For each
many-to-many relationship, we'll need one join table.

At a minimum, each join table should have `two foreign_key columns`, for the tables
it's joining. e.g. for PlaylistEntries, we should have a `song_id` column and
a `playlist_id` column.

We can add additional columns as needed to store additional information about
the relationship. For example, we may choose to add an `order` column which
stores an integer representing what order that song appears on the playlist.
(e.g. a song may be first on one playlist, but 10th on another... that info
would be stored on the join table.)

### Join Models

In rails, we should always create a model to represent our join table. The name
can technically be anything we want, but really the model name should be as
descriptive as possible, and indicate that it represents an *association*.

### Think/Pair/Share - 3/2 (5 min)

Turn to your Neighbor and discuss the following for each example:

1. Is this relationship a Many-to-Many or One-to-Many?
2. If a Many-to-Many, what might we call the Join Model/Join Table?

Examples:

* Users & Events
* Users & Courses
* Events & Locations  
* Photos and Groups

>Many-to-Many Examples:
* To join posts to categories, we might have a `CategoryEntry` or `Categorization` model
* To join songs to playlists, we might have a `PlaylistEntry` model
* To join users and events, we might create an `Attendance` model
* To join users and courses, we might create an `Registration` model
* To join photos to groups, we might have a `GroupMembership` model

### Generating the Model / Migration (5 min)

We generate the model just like any other!

If we specify the attributes (i.e.
columns on the command line, Rails will automatically generate the correct
migration for us.

```bash
$ rails g model Attendance user:references event:references num_guests:integer
```

This will generate an Attendance model, with `user_id`, `event_id` and
`num_guest` columns.

### You Do: Create the PlaylistEntry Model (5 min)

Instructions:

***Make sure to checkout the playlists-starter branch locally***

1. Fork and Clone the Tunr Repo:[Tunr Playlists Starter](https://github.com/ga-dc/tunr_rails)
2. `$ git checkout playlists-starter`
3. `$ bundle install`
4. `$ rake db:drop`
5. `$ rake db:create`
6. `$ rake db:migrate`


Create a model / migration for the `PlaylistEntry` model. It should have `song_id`,
`playlist_id`, and `order` columns.

### Adding the AR Relationships (5 min)

Once we create our join model, we need to update our other models to indicate
the associations between them.

For example, in our Users/Events example, we should have this:

```ruby
# models/attendance.rb
class Attendance < ActiveRecord::Base
  belongs_to :event
  belongs_to :user
end

# models/event.rb
class Event < ActiveRecord::Base
  has_many :attendances
  has_many :users, through: :attendances
end

# models/user.rb
class User < ActiveRecord::Base
  has_many :attendances
  has_many :events, through: :attendances
end
```

### You-Do Exercise: Update our Models (5 min)

Update the Song, Playlist, and Playlist Entry models to ensure we have the
correct associations.

### Testing our Association (5 min)

It's a good idea to use the `rails console` to test creating our associations.

Here's an example of using the association of users / events:

```ruby
bob = User.create({username: "Bob", age: 25})
carly = User.create({username: "Carly", age: 28})

prom = Event.create({title: "Under the Sea: 2015 Prom", location: "Greenville High School"})
after_party = Event.create({title: "Eve's Awesome After-party", location: "Super Secret!" })
brunch = Event.create({title: "BRUNCH!", location: "IHOP" })

# We can create the association directly
bob_going_to_the_prom = Attendance.create(user: bob, event: prom, num_guests: 1)

# Or using helper functionality:
bob.attendances.create(event: after_party, num_guests: 0) # bob's going alone :(

# or the other way
brunch.attendances.create(user: carly, num_guests: 10)
prom.attendances.create(user: carly, num_guests: 1)

# to see who's going to an event:
prom.users
after_party.users
brunch.users


# to see a user's events
bob.events
carly.events

# to delete an association
Attendance.find_by(user: bob, event: prom).destroy # will only destroy the first one that matches

Attendance.where(user: bob, event: prom).destroy_all # will destroy all that match
prom.attendances.where(user: bob).destroy_all
```

### You-Do Exercise: Update Playlists Controller (10 min)

We have already defined the Routes and Views to add and remove songs from a Playlist.

Update the associated `add_song` and `remove_song` actions in the playlists controller to
add and remove songs from the playlist. Look at the `playlists/show.html.erb`
view to see how we route to these actions.

### Garnet Example (5 min)

[Garnet Many-to-Many Example](https://github.com/ga-dc/garnet/blob/master/app/models/tagging.rb)

* `Membership` and `Tags`, through `Tagging`.
>Tagging is our model representing our joined table between Membership and Tags

* `Cohort` and `User`, through `Membership`
>Membership is our model representing our joined table between User and Tags

## Closing

### Additional Resources
* [Andy's Blog Post](http://andrewsunglaekim.github.io/many-actives-to-many-records/)
* [Rails Guides - Has Many Through](http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association)

### References

* [Tunr Playlists Starter](https://github.com/ga-dc/tunr_rails/tree/playlists-starter)
* [Tunr Playlists Solution](https://github.com/ga-dc/tunr_rails/tree/playlists-solution)
