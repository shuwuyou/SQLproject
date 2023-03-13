# SQL CRUD

An assignment to design relational database tables with particular applications in mind.

The contents of this file will be deleted and replaced with the content described in the [instructions](./instructions.md)

## part one
restaurants csv: https://github.com/dbdesign-assignments-spring2023/sql-crud-shuwuyou/blob/main/data/restaurants.csv

Create a new database called sql_workship.db and tables and import the files from my folder. I then choose the mode to display and then display all data.
```SQL
sqlite3 sql_workshop.db

CREATE TABLE restaurants (
    id INTEGER PRIMARY KEY,
    restaurant TEXT,
    category TEXT,
    price_tier TEXT,
    neighborhood TEXT,
    opening_hour TIME,
    closing_hour TIME,
    average_rating INTEGER,
    good_for_kids INTEGER
);

 CREATE TABLE reviews(
    restaurant_id PRIMARY KEY,
    restaurant TEXT,
    reviewer TEXT,
    review TEXT
    );


.mode csv
.import /Users/shuwuyou/Desktop/NYU/大三下/Database_and_implement/sql-crud-shuwuyou/data/restaurants.csv restaurants --skip 1
.headers on
.mode columns
select * from restaurants;
```

### Question 1.1 Find all cheap restaurants in a particular neighborhood (pick any neighborhood as an example).
I choose to show all cheap restaurants from Astoria, along with their ids to prevent same restaurant name.
```SQL
SELECT id, restaurant FROM restaurants WHERE neighborhood = "Astoria" AND price_tier = "cheap";
```

### Question 1.2 Find all restaurants in a particular genre (pick any genre as an example) with 3 stars or more, ordered by the number of stars in descending order.
I choose to show all restaurants that are French food with rating above 3 stars and ordered by rating in descending order. I also include the id column to prevent same restaurant name describing different restaurants.
```SQL
SELECT id, restaurant, average_rating FROM restaurants WHERE category = "France" AND average_rating >= 3 ORDER BY average_rating DESC;
```
### Question 1.3 Find all restaurants that are open now (see hint below).
open now at current time.
```SQL
SELECT id, restaurant FROM restaurants WHERE opening_hour <= strftime('%H:%M', 'now') AND closing_hour >= strftime('%H:%M', 'now');
```

### Question 1.4 Leave a review for a restaurant (pick any restaurant as an example).

I have already created a table called reviews at the beginning. Here, I fill the reviews with restaurant_id, restaurant name, reviewer's name, and review. I choose the restaurant Fix San with unique id 43 and leave a message by Jone.
```SQL
INSERT INTO reviews (restaurant_id, restaurant, reviewer, review) VALUES (43, "Fix San", "Jone", "The food is very nice. I love it.");
```

### Question 1.5 Delete all restaurants that are not good for kids.

I delete all restaurants where they are not good for kids. After deleting, there are 515 restaurants remaining.
```SQL
DELETE FROM restaurants WHERE good_for_kids = "false";
```

### Question 1.6 Find the number of restaurants in each NYC neighborhood.

I show neighborhoods and the restaurants within as num_restaurants and count the number.
```SQL
SELECT neighborhood, COUNT(restaurant) AS num_restaurants FROM restaurants GROUP BY neighborhood;
```

## part two
users csv: https://github.com/dbdesign-assignments-spring2023/sql-crud-shuwuyou/blob/main/data/users.csv
posts csv: https://github.com/dbdesign-assignments-spring2023/sql-crud-shuwuyou/blob/main/data/posts.csv

