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

To solve this issue I needed to record the characters that had been correctly guessed, and leave the same characters alone if they were repeated.  The process was simplified when I wrote it out in plain English.  The criteria for a character to be yellow is:
1. Present in the wordle
2. Not present in the characters that have already been guessed (another way of saying this is that the character is present in the characters that haven't been guessed).

To represent this I created a leftovers list, made up of characters from the wordle that were not green.  I then used a for loop to check if a character was in the wordle.  If it was, then it was checked to see if it was also present in the 'leftover' letters.  If so, the character was in the wordle, but in the wrong place and marked as lowercase.  If not, the loop moved on to the next character.  This loop ran through all characters, concluding the program and outputting the result of their guess.
```
  leftovers = [x for x in check_letters if x not in used_pos]
  letter_list = str(incorrect_letters)
  for char in leftovers:
      if guess[char] in wordle:
          if guess[char] in letter_list:
              progress[char] = guess[char].lower()
```
# Record results
As an afterthought and to test my dictionary knowledge I added a function to record user results and output them at the end of the program.  This gave me some experience with updating txt files outside of the program, and storing different usernames.  The funtion asks for the users name, and then either initiates a scoring section for them, or moves on if they already have an account (and therefore scores saved).
```
def trigger_results():
    global user
    try:
        with open('user_results.txt', 'r') as file:
            user_results = json.load(file)
    except FileNotFoundError:
        user_results = {}
    user = input("Username: ")
    if user not in user_results:
        user_results[user] = {"score1": 0, "score2": 0, "score3": 0, "score4": 0, "score5": 0, "score6": 0, "scorefail": 0,}
        with open('user_results.txt', 'w') as file:
            json.dump(user_results, file, indent=4)
```
When either the wordle is correctly guessed, or the user runs out of attempts, the program outputs their result in a simple count of their attempts, and then a historical record.  The current result is added to the tally and then output so that they can keep track easily.
```
#Code for when user guesses the wordle
print(f"Correct: You got it in {i}")
with open('user_results.txt', 'r') as file:
    user_results = json.load(file)
    score_key = f"score{i}"
    user_results[user][score_key] += 1
with open('user_results.txt', 'w') as file:
    json.dump(user_results, file, indent=4)
print("Your previous results are: ")
attempts = 1
for i in user_results[user]:        
    print(f"{attempts} attempts: {user_results[user][i]}")
    attempts += 1
    if attempts == 7:
        attempts = "Failed"
        print(f"{attempts} attempts: {user_results[user][i]}")
        break
```
# Next steps
Now that I have a functioning wordle program, the next step will be to build a solver.  I've got some ideas that are relatively simple, but would ideally like to work up to a point where the program effectively plays against itself.  I'll write another blog when this is completed, I've got some preliminary progress already.
