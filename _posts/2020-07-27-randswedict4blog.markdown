---
title:  "Bad password policies"
date:   2020-07-27 11:25:35 
categories: security, python
---

# Intro
One thing that really riles me up is the how companys really likes to implement bad password policies, policies known as basic8, comp8 and so on.
The password policy basic8 just demands that the password is 8 characters of length, while comp8 demands 8 characters including uppercase, lowercase, a symbol and a digit.

## Problems with password policies
The problem with these types of policies is the fact that users have a hard time remembering these types of password containing different groups of characters, and thus tends to create easily guessed password like "Summ3r!!" to actually remember the passwords. 
Another problem with these types of password policies is that sys-admins tends to create default passwords like "Summer2020!" for employees, who then changes it to "Summer2021!" when they are forced to change passsword. By enforcing password changes on a regular intervall employees tends to stick to these systems for remembering passwords, thus solidifying the issue. 


## Solutions
So what is the solution for these bad implementations? 

1. Instead of genereating comp8 passwords, generate a passphrase containing random words. 
2. Don't change passwords on regular intervalls, just change passwords when a breach is known. 
3. Encourage the use of password managers, as these can generate strong passwords containing all character types (as this gives higher entropy per character), and also remember the passwords for the user. 
Ps. A strong generated passphrase can be the actual password for the password manager. 

This is all what led up to writing "randswedict4.py", wich is just a simple passhrase generator aimed att Swedish usage. But there is also a English version called "randdict4.py".

## The scripts

### randswedict4.py
~~~
import random
import string

#Function for getting a random passphrase
#Choses 4 random words from wordlist and concatenates as passphrase.
def passwd():
    with open('/usr/share/dict/swedish', 'r', encoding='iso-8859-1') as allWords:
        wordList = allWords.read()
        wordList = wordList.split()

    #Collects 4 random words and joins these to get the passphrase.
    for word in  wordList:	    
        randWord1 = random.choice(wordList)
        randWord2 = random.choice(wordList)
        randWord3 = random.choice(wordList)
        randWord4 = random.choice(wordList)
        randPasswd = '{0}{1}{2}{3}'.format(randWord1,randWord2,randWord3,randWord4)
    
    #Checks if password is longer then 20 characters, if not it runs again.
    if len(randPasswd) > 20:
        print('*' * len(randPasswd) + '*****','\n',
        'Random words are:\n', randWord1.capitalize(), randWord2.capitalize(), randWord3.capitalize(), randWord4.capitalize())
        print('*' * len(randPasswd) + '*****','\n',
        'Random passphrase is:\n',randPasswd, '\n Passphrase length:', len(randPasswd), 'characters')
        print('*' * len(randPasswd) + '*****')
    else:
        print('*** \t Short random passphrase, running again \t***')
        passwd()
        
passwd()
~~~
{: .language-python} 

### randdict4.py
~~~
import random
import string

#Function for getting a random passphrase
#Choses 4 random words from wordlist and concatenates as passphrase.
def passwd():
    with open('/usr/share/dict/american-english', 'r') as allWords:
        wordList = allWords.read()
        wordList = wordList.split()

    #Collects 4 random words and joins these to get the passphrase.
    for word in  wordList:	    
        randWord1 = random.choice(wordList)
        randWord2 = random.choice(wordList)
        randWord3 = random.choice(wordList)
        randWord4 = random.choice(wordList)
        randPasswd = '{0}{1}{2}{3}'.format(randWord1,randWord2,randWord3,randWord4)

    #Checks if password is longer then 20 characters, if not it runs again.
    if len(randPasswd) > 20:
        print('*' * len(randPasswd) + '*****','\n',
        'Random words are:\n', randWord1.capitalize(), randWord2.capitalize(), randWord3.capitalize(), randWord4.capitalize())
        print('*' * len(randPasswd) + '*****','\n',
        'Random passphrase is:\n',randPasswd, '\n Passphrease length:', len(randPasswd), 'characters')
        print('*' * len(randPasswd) + '*****')
    else:
        print('Short random passphrase, run again')

passwd()
~~~
{: .language-python}


### Results after exec

![randswedict4](https://dvardo.github.io/images/randswedict4/randswedict4.png)
{: .full}