I create a new database called social_media.db and create two tables and the csv files into the system.
```SQL
sqlite3 social_media.db
CREATE TABLE users (
    user_id INTEGER PRIMARY KEY,
    email TEXT,
    passwords TEXT,
    user_name TEXT
);

CREATE TABLE posts (
    post_id INTEGER PRIMARY KEY,
    post_type TEXT,
    user_id INTEGER NOT NULL,
    receiver_id INTEGER,
    post_time DATETIME,
    visibility INTEGER,
    content TEXT
);

.mode csv
.import /Users/shuwuyou/Desktop/NYU/大三下/Database_and_implement/sql-crud-shuwuyou/data/users.csv users --skip 1;
.headers on
.mode columns
select * from users;

.mode csv
.import /Users/shuwuyou/Desktop/NYU/大三下/Database_and_implement/sql-crud-shuwuyou/data/posts.csv posts --skip 1;
.headers on 
.mode columns
select * from posts;
```

### Question 2.1 Register a new User.
insert a new user.
```SQL
INSERT INTO users (email, passwords, user_name) VALUES ("shuwuyou@nyu.edu", "helloworld", "nyushuwuyou");
```

### Question 2.2 Create a new Message sent by a particular User to a particular User (pick any two Users for example).


insert a new message from user 231 to friend with id 11 at this moment.
```SQL
INSERT INTO posts (post_type, user_id, receiver_id, post_time, visibility, content) VALUES ("message", 231, 11, datetime("now"), "TRUE", "Hello, my friend.");
```

### Question 2.3 Create a new Story by a particular User (pick any User for example).

insert a story by user 4 at this moment.
```SQL
INSERT INTO posts (post_type, user_id, receiver_id, post_time, visibility, content) VALUES ("story", 4, "", datetime("now"), "TRUE", "What a nice day.");
```

### Question 2.4 Show the 10 most recent visible Messages and Stories, in order of recency.

Show the 10 most recent visible Messages and Stories, in order of recency.
```SQL
SELECT post_id, post_type, content, post_time, visibility FROM posts WHERE visibility = "TRUE" ORDER BY ROUND((JULIANDAY("now") - JULIANDAY(post_time)) * 24) ASC LIMIT 10;
```

### Question 2.5 Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.

Show the 10 most recent visible Messages sent by a user 544 to a user 133 in order of recency
```SQL
SELECT post_id, post_type, content, user_id, receiver_id, post_time
FROM posts
WHERE post_type = "message"
  AND user_id = 544
  AND receiver_id = 133
ORDER BY ROUND(JULIANDAY('now') - JULIANDAY(post_time))*24 ASC LIMIT 10;
```
### Question 2.6 Make all Stories that are more than 24 hours old invisible.

Make all Stories that are more than 24 hours old invisible.
```SQL
UPDATE posts SET visibility = "FALSE" WHERE post_type = "story" AND ROUND((JULIANDAY("now") - JULIANDAY(posts.post_time))*24) >= 24;
```

### Question 2.7 Show all invisible Messages and Stories, in order of recency.

Show all invisible Messages and Stories, in order of recency.
```SQL
SELECT post_id, post_type, content, post_time, visibility
FROM posts
WHERE visibility = "FALSE"
ORDER BY (ROUND(JULIANDAY("now") - JULIANDAY(post_time))*24) ASC;
```

### Question 2.8 Show the number of posts by each User.

Show the number of posts by each user. I order them from the most posts to the fewest.
```SQL
SELECT users.user_name, COUNT(posts.post_id) AS num_posts
FROM users
LEFT JOIN posts ON users.user_id = posts.user_id
GROUP BY users.user_id
ORDER BY num_posts DESC;
```

### Question 2.9 Show the post text and email address of all posts and the User who made them within the last 24 hours.

Show the post text and email address of all posts and the User who made them within the last 24 hours.
```SQL
SELECT posts.content, users.email, users.user_name, posts.post_time FROM posts
INNER JOIN users ON posts.user_id = users.user_id
WHERE ROUND((JULIANDAY('now') - JULIANDAY(posts.post_time))*24) <= 24;
```

### Question 2.10 Show the email addresses of all Users who have not posted anything yet.

Show the email addresses of all Users who have not posted anything yet.
```SQL
SELECT users.email FROM users
LEFT JOIN posts ON users.user_id = posts.user_id
GROUP BY users.user_id HAVING COUNT(posts.user_id) = 0;
```







