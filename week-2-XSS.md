# XSS vulnerability in google-gruyere web app 

## Problem statement: XSS vulnerabilities have far reaching impact and the implementation of a solution to prevent it must be carefully considered

During the class we learnt about the 3 different kinds of XSS:
Reflected
Stored
DOM based

Reflected XSS:
The script is reflected back to the user often through a manipulated URL, this can be used to target a specific user in a spear phishing attack.

Stored XSS:
The script is stored on the web page. Often this occurs in the comment sections of websites. Since the script is stored on the website many more users can be exposed to it including potentially even administrators.

DOM based:
DOM XSS targets the DOM running on the web app


I started by trying to break into the google-gruyere vulnerable web app (google-gruyere.appspot.com) using XSS.
The first page I saw after opening it was as followed:

[Evidence - main menu](./1.png)
 
Initially I wanted to find a search field so I could see if the URL was displaying the query as I know that is an attack vector for reflected xss. After browsing through the various pages I wasn’t able to find any search field (even when logged in). I did however notice that the URL was changing for each page and especially when I pressed on all snippets for either of the users the url would change to include the user
https://google-gruyere.appspot.com/555454490663195157123352352042061832226/snippets.gtl?uid=brie
 
[Evidence - user screen](./2.png)

I knew that the user is found using the “uid?=brie” field because when I changed it to the username of the other user I saw their page

https://google-gruyere.appspot.com/555454490663195157123352352042061832226/snippets.gtl?uid=cheddar
 
 [Evidence - new user screen](./3.png)

This time I tried to see what would happen if I entered a user that didn’t exist. I did this because I noticed that it showed the users name on the webpage and I know that if its showing the name on the webpage it provides an opportunity for reflected as the page would be processing user Input:
https://google-gruyere.appspot.com/555454490663195157123352352042061832226/snippets.gtl?uid=doesthepageshownames
 
 [Evidence - testing XSS](./4.png)
 
As can be seen on the page it displays any uid that I provide so I tried to then run a simple script to perform a XSS attack. I did this by changing the UID to “<script>alert(‘xss’)</script>”. This script wouldn’t cause anything malicious to happen however it would clearly display an alert prompt on the screen saying “xss” that would show to me that the webpage is processing and running the script.
 
 [Evidence - performing XSS](./5.png)
 
As can be seen I was able to perform a xss attack by manipulating the uid field of the URL. This means that I am able to have the webpage run any javascript that I input to the address field. In order for this attack to be successful I would need the victim to open open the webpage with my chosen javascript exploit in the uid field. The best way to do that would be to send a phising email to the chosen target providing them a link to the website. The issue with this is that people with more technical knowledge will notice a link that says “<script>alert(‘xss’)</script>” in the url. The attacker can obfuscate the url by using a url shortening service such as goo.gl or bit.ly which wouldn’t show what is in the url and it would instead automatically redirect to the malicious URL. 

Next I decided to try and pivot to perform a DOM based XSS, I wanted to try and steal the cookies belonging to the user as this would allow me to hijack the users session and if I were to perform this attack against an important stake holder such as an executive of the business it would allow me to impersonate them on the website, letting me view potentially sensitive information that can be used for blackmail or to further gain access to other places (eg I can access information that would be needed to login to the executives email or facebook). It could also give me the ability to make posts as the executive on their twitter or on a blog, the attacker could include negative information designed to make the share price drop. Another stakeholder that could be impacted by this is those with admin credentials such as the IT department; if they are targeted by this and the attacker is able to hijack their session the attacker could also potentially change the website to include malicious code that would impact all users of the webpage.

I added “<script>alert(document.cookie)</script>” to the uid to attempt a session hijacking and saw the following page:

[Evidence - DOM XSS](./6.png)
 
As we can see the DOM XSS was successful and it showed my username (test) however it didn’t provide enough information to hijack the users session. Although no session information was revealed the username alone potentially provides another attack vector as the attacker now knows what the username is of the victim it can be combined with a brute force attack to find the password or it can be used to search through data leaks to potentially find the password or other information such as ip, email or address. 

Having logged in to the webpage I can see I have an opportunity to submit my own “snippet” and the home page shows a list of the recent snippets. I decided to look into including a XSS attack in my snippet so that it would show up on the recent snippets section on the front page of the website. If I am successfully able to add an XSS attack to one of my snippets it would provide an very large reach for the exploit and could target even casual visitors who aren’t logged in and wouldn’t click on a link directing them to the website. When I selected “New Snippet” I saw the following page:
 
 [Evidence - Attempting stored XSS](./7.png)
 
Immediately I noticed that it says that there is support for limited HTML tags. As this webpage has a function similar to a blog or a forum I assumed that the <a> tag would work and that I could perform a stored XSS attack using this. I then added the following text to the snippet which would create the same alert prompt saying “XSS” when the user moved their mouse over the snippet post. 
<a onmouseover="alert('XSS')" href="LINK">DISPLAY_TEXT</a>

[Evidence - Stored XSS Success](./8.png)
 
As can be seen the snippet I added was shown in the recent snippets page and when I place my cursor over the snippet title it shows the “XSS” prompt. This means that this attack is able to target all visitors to the website, this can be used to steal cookies using the DOM XSS exploit that was used earlier. IT could also be used to redirect the user to a pharming website which looks like the real website but is actually a fake website controlled by the attack. This would mean that the user would use the attackers website like it was the real one, giving the attacker access to any data they type such as the usernames, passwords or credit card details. An XSS attack could potentially change the way the page displays information potentially defacing the website causing a negative reputation that can impact future business as well as diverting users (such as customers on a shopping website) away from the website. 

The next thing I wanted to try looking at was seeing how various websites protect themselves against XSS attacks. I decided to do this in line with the CERT-EU responsible disclosure policy that allows people to test EU websites provided that they follow rules regarding responsible disclosure. 

I tried various different EU institutions because I believed that they would have different implementations of protection from XSS attacks. The first webpage I tried was that belonging to the European parliament, when I put <script>alert(‘xss’)</script> into the search field the webpage provided a search with various articles being found. No prompt appeared however the URL included <script>alert%28%27xss%27%29<%2Fscript> 

[Evidence - European parliment](./9.png)
 


The next page I tried was that belonging to the European council on this page when I tried to enter <script>alert(‘xss’)</script> I was taken to a page telling me that I have been blocked for security 
 
 [Evidence - European council](./10.png)
 
I tried a number of things to circumvent this that didn’t work including:

* <ScRIpT>alert(‘xss’)</ScRIpT> = Same blocked page
* <BODY ONLOAD=alert('XSS')> = Infinite Captcha loop
* < > = Processed the search
* < test> = Generic error page

As can be seen it seems that the implementation of XSS security is to use filters designed to catch the most noticeable marks of XSS attacks (eg <script>, </script>, etc). However it does then depend on the organization to decide what happens when something is detected that fails to pass those filters. Either the website process can escape the XSS and process the request like a normal search like how the European parliaments does it or it can be completely blocked similar to how the European council does this. The benefit of outright blocking it is that it gives the attacker warning that they have been noticed and it can encourage them to stop where as processing the request can prevent normal requests from being blocked because of a strict filter.
