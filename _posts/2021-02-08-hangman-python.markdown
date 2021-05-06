---
layout: post
title: "Hangman in Python"
date: 2021-02-08 20:35:46 -0800
categories: game-dev python
---

This is great practise for anyone who is new to coding. This was one of the many first projects that I did before attending a coding bootcamp.

This app has the following features:

- Create one or more word banks.
- Randomly selects a word/phrase from the word bank.
- Takes selected word/phrase, and converts the string into dashes, one dash for each character. If it's  a statement, we will put spaces in between each word.
- As the player guesses Correctly , replace a dash with a letter.
- The game allows the player to have 10 wrong guesses.
- If the player loses all 10 tries, then it's game over man. Game Over!
- If the player guesses all the characters correctly in the allotted amount of tries, the app will end.
- A simple  hangman game that will impress your friends and family, you can be the cool one at the party with this game.

All notes will be inserted into the codebase, use this as a reference later on down the line.

{% highlight python %}

def start_guessing():  
    # This will replace the characters of the string with dashes as well as  
    # empty spaces between words.
    dashes = "-" * len(current_word)
    dashes = replace_dashes_with_guess(current_word, dashes, ' ')
    # set this to 0 so that players can have 10 tries.
    guesses_taken = 0  
    # If you want more/less tries for the player you can change 
    # this number.
    while guesses_taken < 10: 

        print(dashes)
        # This is how you set the guess of the player to the input of their key 
        # strokes.
        guess = input()  
        # This handles if a guess is in the current word/phrase.
        if guess in current_word:
            print("That letter is in the secret word!")
            dashes = replace_dashes_with_guess(current_word, dashes, guess)
        else:
            print("Sorry, that is an invalid guess.")
            guesses_taken += 1
            # This tells how man guesses the player has left.
            print("Guesses remaining " + str(10 - guesses_taken))

        if guesses_taken < 10:
            print("you win!")
        else:
            print("you lose")
          # exits out of game
        sys.exit()      

    # We set empty_list to an empty string so that we can actually fill 
    # it with a string.
    # This sets the logic to actually change the dashes to the letter 
    # if it's the right one.

def replace_dashes_with_guess(current_word, dashes, guess):
    empty_list = ""
    # This handles the logic to switch dashes to the character of the 
    # word/phrase in the wordbank. This logic also adds +1 to guess
    # if player input is wrong.
    for i in range(len(current_word)):
        if guess == current_word[i]:
            empty_list += guess
        else:
            empty_list += dashes[i]
    return empty_list

# Initialize and start the game.
# A list of words/phrases that the player has to guess.
# You can add more at your leisure.
word_bank = ["i wish for a dog", "i wish for a cat", "i wish for a car"]  
# Chooses a word from the word bank.
current_word = random.choice(word_bank) 
# Gets the game started
start_guessing()
{% endhighlight %}

