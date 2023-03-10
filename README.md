# Modeling-Game-Player-Data-with-Amazon-DynamoDB

## Overview
"Imagine you are building an online multiplayer game, such as a battle royale game. In your game, groups of 50 players join a session to play a game, which typically takes around 30 minutes to play. During the game, you have to update a specific player’s record to indicate the amount of time the player has been playing, the number of kills they’ve recorded, or whether they won the game. Users want to see old games they’ve played, either to view the games’ winners or to watch a replay of each game’s action."

![maxresdefault](https://user-images.githubusercontent.com/106786020/215302115-8e3b1c7e-cfd2-45a5-b512-40561002e869.jpg)

## Basic setup
I started by setting up my AWS Cloud9 IDE "AWS Cloud9 is a cloud-based integrated development environment (IDE) that lets you write, run, and debug code with just a browser. AWS Cloud9 includes a code editor, debugger, and terminal. It also comes prepackaged with essential tools for popular programming languages and the AWS Command Line Interface (CLI) preinstalled so that you don’t have to install files or configure your lapto. Your AWS Cloud9 environment will have access to the same AWS resources as the user with which you signed in to the AWS Management Console."

![image](https://user-images.githubusercontent.com/106786020/215303773-c95555be-0532-4f59-8fcf-c2453e8e18f7.png)

I setup a new Cloud9 environment and configured a c1.medium instance, reviewed everything and created the environment. Cloud9 has three main areas that you need to be familiar with dealing with this kind of project.

- File explorer: On the left side of the IDE, the file explorer shows a list of the files in your directory.

- File editor: On the upper right area of the IDE, the file editor is where you view and edit files that you’ve selected in the file explorer.

- Terminal: On the lower right area of the IDE, this is where you run commands to execute code samples.


![image](https://user-images.githubusercontent.com/106786020/215304081-12692c6a-f4ab-48b1-ac77-c3905e687b0e.png)

Then I started downloading the supporting code, for this project I used Python scripts to interact with the DynamoDB API. I ran the following command in my terminal to download and unpack the code (download and unpack file). After running the commands I instantly took notice of the two directories in the AWS Cloud9 file explorer.

- application: The application directory contains example code for reading and writing data in your table. This code is similar to code you would have in your real game.

- scripts: The scripts directory contains administrator-level scripts, such as for creating a table, adding a secondary index, or deleting a table.

I really liked this little detail, because it really helps avoid any confusion and saves alot of time.

## Planning the Data Model
To start things off I began building my entity-relationship diagram. In my application I have 3 entities:

User - User entity represents a user in the application. A user can create multiple Game entities, and the creator of a game will determine which map is played and when the game starts. A User can create multiple Game entities, so there is a one-to-many relationship between Users and Games.

Game - Game contains multiple Users and a User can play in multiple different Games over time. Thus, there is a many-to-many relationship between Users and Games.

UserGameMapping - represents the relationship between the user and the game.

![image](https://user-images.githubusercontent.com/106786020/215307266-e18af3da-a209-4ee1-b8aa-a082e7f63413.png)


Next I had to consider user profile access patterns, The users of the game need to create user profiles. These profiles include data such as a user name, avatar, game statistics, and other information about each user. The game displays these user profiles when a user logs in. Other users can view the profile of a user to review their game statistics and other details. As a user plays games, the game statistics are updated to reflect the number of games the user has played, the number of games the user has won, and the number of kills by the user. Based on this information, we have three access patterns:

- Create user profile (Write) 
- Update user profile (Write)
- Get user profile (Read)

Then we had to consider the pregame access patterns. The game is a battle royale game. Players can create a game at a particular map, and other players can join the game. When 50 players have joined the game, the game starts and no additional players can join. When searching for games to join, players may want to play a particular map. Other players won’t care about the map and will want to browse open games across all maps. Based on this information, we have the following seven access patterns:

- Create game (Write)
- Find open games (Read)
- Find open games by map (Read)
- View game (Read)
- View users in game (Read)
- Join game for a user (Write)
- Start game (Write)

Finally we had to consider the in-game and post-game access patterns. During the game, players try to defeat other players with the goal of being the last player standing. The application tracks how many kills each player has during a game as well as the amount of time a player survives in a game. If a player is one of the last three surviving in a game, the player receives a gold, silver, or bronze medal for the game. Later, players may want to review past games they’ve played or that other players have played. After finding out this information we have three access patterns:

- Update game for user (Write)
- Update game (Write)
- Find all past games for a user (Read)


## Core Usage: User Profiles and Games

I started off by designing the primary key, first we have to look at some of our previous entities. User, Game, UserGameMapping A UserGameMapping is a record that indicates a user joined a game. There is a many-to-many relationship between User and Game. Since my data model has multiple entities with relationships among them, I would generally use a composite primary key with both HASH and RANGE values. The composite primary key gives me the Query ability on the HASH key to satisfy one of the query patterns we need. In the DynamoDB documentation, the partition key is called HASH and the sort key is called RANGE, and in this we use the API terminology interchangeably and especially when we discuss the code or DynamoDB JSON wire protocol format.

The other two data entities—User and Game—don’t have a natural property for the RANGE value because the access patterns on a User or Game are a key-value lookup. Because a RANGE value is required, we can provide a filler value for the RANGE key. With this in mind I used the following pattern for HASH and RANGE values for each entity type.


Entity	              HASH	                 RANGE

User              USER#<USERNAME>  	   #METADATA#<USERNAME>

Game	             GAME#<GAME_ID>	     #METADATA#<GAME_ID>

UserGameMapping	   GAME#<GAME_ID>	      USER#<USERNAME>

For the User entity, the HASH value is USER#<USERNAME>. For the RANGE value on the User entity, I'm using a static prefix of #METADATA# followed by the USERNAME value. For the RANGE value, it’s important that I have a value that is known, such as the USERNAME. This allows for single-item actions such as GetItem, PutItem, and DeleteItem. However, I also want a RANGE value with different values across different User entities to enable even partitioning if we use this column as a HASH key for an index. For that reason, we append the USERNAME. The Game entity has a primary key design that is similar to the User entity’s design. It uses a different prefix (GAME#) and a GAME_ID instead of a USERNAME, but the principles are the same.Finally, the UserGameMapping uses the same HASH key as the Game entity. This allows us to fetch not only the metadata for a Game but also all the users in a Game in a single query. We then use the User entity for the RANGE key on the UserGameMapping to identify which user has joined a specific game.

Next it was time to create a table I deployed the following Python script (create_table file). After this I declared the names and types of attributes that are used for primary keys, because I'm storing different entities in a single table, I can’t use primary key attribute names such as UserId. The attribute means something different based on the type of entity being stored. For example, the primary key for a user might be its USERNAME, and the primary key for a game might be its GAMEID. Accordingly, we use generic names for the attributes, such as PK (for partition key) and SK (for sort key). After configuring the attributes in the key schema, I specified the provisioned throughput for the table. DynamoDB has two capacity modes: provisioned and on-demand. In provisioned capacity mode, you specify exactly the amount of read and write throughput you want. You pay for this capacity whether you use it or not. In DynamoDB on-demand capacity mode, you can pay per request. The cost per request is slightly higher than if you were to use provisioned throughput fully, but you don’t have to spend time doing capacity planning or worrying about getting throttled. On-demand mode works great for spiky or unpredictable workloads. I decided to go with provisioned capacity. Then I ran the following Python script to create the table (table create command file).

Now it was time to Bulk-load data into the DynamoDB table. I pulled some randomly generated items from a file that include User, Game, and UserGameMapping entities. I then ran a file called bulk_load_table.py (bulk_load_table.py file) that reads the items in the items.json file and bulk-writes them to the DynamoDB table. In this script we use a higher-level Resource object. Resource objects provide an easier interface for using the AWS APIs, in this situation the resource object batches our requests.  The BatchWriteItem operation accepts as many as 25 items in a single request. The Resource object handles that batching for me rather than making us divide our data into requests of 25 or fewer items. After running the bulk_load_table.py script I loaded the table with data by running the following command (Python scripts file). I then ensured that all the data was loaded into the table by running a scan operation and getting the count. (dyanmodb scan file).

Now I had to set everything up to recive multiple entity types in a single request, unlike a relational database DynamoDB does not have joins that a relational database has. Instead, I have to design my table to allow for join-like behavior in requests. This type of requests spans two entity types: the Game entity and the UserGameMapping entity. I then ran the following code to retrive both the Game entity and the UserGameMapping entity for the game in a single request (fetch_game_and_players file). At the beginning of this script, I import the Boto 3 library and some simple classes to represent the objects in the application code. The real work of the script happens in the fetch_game_and_users function. This is similar to a function you would define in your application to be used by any endpoints that need this data. The fetch_game_and_users function does a few things. First, it makes a Query request to DynamoDB. This Query uses a PK of GAME#<GameId>. Then, it requests any entities where the sort key is between #METADATA#<GameId> and USER$. This grabs the Game entity, whose sort key is #METADATA#<GameId>, and all UserGameMappings, entities, whose keys start with USER#. Sort keys of the string type are sorted by ASCII character codes. The dollar sign ($) comes directly after the pound sign (#) in ASCII, so this ensures that I will get all mappings in the UserGameMapping entity. When I receive a response, we assemble the data entities into objects known by the application. I know that the first entity returned is the Game entity, so I create a Game object from the entity. For the remaining entities, I create a UserGameMapping object for each entity and then attach the array of users to the Game object. The end of the script shows the usage of the function and prints out the resulting objects. I then ran the following script to print the Game object and all  UserGameMapping objects to the console (appliation fetch_game_and_players file). This script shows how I can model the table and write my queries to retrieve multiple entity types in a single DynamoDB request. In a relational database, you use joins to retrieve multiple entity types from different tables in a single request. With DynamoDB, you specifically model your data, so that entities you should access together are located next to each other in a single table. This approach replaces the need for joins in a typical relational database and keeps the application high-performing as I scale up.

## Designing Game Access Patterns
  
  I began by modeling a sparse secondary index "Secondary indexes are crucial data modeling tools in DynamoDB. They allow you to reshape your data to allow for alternate query patterns. To create a secondary index, you specify the primary key of the index, just like when you previously created a table. Note that the primary key for a global secondary index does not have to be unique for each item. DynamoDB then copies items into the index based on the attributes specified, and you can query it just like you do the table." 
  
  "Using sparse secondary indexes is an advanced strategy in DynamoDB. With secondary indexes, DynamoDB copies items from the original table only if they have the elements of the primary key in the secondary index. Items that don’t have the primary key elements are not copied, which is why these secondary indexes are called “sparse.”"
  
  Since I have two access patterns for finding open games find open games (Read), find open games by map (Read). I can create a secondary index using a composite primary key where the HASH key is the map attribute for the game and the RANGE key is the open_timestamp attribute for the game, indicating the time the game was opened. The important part for me is that when a game becomes full, the open_timestamp attribute is deleted. When the attribute is deleted, the filled game is removed from the secondary index because it doesn’t have a value for the RANGE key attribute. This is what keeps our index sparse: It includes only open games that have the open_timestamp attribute. I then created the sparse secondary index for open games (games that are not at max capacity) by running the following script file (add_secondary_index file). I then finalized the and created the secondary index with the following command (create_add_secondary_index file).

Now it was time to query the sparse secondary index, I used the Query API against the secondary index that I created in the previous step to find all open games by map name. The secondary index is partitioned by map name, allowing me to make targeted queries to find open games. I then ran the following function (find_open_games_by_map file). The function accepts a map name and makes a query against the OpenGamesIndex to find all open games for the map. It then assembles the returned entities into Game objects that can be used in the application. I then executed the script by running the following command (py.find.open.games file).

Now that we set up everything so the user can find a game for a specific map, other users may be willing to play on any map. So I started setting up the scan for the sparse secondary index for the players that are willing to play in any open game regardless of the map. In general, you do not want to design your table to use the DynamoDB Scan operation because DynamoDB is built for surgical queries that grab the exact entities you need. A Scan operation grabs a random collection of entities across your table, so finding the entities you need can require multiple round trips to the database. However, sometimes Scan can be useful. In this situation, I have a sparse secondary index, meaning that the index shouldn’t have that many entities in it. In addition, the index includes only those games that are open, and that is exactly what I need. I then ran the following code (find open game (random map) file). The diffrence between this action and what I did before was instead of running the query() method on the DynamoDB client, I used the scan method instead. I then ran the following script with the following command (find open game (random map) command file)

After setting up the query() and scan() commands it was now time to add users to a game. When adding users to the game we need to do the following: 
- Confirm that there are not already 50 players in the game (each game can have a maximum of 50 players).
- Confirm that the user is not already in the game.
- Create a new UserGameMapping entity to add the user to the game.
- Increment the people attribute on the Game entity to track how many players are in the game.

"Note that accomplishing all of these things requires write actions across the existing Game entity and the new UserGameMapping entity as well as conditional logic for each of the entities. This is the kind of operation that is a perfect fit for DynamoDB transactions because you need to work on multiple entities in the same request, and you want the entire request to succeed or fail together."

I then setup the following script, the function in this script uses a DynamoDB transaction to add a user to a game(join.game file). I then ran the script with the following command (join.game.py.file).

Since the configuration for a user to join the game had been taking care of it was time to make sure there was an action to allow the game to start. As soon as a game has 50 players the game can start. When the application backened recieves a request to start the game we check three things:

- The game has 50 people signed up.
- The requesting user is the creator of the game.
- The game has not already started.

The way to handle each of these checks is to use a condition expression in a request to update the game, when all the checks pass we need to update the entity in the following ways:

- Remove the open_timestamp attribute so that it does not appear as an open game in the sparse secondary index from the previous module.
- Add a start_time attribute to indicate when the game started. To ensure all of these changes were met I set up the following script (start game file), It takes a game_id, requesting_user, and start_time, and it runs a request to update the Game entity to start the game. After i ran the script with the following command (start.game.py file).

Next it was time to add an inverted index, I set up the following script to add it (add.inverted.index file). I then ran the script with the following command (add.inverted.index.py file) and everything worked out well on that end.

Now that the inverted index was created it was time to retrieve games for the user. To handle this, we need to query the inverted index with the User whose Game entities we want to see. To do that I set up the following script (find.games.for.user file). This function takes a user name and returns all the games played by the given user. I then ran the script with the following command (find.game.for.user.py)


## Conclusion

After ironing out the last few parts of the application everything ending up running smoothly. I had alot of fun building this because there is a good bit of things that you can customize your own way. I also learned a few things that I didn't know about DynamoDB prior to doing this project. It's always nice to polish your skills and brush up on new topics, especially when there beneficial to your skill set. Being able to see the infrastructure of how most games work is also a plus, due to the fact that it will have your mind thinking about how the game could be working the next time you load one up. That being said other than a few bugs I ran into after fixing evrything and watching it all come together it definitely was more fun than I expected.


