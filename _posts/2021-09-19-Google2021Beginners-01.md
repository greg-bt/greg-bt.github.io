---
layout: writeup
title: "1 - Novosibirsk Chemical plant"
ctf: Google CTF 2021 Beginners Quest
nextWrite: "Google2021Beginners-02"
---
This first challenge takes us to a [website](https://cctv-web.2021.ctfcompetition.com/ "website") where we are greeted with a password form. Viewing the page source we can see a script tag, containing the `checkPassword` function. This function is ran when we submit the form so our first approach will be to reverse the function.

```javascript
const checkPassword = () => {
  const v = document.getElementById("password").value;
  const p = Array.from(v).map(a => 0xCafe + a.charCodeAt(0));

  if(p[0] === 52037 &&
     p[6] === 52081 &&
     p[5] === 52063 &&
     p[1] === 52077 &&
     p[9] === 52077 &&
     p[10] === 52080 &&
     p[4] === 52046 &&
     p[3] === 52066 &&
     p[8] === 52085 &&
     p[7] === 52081 &&
     p[2] === 52077 &&
     p[11] === 52066) {
    window.location.replace(v + ".html");
  } else {
    alert("Wrong password!");
  }
}
```

Dry running the program in the console, using "password" as test data, we can see that `v` is converted into an array `["p", "a", "s", "s", "w", "o", "r", "d"]` then each character is converted to a key code which is added to `0xCafe`. This gives us:

`[52078, 52063, 52081, 52081, 52085, 52077, 52080, 52066]`
which is then compared with the following values:


`[52037, 52077, 52077, 52066, 52046, 52063, 52081, 52081, 52085, 52077, 52080, 52066]`

<h2>Reversing</h2>

Reversing this function is quite easy and we can do it in the browser. Turning the values above into an array, we can then run them through an inverse function which looks something like this: 
```javascript
[].map(a => String.fromCharCode((a- 0xCafe))).join("")
```

Running this function on the array returns the string `GoodPassword` which we can the use to log into the website. Logging in to the site we are greeted with the following images.

![cctv]({{ site.baseurl }}/images/CCTVflag.png)

This reveals the flag to be: `CTF{IJustHopeThisIsNotOnShodan}`