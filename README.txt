SER 421 - Module 4 Lab Submission:
Student: Nina Mason
Date: 9/16/2025

- MODULE 4 ACTIVITY 1: I fulfilled all 3 requirements for this part:
 - Implemented a Jeopardy game with 3 players, 4 categories, 5 questions per category using an HTML table for the board.
 - Randomly pull categories from the API and easy questions are $100 and $200, medium are $300 and $400, and hard are $500.
 -  When a player clicks on a dollar value the app shows the category and value selected followed by a trivia question and true or false buttons to answer with.
 - The player then answers the trivia question and either gets a "Correct!" or "Incorrect!" response, followed by which player goes next.
 - The box for the completed question shows red or green with a "P#" after the player answers showing which questions have been answered by what player and whether or not they got them wrong.
 - Play continues until all questions on the board have been completed, then once the board is complete a message is displayed saying "Player # wins with $(maxScore)!" if there is only one winner and "It's a tie! Players #, # win with $(maxScore)" if there is a tie between players.
 - I externalized the exceptions for the categories to a config.js file as well as a few other constants.
 - I implemented this project as a Vue SFC using version 3 and the Options API only. I also did not pull from any external libraries for this assignment.


- MODULE 4 ACTIVITY 2: I fulfilled the final 2 requirements (R5 and R6) for this assignment by copying my project folder from Activity 1 and adding onto the code in App.vue:
 - Implemented Double Jeopardy where the player is able to "wager" an amount of money up to the amount they have as their current balance:
  - It occurs 10% of the time.
  - If the players balance is less than the balance on the board it automatically sets their wager at face value, otherwise it will prompt them to provide an amount to wager and won't allow them to enter a value greater than their balance or less than 1. Then, the score adjusts accordingly.
 - Implemented a way to dynamically configure the number of players and categories before starting the game:
  - A page loads before the game starts that allows users to input "Number of players:" which can be between 2-6, as well as "Number of categories:" which can be between 2-10.
  - When these values are input and "Start Game" is clicked, the resulting player scoreboard and gameboard are loaded, where each category gets 5 questions each.

- HOW TO RUN: Using command line, navigate to the project directory and simply run command 'npm run dev', then copy and paste the localhost link into browser.
