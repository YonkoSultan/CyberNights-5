## Taking a look
First when we enter the login page, we see a 2 options, either you `sign in` or `sign up`,
<br>
and at the very bottom there's ``Forgot password?`` link. 

![Screenshot (59)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/8bca6bf9-1c52-40e4-832e-e3f96bba909a)

However, only `Sign in` works here. If we try to pass any random values in the login form we will get a json response : `{"Error":"Wrong Credentials"}`

![Screenshot (58)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/4d57eb68-0026-47e9-9f4a-3e52eb650610)

## Time for work
Ok, let's move the POST request to burp, firstly I tried array injection but I got `400 Bad request` :

![Screenshot (62)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/e8db0192-9588-4f89-8206-2214056ab1fa)

If we test for SQL injection using a simple payload `'or 1=1 -- -` we get `Hacker detected` response, which indicate that the WAF ( Web application firewall ) has blocked our request :

![Screenshot (61)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/c67cad68-eaa2-47c4-8224-4f462a19a035)

After playing with the query to find out what is exactly getting blocked by WAF I found that it's blocking any spaces :

![Screenshot (63)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/57e31fe8-5f0b-4340-b5e5-d8cb20c1afcc)

## Bypassing WAF
That's why the challenge is named `Space`. Now we need to find a way to bypass the WAF which is blocking the space character.
So, when searching about `space block sql injection` we find an [interesting answer](https://security.stackexchange.com/a/127658) :

![Screenshot (64)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/4db04450-6b7b-49ea-9878-7bed8012c5a9)

So we need to replace the space with other whitespace character that isn't getting blocked by WAF. After trying various payloads it seems that both `%0d` (carriage return) & `%0a` (newline) doesn't get blocked. However, when we want to login using our new query we get the same error again `Hacker detected` :

![Screenshot (65)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/23664a12-1ce9-408e-959a-06c7ab30c5e7)

It seems that also `--` is getting blocked. How did I know? Simply, If you test for `--` separately you will get this response : 

![Screenshot (71)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/82e68c5e-43bc-438c-a9e6-7362d687d9a1)

it's generally a good way to check for characters that are getting blocked by WAF.

--------
Now, try to think..
<br>
How can you bypass this problem?
<br>
<br>
Similarly to what we did with the space character, we will search for [other ways to comment in SQL](https://portswigger.net/web-security/sql-injection/cheat-sheet) :

![Screenshot (72)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/f2f11291-f4c0-4f01-a8cc-539c20193607)

So if we try out these ways to comment in our query we find that `/*` works and finally we bypassed the WAF! : 

![Screenshot (66)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/457dca36-0e4a-44a2-a091-6fd905c15378)

Cool isn't it? But unfortunately nothing interesting here, except the `Logged in successfully` message,
<br>
so it might be `Blind SQL injection`.

## The final chapter

Now lets firstly test for `Blind SQL injection - Error based`. You can use this payload to guess the table name :

    'or (select 1 from table_name limit 0,1)=1 /*
    Don't forget to replace the spaces with %0a or %0d
<br>
And I got this result :
<br>

![Screenshot (70) - Copy](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/2ba54365-c448-45e1-84d5-22ea92b70442)

Great, now we know that there's a table named `users` let's get the `username`, `password` and `id` using this payload : 
    
    ' or substring((select column_name from table_name limit 0,1),(index number),1)='(letter)' /*

How did you know the column names? Simply by guessing!
<br>
<br>
Now you rather do this manually ( you will get Finger issues ) OR you can automate it using a simple Python script :

![Screenshot (67)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/615ff90e-8d27-4490-8f48-2676fce6db87)

After dumping the user data which are :
<br>
id : 1, username : Jackie_Lando, password : %_%MYS3cr3tPasSword%_%

We find no `flag` here, even if we tried to search for `flag` in the `users` table; we will get no result.

---

In CTF challenges, the `flag` is usually in a table named `flag` precisely in a column named `flag`.
<br>
Nevertheless, when testing the very first payload for checking for an existing table 

    'or (select 1 from table_name limit 0,1)=1 /*

we get no result for a table named `flag`; that means it doesn't exist. 
<br>
<br>
Finally, after guessing various names, we finally found the `flags` table :

![Screenshot (68)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/2f4b589d-fbc8-402e-ad9e-af73815add19)

Assuming that the column is named `flag`, we will use the previous script to dump the table. 
<br>
And we finally got the `flag`!

![Screenshot (69)](https://github.com/YonkoSultan/CyberNights-5/assets/107263975/68f4a5ac-f925-47a9-8443-6ec824c7fc6d)


