---
layout: post
title: "lotto challenge in Pwnable.kr"
date: 2019-08-01 +0100
categories: ctf
---
In this post we solve the *lotto* challenge of [pwnable.kr](https://www.pwnable.kr).

### lotto

The challenge starts by inviting us to play a game of chance 

```
Mommy! I made a lotto program for my homework.
do you want to play?


ssh lotto@pwnable.kr -p2222 (pw:guest)
```
Inspecting the code, it reads 6 bytes from `/dev/urandom` which leaves us with 256^6 possible values. My initial thought, before seeing the code was that this would be poorly implemented seeding and use of a random integer function, however, `/dev/urandom/` is safe as it is seeded by the OS during boot. 

Later in the code, the check against the password is made, looping over the random bytes and the user input 36 times. An `int` is incremented for each match. 

{% highlight c %}
// calculate lotto score
int match = 0, j = 0;
for(i=0; i<6; i++){
        for(j=0; j<6; j++){
                if(lotto[i] == submit[j]){
                        match++;
                }
        }
}
{% endhighlight %}

This however, means that if we just match with one of the bytes from the random input 6 times, we get the flag. Hence any byte repeated 6 times wins the challenge.

{% highlight bash %}
lotto@prowl:~$ ./lotto
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes : $$$$$$
Lotto Start!
bad luck...
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes : $$$$$$
Lotto Start!
bad luck...
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes : $$$$$$
Lotto Start!
bad luck...
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes : $$$$$$
Lotto Start!
[flag omitted]
{% endhighlight %}
