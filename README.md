# Indexing-in-databases
indexing in databases to make the queries faster for a big scale application where queries needs to be executed faster

--after starting a postgres db locally and we create a table for users and their posts
--then we run a command that creates 5 users and for each user they create 250000 posts

# command to create table of users and posts>>>

CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    name VARCHAR(255)
);
CREATE TABLE posts (
    post_id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    image VARCHAR(255),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

--command to create 5 users and for each users there are 250000 posts>>>

DO $$
DECLARE
    returned_user_id INT;
BEGIN
    -- Insert 5 users
    FOR i IN 1..5 LOOP
        INSERT INTO users (email, password, name) VALUES
        ('user'||i||'@example.com', 'pass'||i, 'User '||i)
        RETURNING user_id INTO returned_user_id;

        FOR j IN 1..500000 LOOP
            INSERT INTO posts (user_id, title, description)
            VALUES (returned_user_id, 'Title '||j, 'Description for post '||j);
        END LOOP;
    END LOOP;
END $$;

--then we do this and see>

 # EXPLAIN ANALYSE SELECT * FROM posts WHERE user_id=1 LIMIT 40;


 it takes 0.75ms which is less

 but if we do explain analyse select count(*) from posts (((which counts all the posts in the table that is 5 mil)

 it takes 70.89 ms which is very high and not good for a large scale application




# results when we run a command which selcts posts from a user with id = 2 limits= 40>>

--EXPLAIN ANALYSE SELECT * FROM POSTS WHERE user_id=2 limit 40;
                                                    QUERY PLAN
-------------------------------------------------------------------------------------------------------------------
 Limit  (cost=0.00..4.53 rows=40 width=563) (actual time=29.699..29.704 rows=40 loops=1)
   ->  Seq Scan on posts  (cost=0.00..56542.00 rows=499000 width=563) (actual time=29.697..29.700 rows=40 loops=1)
         Filter: (user_id = 2)
         Rows Removed by Filter: 500000
 Planning Time: 0.077 ms
 Execution Time: 29.745 ms
(6 rows)

it takes 30 ms pretty slow

# to make it faster >>>

we addd indexing to it in the tables

# ADD and index to user_id >>>

CREATE INDEX idx_user ON posts(user_id);


# changes after indexing >>>

 EXPLAIN ANALYSE SELECT * FROM posts WHERE user_id=13 limit 40;
                                                       QUERY PLAN
------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=0.43..4.45 rows=1 width=563) (actual time=0.018..0.019 rows=0 loops=1)
   ->  Index Scan using idx_user on posts  (cost=0.43..4.45 rows=1 width=563) (actual time=0.017..0.017 rows=0 loops=1)
         Index Cond: (user_id = 13)
 Planning Time: 0.219 ms
 Execution Time: 0.045 ms
(5 rows)

the executiuion time changes to 0.045 ms

# what it did to make it faster >>>

Index Creation:

It creates an index named idx_user_id associated with the user_id column of the posts table.
Purpose of Index:

Indexes are used to improve the performance of queries that involve the user_id column, especially for operations like:
Searching: Queries like SELECT * FROM posts WHERE user_id = 42; will be faster.
Sorting: Queries that order rows by user_id can benefit from the index.
Joins: When joining posts with another table based on user_id, the index can optimize the operation.
How It Works:

The database creates a data structure (like a B-tree or a hash table) for the user_id column. This structure allows the database to locate rows with specific user_id values much more quickly than scanning the entire table.
Performance Considerations:

Benefits: Speeds up read operations (queries).
Trade-offs:
Slows down write operations (INSERT, UPDATE, DELETE) because the index must be updated whenever the user_id column changes.
Consumes additional storage space.
Name of the Index:

The index is named idx_user_id for reference. You can use this name to manage or drop the index later.
General Use Case:

Indexes are often created on columns that are frequently used in WHERE clauses, JOIN conditions, or ORDER BY clauses.
In summary, this command is optimizing your database queries involving the user_id column in the posts table by creating an index for it.

# in short what the indexing does is >>

it creats another data structure where suppose the posts of each user's address in the database are stored in that data structure in a sorted manner


![image](https://github.com/user-attachments/assets/eba68dee-3840-432c-b83b-65f3b0830c0f)


so basically from the image  --> it points to the location of the data in the database

so suppose someone asks for user 5 posts then you dont linearly search the whole database>

what you do is we search the data structure we made that is indexing there it is in sorted manner >

applly binary search over there and get the location of the posts of the user with id  = 5 ; 
which results in quering faster in O(log(n))


# ANOTHER PROBELM THAT ARISES IN DATABASES>>

suppose users are searching for a post with title and description then it will take time too .

like >>

![image](https://github.com/user-attachments/assets/750ddb49-7ad0-45d8-8440-d299bbba0c45)


as you can see it takes around 174 ms which is very very high.


so to tackle this we use a concept called complex indexing >>

we can see we are searching based on title and description of the posts and not just user_id

CREATE INDEX idx_posts_user_id_title ON posts (description, title);

after complex indexing >>

![image](https://github.com/user-attachments/assets/4dcc4375-c0a9-4a30-9cfa-74a014972e80)


# INDEXING IN PRISMA >>

![image](https://github.com/user-attachments/assets/5a0d6b60-7fbe-414f-afa5-fa4aef627c4f)

what happened above is every posts has index to an user id so that when somone searches for some user's posts then the indexing can happen smoothly
as all the posts are indexed with an userID







