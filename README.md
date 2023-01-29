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


