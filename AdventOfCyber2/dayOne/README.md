# [Advent of Cyber - Day 2](https://tryhackme.com/room/adventofcyber2)

`Machine IP - 10.10.72.193`

First of all, there isn't any need for recon or port scanning because in the task bar we are given the hint that this will be `Web Exploitation`.

So, with that, I directly went to the website `http://10.10.72.193/` and this is what we see

[1.png]

Here, the introduction told us `Register for an account, and then login`. So, doing that we are then redirected to this site.

[2.png]

`Question 1 - What is the name of the cookie used for authentication?`

Not only does the introduction tells us this explicitly, but I knew this from previous rooms and from web development. Right clicking and heading to `inspect element` then all the way to `Application` tab, we are presented with the following table.

[3.png]

Here, we can see the name and value of the cookies

`Question 2 - In what format is the value of this cookie encoded?`

So, this I didn't really know. I went to [CyberChef](https://gchq.github.io/CyberChef/) and gave the cookie value as input. Then, since I didn't know the format, I used the `magic` option. 

[4.png]

`Question 3 - Having decoded the cookie, what format is the data stored in?`

So, then I decided to decode that and got the following partial object `{"company":"The Best Festival Company", "username":"hacker"}`.

[5.png]

Now, it's telling us to `Figure out how to bypass the authentication`. This is where I got a little bit stuck for a few two minutes thinking how I was going to do that when I realized that in the next question it gave me the answer.

`Question 4 - What is the value of Santa's cookie?`

So, given the fact that this is an object with two properties `company` and `username`, I thought that we should just change the `username` to `santa` then encode that with the previous cookie encoded method using [rapidtables](https://www.rapidtables.com/convert/number/ascii-to-hex.html).

[6.png]

Doing that, I then inserted the value into the value for the cookie in the previous `Application` tab. Since it didn't change right away, I refreshed to apply changes. Giving me the following panel.

[8.png]

Obviously, `Now that you are the santa user, you can re-activate the assembly line!`

`Question 5 - What is the flag you're given when the line is fully active?`

[9.png]