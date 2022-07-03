# Current-Project-Investigating-a-poorly-designed-database-for-a-soial-news-aggregator-In-Progress-
Unfortunately, due to some time constraints before the initial launch of the site, the data model stored in Postgres hasn’t been well thought out, and is starting to show its flaws. I’ve been brought in for two reasons: first, to make an assessment of the situation and take steps to fix all the issues with the current data model, and then, once successful, to improve the current system by making it more robust and adding some web analytics.

3 sections to complete:

* Part I: Investigate the existing schema
* Part II: Create the DDL for your new schema
* Part III: Migrate the provided data


Here is the DDL used to create the schema:

CREATE TABLE bad_posts (
	id SERIAL PRIMARY KEY,
	topic VARCHAR(50),
	username VARCHAR(50),
	title VARCHAR(150),
	url VARCHAR(4000) DEFAULT NULL,
	text_content TEXT DEFAULT NULL,
	upvotes TEXT,
	downvotes TEXT
);

CREATE TABLE bad_comments (
	id SERIAL PRIMARY KEY,
	username VARCHAR(50),
	post_id BIGINT,
	text_content TEXT
);

Part I: Investigate the existing schema
1. As a first step, investigate the schema 

Part II: Create the DDL for your new schema
2. Dive deep into the heart of the problem and create a new schema.
3. New schema should at least reflect fixes to the shortcomings.  
4. To help you create the new schema, a few guidelines are provided:

	Guideline #1: here is a list of features and specifications that Udiddit needs in order to support its website and 	administrative interface:
    * Allow new users to register:
        * Each username has to be unique
        * Usernames can be composed of at most 25 characters
        * Usernames can’t be empty
        * We won’t worry about user passwords for this project
    * Allow registered users to create new topics:
        * Topic names have to be unique.
        * The topic’s name is at most 30 characters
        * The topic’s name can’t be empty
        * Topics can have an optional description of at most 500 characters.
    * Allow registered users to create new posts on existing topics:
        * Posts have a required title of at most 100 characters
        * The title of a post can’t be empty.
        * Posts should contain either a URL or a text content, but not both.
        * If a topic gets deleted, all the posts associated with it should be automatically deleted too.
        * If the user who created the post gets deleted, then the post will remain, but it will become dissociated from that user.
    * Allow registered users to comment on existing posts:
        * A comment’s text content can’t be empty.
        * Contrary to the current linear comments, the new structure should allow comment threads at arbitrary levels.
        * If a post gets deleted, all comments associated with it should be automatically deleted too.
        * If the user who created the comment gets deleted, then the comment will remain, but it will become dissociated from that user.
        * If a comment gets deleted, then all its descendants in the thread structure should be automatically deleted too.
    * Make sure that a given user can only vote once on a given post:
        * Hint: you can store the (up/down) value of the vote as the values 1 and -1 respectively.
        * If the user who cast a vote gets deleted, then all their votes will remain, but will become dissociated from the user.
        * If a post gets deleted, then all the votes for that post should be automatically deleted too.

	Guideline #2: In order to support its website and administrative interface. 
    * List all users who haven’t logged in in the last year.
    * List all users who haven’t created any post.
    * Find a user by their username.
    * List all topics that don’t have any posts.
    * Find a topic by its name.
    * List the latest 20 posts for a given topic.
    * List the latest 20 posts made by a given user.
    * Find all posts that link to a specific URL, for moderation purposes. 
    * List all the top-level comments (those that don’t have a parent comment) for a given post.
    * List all the direct children of a parent comment.
    * List the latest 20 comments made by a given user.
    * Compute the score of a post, defined as the difference between the number of upvotes and the number of downvotes

	Guideline #3: Use normalization, various constraints, as well as indexes in new database schema. 

	Guideline #4: New database schema will be composed of five (5) tables that should have an auto-incrementing id as their primary key.


Part III: Migrate the provided data. It’s time to migrate the data!  A few guidelines:
1. Topic descriptions can all be empty
2. Since the bad_comments table doesn’t have the threading feature, migrate all comments as top-level comments, i.e. without a parent
3. You can use the Postgres string function regexp_split_to_table to unwind the comma-separated votes values into separate rows
4. Don’t forget that some users only vote or comment, and haven’t created any posts. You’ll have to create those users too.
5. The order of your migrations matter! For example, since posts depend on users and topics, you’ll have to migrate the latter first.
6. Tip: You can start by running only SELECTs to fine-tune your queries, and use a LIMIT to avoid large data sets. Once you know you have the correct query, you can then run your full INSERT...SELECT query.
7. Write the DML to migrate the current data in bad_posts and bad_comments to your new database schema.
