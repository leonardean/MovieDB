MovieDB
=======

A database design for movie website.
##Database ER Diagram
![er](http://www.mftp.info/20140202/1392173405x1927179616.png)
##Tables
####Users
Users have many attributes, and this table stores the basic, atomic user data. All the user information that consists of a single instance is stored in a single table to avoid 1:1 relationship, it did not make sense to split this up.

Because oracle does not have a data type for Boolean, the Locked/Logged status of a user is stored as a char with the value ‘Y’ or ‘N’.

The table contains: userid as primary key, username, password, firstname,lastname, email,creationDate, securityLevel, ip, paidUntilDate,lastLogin, coins, likes, avatar,homepage.

####Movies
Our database centres around the Movie table. Most of attributes in the table were constant: movie id, name, description, studio, average rating, overall rank, image url, website url. To include more specific information, age rating, director, and running time were added to the table.

Avg rating is redundant because it could be obtained through a query, but in this way we avoid to calculate it every time we have to show it.

####Genre
In order to allow one movie to belong to multiple genres and avoid non-atomic values, a table was created specifically for this information. The table contains: movie id, genre, and the rank in that genre. As a foreign key, the movie id in this table refers to the movie id in the table of movies. Integrity was added to let the movie id change according to its reference.

####Actors
We created this table to store informations about all the actors in the database, we store their roles in the table “Casting” to avoid a many to many relationship. This table stores also directors, as an actor could be a director and the opposite. We can retrieve the directors list using more complex queries as the actorID of the director is stored in the Movies table; this avoids redundancy and an useless attribute for a big percentage of the actors.

####Casting
Because of the natural of movies, a table for unordered casting list was created. The table contains a movie id, and an actor id with movie id referring to table of movies.

####Collection
This table contains information of all the collection items of a user instead of separating into three tables. Two assumption were made in here:

1. The comment and amount of likes of a movie in collection and those in personal lists are to be different.

2. When a user subscribes a list, the list here means user created list instead of the list for collection.

With the assumptions, the table contains a user id with reference, a movie id with reference, a rating to the movie, whether the users owns/wishes/watched the movie. Again the technique used to represent Boolean was the same as mentioned before.

####Friend
The table for friends was used to store an unordered list of friends with keys: a user id and a friend id both referencing to the table of users.

####Lists
This table is used to store user created lists. As they will need to be subscribed, each list has to have an identifier. The table contains: a list id as primary key, user id with reference, list name, description, an image url, amount of likes, and a tip-box.

####List Item
The list item maps between user created lists elements and movies in the lists. The table contains a list id with reference, a movie id with reference, position in the list, comments, amount of likes, tip-box, and an image url.

####Post
We choose to merge thread and posts because of these reasons:

* They shared mostly the same attributes, a thread being a list of posts
* When a new thread is created an opening post is created too.

We can obtain the posts of a thread making a query that returns all the posts with a specific threadID. the posts can be sorted using postDate and that maintains the discussion structure. All the Thread info can be obtained from the first post.

The post contains: post id, user name,title, post date, text, amount of likes, tip-box , and whether the post is moderated or locked.

####Subscription
With movies, lists, and threads being able to be referenced and attached, the table for subscription was created to store user subscribed items. The table contains: user id with reference, movie id with reference, list id with reference, and thread id with reference.

We should make sure that only one of movieid, listid and threadid is activated for every row.

####Thumbs
Without this table users will be able to give infinite thumbs to the same items, so we created this table storing only userid and objectid as primary foreign key.

####Ratings
We had to take track of ratings to calculate the average, and allow user to change it. This table has userId and movieId as primary foreign key and a value that represent the rating value.

####CoinTransactions
We assumed that tracking coin transaction would be an useful feature, so we create this table with ID as primary key because is it possible to have multiple transactions between the same users and giver and receiver as userId from Users Table a reason and an amount of the coins moved.

####User-suggested movies
As part of the requested functionality, users can suggest additions to the movie database. Information they enter would be stored in this table (VALIDATEMOVIES) until a moderator could check it for validity.

##Normalizations
To convert to a 1NF relation, any non-atomic values (for example collections, lists, and posts) had to be split up. The solution was to create tables as containers to map the values to their dependencies. The tables were naturally in 3NF because the dependency in each table was very straightforward: all attributes were dependent on ID. So there were no partial dependencies and transitive dependencies.

The tables “Casting” and “List Item” were created to remove many to many relationships creating a new table and two one to many relationships. We removed redundancies and avoided creating subclasses, for example Top10 and threads are not separate entities but are part of Lists and Posts respectively.

##Primary Keys
```
Users: userId
Movies: movieId
Genre: genre, movieId
Actors: actorId
Collection: userId, movieId
Casting: movieId, actorId
Friend: userId, friendId
Lists: listId
ListItem: listId, movieId
Post: postId
Subscription: userId, movieId, listId, postId Ratings: movieId, userId
Thumbs: movieId, userId
Coin Transactions: id
```

##Relationships
```
Create [1-M] (User - Lists)
Contain [1-M](Lists - ListItem) 
Represents [M-1] (ListItem - Movies) 
Have [1-M] (Movies - Casting) 
Starred [1-M] (Actors - Casting) 
BelongTo [1-M] (Movies - Genre) 
IsOwned [1-M] (Movies - Collection) 
Own [1-M] (Users - Collection) 
Friendship [1-M] (Users - Friend) 
Rated [1-M] (Users - Ratings)
Given [1-M] (Users - Thumbs)
Created [1-M] (Users - Post)
Sent [1-M] (Users - CoinTransactions) 
SubscribedTo [1-M] (Users - SubScription)

```
##Functionality
The system can perform the following functionalities:
* Find all the users who have a particular item in their collection
* Automatically update an item’s average rating whenever a user adds or updates their rating for that item
* Change the database so that an item’s average ratings only appears if it has been rated by 10 or more users
* At Christmas, give a present of 5 coins to all users
* CREATE LIST FOR EACH GENRE
* Allow the user to buy a year’s subscription with 500 coins of virtual cash
* Check new usernames against a list of obscene or offensive terms (if possible check for sub-strings as well as the whole name). If such a username exists, lock the account
* Allow users to send private messages to one another.
* select the best three movies Woody Allen has directed after 1999-07-24
* elect all movies produced by the studio 'X' but without 'Mark Ruffalo' being in the casting list
* select all actors who has co-operated with director 'Paul Thomas Anderson'
* ADD TITLE TO THE PROFILE
* Users are able to submit new items to the database
* Suggest another user as a friend if their ratings are similar to the user’s
* Automatically suggest an item which a user might be interested in.


