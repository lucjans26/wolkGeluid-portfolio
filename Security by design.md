## 1. Learning outcome
You investigate how to minimize security risks for your application, and you incorporate best practices in your whole software development process. 

## 2. Sonarcube 
To be implemented

## 3. Sonarlint
From the start of my development process, my IDE (phpStorm) has been equipped with [Sonarlint](https://www.sonarsource.com/products/sonarlint/). This free and open source extension helps identify and fix quality and security issues in real time during development. This makes sure that code is written cleanly and securely from the start and prevents having to change major issues in the future after a full scan. Besides just pointing out errors, it also explains why certain issues are issues in the first place so these can be avoided in the future.

![image](https://user-images.githubusercontent.com/46562627/171626458-90b1ec25-9292-4da4-b49a-d75690d82302.png)

## 4. Owasp top 10
The OWASP top 10 is a standard document that serves the purpose of creating awareness about the 10 most critical security issues in web applications. I created my own version of the document elaborating on what certain issues mean and how I (plan to) mitigate them.

## 5. Authentication and Authorization
The web application needs a way to register customers and a way to login. However this poses a major security risk due to data storage like; names, emails, adresses, passwords, etc. 
The best way to mitigate this issue is by using a good third party for authentication, especially for outside customers.
For authorisation Role-based access control is implemented. It creates systematic repeatable assignment of permissions, quick addition and changes of roles, intergration of third party user by using pre-defined roles, and easily auditable user privileges.

### 5.1 Authentication
The web application needs a way to register customers and a way to login. However this poses a major security risk due to data storage like; names, emails, adresses, passwords, etc. 
The best way to mitigate this issue is by using a good third party for authentication.
In my app I used Google as said third party.
That workflow would look something like this:

![image](https://developers.google.com/identity/protocols/oauth2/images/flows/implicit.png)

After logging in to the google cloud platform, I was easily able to create the credentials needed for the [Laravel Socialite driver](https://laravel.com/docs/9.x/socialite) driver:

![image](https://user-images.githubusercontent.com/46562627/171624018-4c02fccc-6028-433b-8f08-f1cdde7f4bbd.png)

After fumbling around with the callback URI for a bit, the third party login was functional:

![image](https://user-images.githubusercontent.com/46562627/171625068-f2a2dc16-d3ef-4924-8c8e-d74681268480.png)
![image](https://user-images.githubusercontent.com/46562627/171625197-f7bd2213-0a83-4ae8-ae69-6d7b17643774.png)

And the user is logged in or registered and recieves a [Laravel Sanctum accessToken](https://laravel.com/docs/9.x/sanctum) with the fitting scopes for the user:

![image](https://user-images.githubusercontent.com/46562627/171625743-4529d79c-7989-4a1c-8c18-ed9991083eb4.png)

Besides the customers, moderators also need an account. To keep this fully internal a custom authentication method is built. Just like a customer, a moderator receives a certain scope or scopes that allow access to certain endpoints.

### 5.2 Authorisation
As mentioned in the Authentication portion, the logged in user recieves a token. 


## 6. Reflection
