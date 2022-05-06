Hello hackers, I hope [TT1]you are doing well. In this writeup I will be talking about two of my findings that I pursued in the last couple of months.

The first one was misconfigured Google Site in which, I accidentally got access to one of their internal KB instances, it had many google sheets, username, password, meeting schedules, etc.

The second was a simple Unauthenticated IDOR, in which I was able to delete anybody’s account. So, without further delay, let’s get started.

# A misconfigured Google Sites instance

### What is Google Sites?

[Google Sites](https://gsuite.google.com/products/sites/) is a website-building platform from Google. If you're familiar with other website platforms like WordPress or Wix, you can think of Google Sites as something that's similar, but perhaps more specialized for businesses and web-based teams.

So a typical google sites instance looks like **[https://sites.google.com/company.com](https://sites.google.com/company.com).** 

So, while hacking on one the private program on Bugcrowd, let’s call it **private.com** I came across a Google Sites instance, which was publicly accessible at **https://sites.google.com/private.com/privatekb**

when I clicked on that, first I thought the instance is public, but after finding some internal stuff in there, I got access to a lot of their Google Sheets, username password, their internal work flows etc.  quickly I made a report, and very next day the program took down that Google Sites instance and the Vulnerability was fixed.

Now here comes the interesting part of this finding - there were certain google drives that were private. So, I thought I should try my luck and request for the access. Next day, after waking up I checked my email and surprisingly one of their employees had granted me access to their Google Drive. 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1eec1256-3cb0-4d4a-8c9f-dad00f1db9a6/Untitled.png)

As soon as I saw this email, I quickly turned on my Mac, and started exploring the google drive. The said Gdrive had lots of internal contents like their meeting schedule for the whole year (2021), what they’ve discussed, what’s their planning for 2022. The Gdrive also contained, hundreds of Google sheets, Zip files, etc.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/98ba9310-34aa-4e46-84bb-38116fd1f62d/Untitled.png)

## A simple IDOR that led me to delete anybody’s account

I was onboarded to a new program on Synack and Upon visiting the scope URL, it turned out to be an ecommerce application that had two roles. 

1. Seller
2. User

The seller had permission to invite new users. So, I tried to test this feature as it looked interesting. 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/61579eb2-d329-4dbd-b259-37db596024ad/Untitled.png)

I quickly sent an invitation to my email id and check the response in my Burp proxy. I saw that, the sent invitation response contained a numeric parameter named `user_number`

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a442dae-0901-4961-ae34-2cb875b130ca/Untitled.png)

To check  whether the userID is incremental or not, I sent another invitation and as suspected it was in the incremental order. So, if the previous ID was 6865834, then the next ID would be 6865835, 6865836 and so on.

The first thing that came to my mind was to test for IDOR on the delete account feature. To do that, I quickly intercepted the delete user request and sent it to Burp Repeater.

Next, I decided to remove the cookie header completely and check whether it works or not, and yes it was working. So, any attacker can delete anyone’s account without being authenticated to the application.

But the problem was that this request had a **X-Csrf-token** header, which contained a random CSRF token in the request. Without that header, the response was 403 forbidden.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/41a752e3-bea9-4ef1-b4ed-8e0223e36f91/Untitled.png)

After a while I figured out that, there is no need to put an actual CSRF token. You just have to put any Random string like ABCD in the header, so you can put like **X-Csrf-Token-ABCD.** and it would get accepted

The final request for the account deletion was

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9f33b7a7-4a17-4277-aa46-a61b4f7bda4f/Untitled.png)

The impact was critical as I was able to delete anyone’s account with just a simple intruder attack. The bug was fixed within few hours, and I got the PV from the client.

 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9609f581-c6e2-4e8f-9ba1-92439dca6168/Untitled.png)


[TT2]
[TT1]An introduction to the said topic?
[TT2]Can you add a little bit of conclusion?
