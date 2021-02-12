---
layout: post
title: "Hangman in Python"
date: 2021-02-08 20:35:46 -0800
categories: game-dev python
---
This is great practise for anyone who is just starting out coding, and it was one of the many first projects that I did before attending a coding bootcamp.
The main features of this are as follows:
-

This first part of the code is here to take a string, and replace each character with a dash.
We set guess = input so this allows the player to do an input

We then set it so that if the guess is equal to the current word, the player wins a letter or losses a try.

    def start_guessing():  # creates a function for guessing
        dashes = "-" * len(current_word)
        dashes = replace_dashes_with_guess(current_word, dashes, ' ')

    guesses_taken = 0  # set this to 0 so that players can have 10 trys
    while guesses_taken < 10:  # sets boundrys , giving playing 10 trys

        print(dashes)

        guess = input()  # allows player to do input

        if guess in current_word:
            print("That letter is in the secret word!")
            dashes = replace_dashes_with_guess(current_word, dashes, guess)
        else:
            print("Sorry, that is an invalid guess.")
            guesses_taken += 1
            print("Guesses remaining " + str(10 - guesses_taken))

this will handle our print statement to let the player know if their turn was right or wrong



             # print a statement letting player know they lost/won
      if guesses_taken < 10:
          print("you win!")
      else:
          print("you lose")

      sys.exit()        # exits out of game


we set empty_list to an empty string so that we can actually fill it with a string.  This sets
the logic to actually change the dashes to the letter if its the right one.

    def replace_dashes_with_guess(current_word, dashes, guess):
    empty_list = ""

    for i in range(len(current_word)):
        if guess == current_word[i]:
            empty_list += guess
        else:
            empty_list += dashes[i]
    return empty_list


This final bit is our word bank

    word_bank = ["i wish for a dog", "i wish for a cat", "i wish for a car"]  #a list of words that the player has to guess
    current_word = random.choice(word_bank)     #chooses a word from word bank

    start_guessing()

This code is simple but very much worthwhile