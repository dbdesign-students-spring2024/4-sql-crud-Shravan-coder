# SQL CRUD

An assignment to design relational database tables with particular applications in mind.

The contents of this file will be deleted and replaced with the content described in the [instructions](./instructions.md)

## Part 1: Restaurant Finder

### Creating the restaurants table

Code:
create table restaurants (id NUMERIC PRIMARY KEY, category TEXT, price_tier TEXT, neighbourhood TEXT, opening_time TEXT, closing_time TEXT, average_rating REAL, good_for_kids NUMERIC);

This creates a table called "restaurants" with all the previously asked for parameters. Since SQLite does not have an inbuilt boolean data type, I stored good_for_kids as a numeric value with only 0 (False) and 1 (True) as the parameters.

### Importing Data into SQLite testing

Code:
.import /Users/shravan/Desktop/4-sql-crud-SHRAVAN-CODER-main/data/restaurants.csv restaurants

Link to restaurants.csv (on my computer): /Users/shravan/Desktop/4-sql-crud-SHRAVAN-CODER-main/data/restaurants.csv

### Queries

#### 1. Find all cheap restaurants in a particular neighborhood

Code:
SELECT * FROM restaurants WHERE price_tier = 'cheap' AND neighbourhood = 'Chelsea';

#### 2. Find all restaurants in a particular genre (pick any genre as an example) with 3 stars or more, ordered by the number of stars in descending order.

Code:
SELECT * FROM restaurants WHERE category = 'Indian' AND average_rating >= 3 ORDER BY average_rating DESC;

#### 3. Find all restaurants that are open now

Code:
SELECT * FROM restaurants WHERE strftime('%H:%M', 'now', 'localtime') BETWEEN opening_hours AND closing_hours;

#### 4. Leave a review for a restaurant

Code:
CREATE TABLE reviews (
    id NUMERIC PRIMARY KEY AUTOINCREMENT,
    restaurant_id NUMERIC,
    rating NUMERIC,
    comment TEXT,
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(id)
);

INSERT INTO reviews (restaurant_id, rating, comment) VALUES (529, 4, 'Great experience at this restaurant!');

Here, we first created a separate table reviews that with a column that references the restaurant ids. Therefore, if we specify which restaurant we are reviewing, we can add a review to the list.

#### 5. Delete all restaurants that are not good for kids

Code:
DELETE FROM restaurants WHERE good_for_kids = 0;

#### 6. Find the number of restaurants in each NYC neighborhood.

Code: SELECT neighbourhood, COUNT(*) as num_restaurants FROM restaurants GROUP BY neighbourhood;

Displays the number of restaurants in each neighbourhood in the column 'num_restaurants'.

## Part 2: Social Media App

### Creating tables

#### users table

Code:
create table users(
    id NUMERIC PRIMARY KEY AUTOINCREMENT, 
    email TEXT, 
    password TEXT, 
    handle TEXT);

#### messages table

Code: 
CREATE TABLE messages (                                                                  
    message_id NUMERIC PRIMARY KEY AUTOINCREMENT,
    sender_id NUMERIC,
    receiver_id NUMERIC,
    message_text TEXT,
    view_status NUMERIC,
    sent_at TEXT,
    FOREIGN KEY (sender_id) REFERENCES users(id),
    FOREIGN KEY (receiver_id) REFERENCES users(id));

#### stories table

CREATE TABLE stories (                                                                                                         
    story_id NUMERIC PRIMARY KEY AUTOINCREMENT,
    user_id NUMERIC,
    story_text TEXT,
    posted_at TEXT,
    invisible_after_24h NUMERIC,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

### Importing Data into SQLite

Code:
.import /Users/shravan/Desktop/4-sql-crud-SHRAVAN-CODER-main/data/users.csv users

link: 4-sql-crud-SHRAVAN-CODER-main/data/users.csv

Code:
.import /Users/shravan/Desktop/4-sql-crud-SHRAVAN-CODER-main/data/postsmessages.csv messages

link: 4-sql-crud-SHRAVAN-CODER-main/data/postsmessages.csv messages

Code:
.import /Users/shravan/Desktop/4-sql-crud-SHRAVAN-CODER-main/data/postsstories.csv stories

link: 4-sql-crud-SHRAVAN-CODER-main/data/postsstories.csv stories

### Queries

#### 1. Register a new User.

Code:
INSERT INTO users (email, password, handle) VALUES ('newuser@example.com', 'password123', 'newuserhandle');

#### 2. Create a new Message sent by a particular User to a particular User (pick any two Users for example).

Code:
INSERT INTO messages (sender_id, receiver_id, message_text, view_status, sent_at) VALUES (121, 221, 'Hello, how are you?', 0, CURRENT_TIMESTAMP);

#### 3. Create a new Story by a particular User (pick any User for example).

Code:
INSERT INTO stories (user_id, story_text, posted_at) VALUES (1, 'This is a new story!', CURRENT_TIMESTAMP);

#### 4. Show the 10 most recent visible Messages and Stories, in order of recency.

Code:
SELECT *
FROM (
    SELECT sender_id AS user_id, message_text AS post_text, sent_at AS post_time
    FROM messages
    WHERE view_status = 1
    UNION ALL
    SELECT user_id, story_text, posted_at
    FROM stories
    WHERE invisible_after_24h = 0
)
ORDER BY post_time DESC
LIMIT 10;


#### 5. Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.

Code:
SELECT * FROM messages WHERE sender_id = 431 AND receiver_id = 153 ORDER BY sent_at DESC LIMIT 10;

#### 6. Make all Stories that are more than 24 hours old invisible.

Code:
UPDATE stories SET invisible_after_24h = 1 WHERE posted_at <= datetime('now', '-1 day');

#### 7. Show all invisible Messages and Stories, in order of recency.

Code:
SELECT sender_id AS user_id, message_text AS post_text, sent_at AS post_time, view_status FROM messages WHERE view_status = 1
UNION ALL
SELECT user_id, story_text, posted_at, invisible_after_24h FROM stories WHERE invisible_after_24h = 1

#### 8. Show the number of posts by each User.

Code:
SELECT users.id, users.handle, COUNT(messages.message_id) AS num_messages, COUNT(stories.story_id) AS num_stories
FROM users
LEFT JOIN messages ON users.id = messages.sender_id
LEFT JOIN stories ON users.id = stories.user_id
GROUP BY users.id, users.handle;

Shows the number of messages and stories by every user in the data set.

#### 9. Show the post text and email address of all posts and the User who made them within the last 24 hours.

Code:
SELECT messages.message_text AS post_text, users.email AS user_email, 'Message' AS post_type, messages.sent_at AS post_time
FROM messages
JOIN users ON messages.sender_id = users.id
WHERE messages.sent_at >= datetime('now', '-1 day')
UNION ALL
SELECT stories.story_text, users.email, 'Story', stories.posted_at
FROM stories
JOIN users ON stories.user_id = users.id
WHERE invisible_after_24h = 1
ORDER BY post_time DESC;

#### 10. Show the email addresses of all Users who have not posted anything yet.

Code:
SELECT email
FROM users
WHERE id NOT IN (
    SELECT DISTINCT sender_id
    FROM messages
    UNION
    SELECT DISTINCT user_id
    FROM stories
);
