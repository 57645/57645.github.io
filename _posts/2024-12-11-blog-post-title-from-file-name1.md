## Wordle Recreation

I've been learning to code recently, mostly following the CS50 intro to python course, which I'm ejoying a lot.  I've been combining my enjoyment of problem solving with more ambitious topics which has helped me to learn more.  I thought of recreating wordle as an interesting challenge because it's a relatively simple topic, the rules and end goals are clearly defined, it's fun to play and because I ultimately want to learn how to make a bot to beat the game (at the time of writing I've actually finished the game, and the bot is a work in process, but I'll get to that).  Eventually I'd like to learn front-end web development too, and this will be a simple way to deploy work I've done to a website and get an easily interactive game to play.

# Initial ideas
THe fundamental ideas behind wordle are simple, and I'm certainly not the first person to try to recreate it.  I aimed to make the game myself in my own way using what I'm comfortable with, and then work backwards to improve functionality and efficiency (a similar philosophy applies to this blog).  I initiated the program with the simplest steps, finding a list of valid wordle answers and guesses.  I used CFreshan's list, available [here](https://gist.github.com/cfreshman/a03ef2cba789d8cf00c08f767e0fad7b), and [here](https://gist.github.com/cfreshman/a7b776506c73284511034e63af1017ee).  Interestingly this is the original wordle list, the New York Times have their own curated list that gets updated and isn't static.  The list that was originally used for the answers was curated by the developers wife, who worked through the 12,000 acceptable guesses and marked the ones that she understood the meaning of.  This meant that theres a few works that aren't valid as guesses that you'd expect to be fine, probably by mistake.

Downloading both of these lists let me access them using python, both for validating user guesses and producing a random wordle every time the program run.  I'll quickly run through the process for both of these functions.

# Load wordle
```
def load_wordles():
    try:
        with open('wordle-answers.txt', 'r') as file:
            all_words = file.read()
            return random.choice((list(map(str, all_words.splitlines()))))
    except FileNotFoundError:
        print("There is no wordle file to pull the answer from")
```
This was a relatively simple function once I wrapped my head around loading files.  I could likely get rid of the except FileNotFoundError catch since it just runs on my computer and I'm not updating the list at all, but assumed it was good practice to include.  The function attempts to load the list of wordle-answers I downloaded, and returns a random individual word from the list.  If there is no file available then the code lets the user know in the terminal.  The function is only called once to define the wordle at the beginning of every run.
```
wordle = load_wordles()
```
# Correct guesses
Another relatively simple process was updating the progress variable to include characters that are both in the word, and in the correct location in the users guess.  Simply checking through the characters using a for loop, and comparing them to their corresponding character in the wordle allowed me to update progress with an uppercase character if it was correct, and move on to the next character if not.
```
    for char in range(len(guess)): #Check through word
        #Check for correct letters in correct place
        if guess[char] == wordle[char]:
            progress[char] = guess[char].upper()
```
# Letters in the wordle, but in the incorrect location (yellow characters)
This was the first source of real trouble with the program.  I initially over simplified and extended the loop checking if a character was green to check if the character was in the wordle at all.  The issue with this is that characters guessed twice in a word behave differently if they're correctly matched to a letter and go green.  E.g. If the wordle is MAGMA, and the users guess is MAMMA, output should be MA_MA.  Using the method mentioned above gives an output of MAmMA, making it hard for the user to correctly guess.
