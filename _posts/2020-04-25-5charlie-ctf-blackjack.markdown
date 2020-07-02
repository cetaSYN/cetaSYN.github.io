---
title: "5Charlie CTF - Blackjack"
date: 2020-04-25 02:15:28 -0500
categories:
- write-up
tags:
- ctf
- write-up
- miscellaneous
- secure-coding
---

A write-up of the Blackjack Python application security challenge from 5Charlie CTF.

## Blackjack

### Blackjack - Challenge

I created a program to play a very simple version of Black Jack, but my friend is winning way more than he should be! Can you take a look at my program and let me know what line my error is in? He said he found the bug after crashing the program...

The flag is the line of code the error is in ( eg. flag{##} )

Max 15 attempts, do not brute force it. We can see your submissions.

**Attachments:** `miscellaneous_misc2_BlackJack.py` (Below)

{% highlight python %}
__author__ = 'Daniel Fitzgerald'
'''
 A VERY simple Black Jack program
 Treat all Aces as 1's and all face cards as 10's
 Reshuffles the deck after a certain number of hands
'''
import random
player = []
dealer = []
deck = [1,1,1,1,2,2,2,2,3,3,3,3,4,4,4,4,5,5,5,5,6,6,6,6,7,7,7,7,8,8,8,8,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10]
num_left = 51 # 0 to 51 = 52 card deck
total_wins = 0
reshuffle = 1
def draw():
    global num_left # not sure if this is needed
    rand = random.randint(0,num_left)
    num_left -= 1
    return deck.pop(rand)
def math(cards):
    total = 0
    for x in range(0,len(cards)):
        total += cards[x]
    return total
def deal(hand):
    card_drawn = draw()
    hand.append(card_drawn) #might not be doing this properly, looks like im trying to effect it globally
    return card_drawn
def end():
    global total_wins
    global player
    global dealer
    while math(dealer) < 16: #do i really want to deal the dealer before i check if i went over?
        print "Dealer drew: %d" %(deal(dealer)) #double check
        print "Dealer total: %d" %(math(dealer)) #double check
    if math(player) > 21:
        print "\n---------------------------"
        print "You Busted, You Lost!"
        print "Player total: %d" %(math(player))
        print "---------------------------\n"
    elif (math(player) < math(dealer)) and (math(dealer) < 22):
        print "\n---------------------------"
        print "You Lost!"
        print "Player total: %d" %(math(player))
        print "Dealer total: %d" %(math(dealer))
        print "---------------------------\n"
    elif (math(player)<22) and (math(dealer)>21):
        print "\n---------------------------"
        print "Dealer Busted, You Win!"
        total_wins+=1
        print "Player total: %d" %(math(player))
        print "Dealer total: %d" %(math(dealer))
        print "---------------------------\n"
    elif (math(player)==math(dealer)):
        print "\n---------------------------"
        print "You Tied!"
        print "Player total: %d" %(math(player))
        print "Dealer total: %d" %(math(dealer))
        print "---------------------------\n"
    elif (math(player)>math(dealer)):
        print "\n---------------------------"
        print "You Win!"
        total_wins+=1
        print "Player total: %d" %(math(player))
        print "Dealer total: %d" %(math(dealer))
        print "---------------------------\n"
    else:
        print "\nError!"
        print "Player total: %d" %(math(player))
        print "Dealer total: %d" %(math(dealer))
def main():
    global reshuffle
    global deck
    global num_left
    print "\nWelcome!"
    while True:
        global dealer
        global player
        player = []
        dealer = []
        if reshuffle != 0:
            print "%d cards left in deck" %(num_left)
            reshuffle -= 1
        elif reshuffle==0:
            deck = [1,1,1,1,2,2,2,2,3,3,3,3,4,4,4,4,5,5,5,5,6,6,6,6,7,7,7,7,8,8,8,8,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10]
            num_left = 51
            reshuffle = 1
            print "Deck reshuffled"
            print "%d cards left in deck" %(num_left)
        print "Number of wins: %d" %(total_wins)
        user_input = raw_input("Press 1 to play  or 2 to exit: ")
        if user_input=="1":
            print "\nSTART!"
            print "Dealer's 1st card: %d" %(deal(dealer))
            print "Player cards and total:"
            print "Players 1st Card: %d" %(deal(player))
            print "Players 2nd Card: %d" %(deal(player))
            print "Players Total: %d" %(math(player))
            while (math(player)<22):
                user_input = raw_input("Enter 1 to Hit or 2 to Stay: ")
                if user_input=="1":
                    print "Players next card: %d" %(deal(player))
                    print "Player total: %d" %(math(player))
                elif user_input=="2":
                    break
                else:
                    print "Not a valid input"
            end()
        elif user_input=="2":
            break
        else:
            reshuffle=2
            print "Not a valid input"
    print "Goodbye"
    print "Thanks for playing!"

#RUN
main()
{% endhighlight %}

### Blackjack - Solution

This challenge is a bit unique because it's a minor logic error that could happen in the real-world after a refactor.
Luckily for us it calls attention to itself by virtue of changing program state on invalid input.
Notice this segment:

{% highlight python %}
        elif user_input=="2":
            break
        else:
            reshuffle=2  # Uh oh
            print "Not a valid input"
    print "Goodbye"
{% endhighlight %}

Invalid input really shouldn't be affecting the program state except maybe in keeping track of errors.
Jumping back up we can see it has its effect in this segment:

{% highlight python %}
        if reshuffle != 0:
            print "%d cards left in deck" %(num_left)
            reshuffle -= 1
        elif reshuffle==0:
            deck = [1,1,1,1,2,2,2,2,3,3,3,3,4,4,4,4,5,5,5,5,6,6,6,6,7,7,7,7,8,8,8,8,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10]
            num_left = 51
            reshuffle = 1
            print "Deck reshuffled"
            print "%d cards left in deck" %(num_left)
{% endhighlight %}

By entering an invalid input between session, we are able to prevent the deck from being reshuffled.
This allows us to analyze the cards remaining and make more informed guesses as to the chances of winning.
The line `reshuffle=2` should be removed.

**Flag:** `flag{111}`
