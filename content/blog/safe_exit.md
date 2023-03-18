---
title: "A Safer Exit Button"
date: 2023-03-18T15:19:42-04:00
showDate: false
url: /blog/safe_exit
---

*If you would like to skip the preamble and just get the code, [click here](https://gist.github.com/adilahmed31/611a2c48537e90f43a1f65d650663f51)*

In the fall of 2021, I was volunteering at the [Madison Tech Clinic](https://techclinic.cs.wisc.edu) to provide technical support to assist victims of technology-enabled surveillance by intimate partners. Simultaneously, we decided to create a simple website to better educate domestic abuse advocates on our services. 

(Un)fortunately, I drew the short straw of designing and deploying the website. Given the nature of the site, we wanted a prominent "Safe Exit" button across every page to allow a user to quickly close the page if needed. With zero experience of any sort of web design, I found this an interesting challenge to work through and decided to write up my observations here. Let me know your thoughts and any possible improvements you can think of! 

The requirements for this button are straightforward:

1. On being clicked, instantly redirect the user to an innocuous page (we chose the google home page)
2. Clicking the back button should not return the user to our website
3. The site leaves no trace in the browser history (stretch goal)

I couldn't find any existing implementations that matched all these requirements (most only matched #1) so I decided to make my own. However, a quick shoutout to [Chris Coyier](https://css-tricks.com/website-escape/)'s simple implementation that served as a valuable starting point.

Challenge 1: href() vs replace()

A redirection can either be performed using `window.location.href` or `window.location.replace` . The key [difference](https://www.geeksforgeeks.org/difference-between-window-location-href-window-location-replace-and-window-location-assign-in-javascript/#:~:text=The%20href%20property%20on%20the,return%20to%20the%20current%20page) here is that `window.location.href` adds the current URL to the history object while the `replace()` function deletes the current object from the history. 

We implement a simple function to hijack every call to href on the page to use replace() instead, as provided below:
```
function new_window(){
if (!document.getElementsByTagName) return false;
    var links = document.getElementsByTagName("a");
    for (var eleLink=0; eleLink < links.length; eleLink ++) {
        var b = links[eleLink];
        if(b.getAttribute("href") && b.hostname == location.hostname){
            links[eleLink].onclick = function() {
                window.location.replace(this.href);
                return false;
            }
        }
    }
}
```

Challenge 2: Prevent navigation using the Back button

Even after re-wiring all the hyperlinks on the page, we still have 2 edge cases. 

1. When the user types the URL directly into the browser, the object is now part of the history and the back button can lead a user to the homepage even though each subsequent page is cleared from history.
2. If the site is linked from a different website using href, the back button would still work and send a user back to the homepage. 

To combat this, in addition to rewriting every link - we reload the page using location.replace() for every page load from an external site - thereby replacing the history object. Every action on the website will simply update this object. On clicking SAFE EXIT, we're left with one object in the history which says "www.google.com"
```
function reload(){
    window.location.replace(window.location);
}
```
Challenge 3: Auto-reload doesn't work when navigating from an external site

One loophole still exists. If the site is linked from somewhere else, this auto-reload behavior will not work. For this we implement a referrer check - if there is no referrer or the referrer is of another site, the `window.location.replace` method is run again to replace the current item in the session history. 
```
if (!document.referrer || document.referrer.split('/')[2].split(':')[0] != "techclinic.cs.wisc.edu"){
```
The complete implementation:
```
<button onclick="exit1()">SAFE EXIT</button>
function exit1(){
    window.location.replace("https://www.google.com");
}

<script>
// Reload the page when linked from the same site

function new_window ()
{
    if (!document.getElementsByTagName) return false;

    var links = document.getElementsByTagName("a");
    for (var eleLink=0; eleLink < links.length; eleLink ++) {
        var b = links[eleLink];
        if(b.getAttribute("href") && b.hostname == location.hostname){
            links[eleLink].onclick = function() {
                window.location.replace(this.href);
                return false;
            }
        }
    }
}
// Reload the page when linked from an external site

function reload1()
{
    if (!document.referrer || document.referrer.split('/')[2].split(':')[0] != "techclinic.cs.wisc.edu"){
        window.location.replace(window.location);
    }
}

new_window();
reload1();

</script>
```


Alas, nothing in life is perfect and that applies doubly for this rough-shod implementation of an exit button. We suffer two main issues in this implementation:

Issue 1: The Back button no longer works on any page

Given the back button goes to the previous object in the window.history, our method might hurt the usability of the site for users who find it intuitive to use the Back and forward buttons to navigate. We can combat this by showing a quick message on page load "To preserve your privacy, the Back button is disabled. Please use the links to navigate and the SAFE EXIT button to close the website"

The site layout is designed in a way to make it easy and clear to navigate using the hyperlinks at the top.

Issue 2: Inconsistency Across Browsers

Requirements #1 and #2 described above worked perfectly across browsers but #3 (clearing history) failed on Firefox and Safari. It seems that Firefox maintains links in history independent of the window.history object.

I plan to experiment a litle more with the History API to see if there's a way around it, but I appreciate any thoughts / tips you have which can save me some time! I'd also love to receive any general opinions / feedback / fact-checks. Just cold-email me at adilahmed31@gmail.com

Finally , here's a link to the website: [https://techclinic.cs.wisc.edu](https://techclinic.cs.wisc.edu). If you or someone close is experiencing Intimate-Partner Surveillance, please consult the resources linked on the website for more assistance. 


*Note: As I have graduated from UW-Madison, I no longer have formal affiliation with the Madison Tech Clinic. All views expressed are personal*