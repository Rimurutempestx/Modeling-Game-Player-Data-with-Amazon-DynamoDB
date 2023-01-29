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

