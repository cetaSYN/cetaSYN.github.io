---
title: "5Charlie CTF - Blackjack 2.0"
date: 2020-04-25 15:46:41 -0500
categories:
- write-up
tags:
- ctf
- write-up
- miscellaneous
- secure-coding
---

A write-up of the Blackjack 2.0 Python application security challenge from 5Charlie CTF.

## Blackjack

### Blackjack - Challenge

I updated my Black Jack program to fix some bugs and update it to Python3! I think it's much better now but my friend says he is still able to cheat more than he should be able to.

Can you take a look at my program and let me know what line my error is in?

The flag is the line of code the error is in ( eg. flag{##} )

Max 10 attempts, do not brute force it. We can see your submissions.

**Attachments:** `blackjack2.py` (Below)

{% highlight python %}
__author__ = 'Daniel Fitzgerald'
'''
 A VERY simple Black Jack program
 Treat all Aces as 1's and all face cards as 10's
'''
import random


def math(cards):
    total = 0
    for x in range(0,len(cards)):
        total += cards[x]
    return total


def deal(hand, deck):
    card_drawn = deck.pop()
    hand.append(card_drawn)
    return card_drawn


def end(player, dealer, deck):
    print ("\n\tDealer's 2nd Card: {}".format(dealer[1]))
    print ("\tDealer total: {}".format(math(dealer)))
    if math(dealer) > math(player):
        print ("\n---------------------------")
        print ("You Lost!")
        print ("\tPlayer total: {}".format(math(player)))
        print ("\tDealer total: {}".format(math(dealer)))
        print ("---------------------------\n")
        return False
    print ("\nDealer stands on 16 or above.")
    while math(dealer) < 16:
        print ("\n\tDealer drew: {}".format(deal(dealer, deck)))
        print ("\tDealer total: {}".format(math(dealer)))
    if math(player) > 21:
        print ("\n---------------------------")
        print ("You Busted, You Lost!")
        print ("\tPlayer total: {}".format(math(player)))
        print ("---------------------------\n")
    elif (math(player) < math(dealer)) and (math(dealer) < 22):
        print ("\n---------------------------")
        print ("You Lost!")
        print ("\tPlayer total: {}".format(math(player)))
        print ("\tDealer total: {}".format(math(dealer)))
        print ("---------------------------\n")
    elif (math(player)<22) and (math(dealer)>21):
        print ("\n---------------------------")
        print ("Dealer Busted, You Win!")
        print ("\tPlayer total: {}".format(math(player)))
        print ("\tDealer total: {}".format(math(dealer)))
        print ("---------------------------\n")
        return True
    elif (math(player)==math(dealer)):
        print ("\n---------------------------")
        print ("You Tied!")
        print ("\tPlayer total: {}".format(math(player)))
        print ("\tDealer total: {}".format(math(dealer)))
        print ("---------------------------\n")
    elif (math(player)>math(dealer)):
        print ("\n---------------------------")
        print ("You Win!")
        print ("\tPlayer total: {}".format(math(player)))
        print ("\tDealer total: {}".format(math(dealer)))
        print ("---------------------------\n")
        return True
    else:
        print ("\nError!")
        print ("\tPlayer total: {}".format(math(player)))
        print ("\tDealer total: {}".format(math(dealer)))
    return False


def main():
    player = []
    dealer = []
    fresh_deck = [1,1,1,1,2,2,2,2,3,3,3,3,4,4,4,4,5,5,5,5,6,6,6,6,7,7,7,7,8,8,8,8,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10]
    deck = fresh_deck
    total_wins = 0
    print ("\nWelcome!")
    random.shuffle(deck)
    while True:
        player = []
        dealer = []
        print(fresh_deck)
        print(len(fresh_deck))
        if len(deck) < 40:
            deck = fresh_deck[:]
            random.shuffle(deck)
        user_input = input("\nEnter 1 to play or 2 to exit: ")
        if user_input=="1":
            print ("\nSTART!\n")
            print ("Dealer cards and total:")
            print ("\tDealer's 1st Card: {}".format(deal(dealer, deck)))
            print ("\tDealer's 2nd Card: [FACE DOWN]")
            deal(dealer, deck)
            print ("\nPlayer cards and total:")
            print ("\tPlayers 1st Card: {}".format(deal(player, deck)))
            print ("\tPlayers 2nd Card: {}".format(deal(player, deck)))
            print ("\tPlayers Total: {}".format(math(player)))
            while (math(player)<22):
                user_input = input("\nEnter 1 to Hit or 2 to Stand: ")
                if user_input=="1":
                    print ("\n\tPlayers next card: {}".format(deal(player, deck)))
                    print ("\tPlayer total: {}".format(math(player)))
                elif user_input=="2":
                    break
                else:
                    print ("Not a valid input")
            if (end(player, dealer, deck)):
                total_wins+=1
            print ("Number of wins: {}".format(total_wins))
        elif user_input=="2":
            break
        else:
            print ("Not a valid input")
    print ("Goodbye")
    print ("Thanks for playing!")


if __name__ == "__main__":
    main()
{% endhighlight %}

### Blackjack - Solution

This challenge is wonderfully frustratingly subtle.
It's logic error that enables you to manipulate the starting deck in a minor way, but when it comes to gambling that's a big deal.

Let's walk through the program

{% highlight python %}
def main():
    player = []
    dealer = []
    fresh_deck = [1,1,1,1,2,2,2,2,3,3,3,3,4,4,4,4,5,5,5,5,6,6,6,6,7,7,7,7,8,8,8,8,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10]
    deck = fresh_deck
    total_wins = 0
    print ("\nWelcome!")
    random.shuffle(deck)
    while True:
        player = []
        dealer = []
        print(fresh_deck)
        print(len(fresh_deck))
        if len(deck) < 40:
            deck = fresh_deck[:]
            random.shuffle(deck)
{% endhighlight %}

Did you catch that error? I certainly didn't my first few times.

{% highlight python %}
        if len(deck) < 40:
                    deck = fresh_deck[:]
                    random.shuffle(deck)
{% endhighlight %}

When the deck starts getting low, we reshuffle the deck using a shallow copy of the fresh deck. That checks out. 👍

But wait

{% highlight python %}
    fresh_deck = [1,1,1,1,2,2,2,2,3,3,3,3,4,4,4,4,5,5,5,5,6,6,6,6,7,7,7,7,8,8,8,8,9,9,9,9,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10]
        deck = fresh_deck
{% endhighlight %}

That's not a copy, that's a reference. When we start, `deck` and `fresh_deck` refer to the same object.
Which means that for our first few plays we're actually manipulating `fresh_deck` too.
The line `deck = fresh_deck` should be changed to `deck = fresh_deck[:]`

**Flag:** `flag{78}`
