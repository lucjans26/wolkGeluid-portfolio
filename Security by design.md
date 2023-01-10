## 1. Learning outcome
You investigate how to minimize security risks for your application, and you incorporate best practices in your whole software development process. 

## 2. GDPR 
[GDPR](https://gdpr.eu/) is a law that protects the personal data and privacy of people in the EU and EEA, and also applies to the export of personal data outside of those regions. It gives people more control over their personal data and makes it easier for businesses to follow data protection rules within the EU. GDPR provides a complete [checklist](https://gdpr.eu/checklist/) which can be used to build a GDPR compliant system.

![image](https://user-images.githubusercontent.com/46562627/210109150-aaee9429-9049-4c52-8b6c-787fefa1fd94.png)

### 2.1 Right to be forgotten
[The right to erasure](https://gdpr-info.eu/art-17-gdpr/), also known as the "right to be forgotten," is part of the GDPR that gives individuals the right to request the deletion or removal of their personal data where there is no good reason for its continued existance. This right is important because it allows users to have more control over their personal data and to request that their data be erased when it is no longer necessary for the reason for which it was collected.

For example, if a user no longer wishes to use a particular service or website, they have the right to request that the company erase any personal data they have collected about them. This is particularly relevant in the context of online services, where personal data can be collected and stored indefinitely. The right to erasure helps ensure that personal data is not kept for longer than is necessary, and that individuals have the ability to request that their data be erased when they no longer want it to be used.

The right to be forgotten is a perfect example for implementation, because it runs throught the entire microservice architecture. 

### 2.2 Implementation
The implementation is quite convoluted but is mostly comes down the the following application flow:

![image](https://user-images.githubusercontent.com/46562627/210110487-0165ed1c-5792-4591-aea9-5504608b9dd4.png)

In the following [video](https://drive.google.com/file/d/1oCtyQiwhsjAHpnWZI2-28eisn8OgQvnW/view?usp=sharing), a demonstration is shown on the implementation of the right to be forgotten in my system.


## 3. Sonarlint
From the start of my development process, my IDE (phpStorm) has been equipped with [Sonarlint](https://www.sonarsource.com/products/sonarlint/). This free and open source extension helps identify and fix quality and security issues in real time during development. This makes sure that code is written cleanly and securely from the start and prevents having to change major issues in the future after a full scan. Besides just pointing out errors, it also explains why certain issues are issues in the first place so these can be avoided in the future.

![image](https://user-images.githubusercontent.com/46562627/171626458-90b1ec25-9292-4da4-b49a-d75690d82302.png)

## 4. Owasp top 10
The OWASP top 10 is a standard document that serves the purpose of creating awareness about the 10 most critical security issues in web applications. I created my own version of the document elaborating on what certain issues mean and how I (plan to) mitigate them.

## 5. Authentication and Authorization
The web application needs a way to register customers and a way to login. However this poses a major security risk due to data storage like; names, emails, adresses, passwords, etc. 
The best way to mitigate this issue is by using a good third party for authentication, especially for outside customers.
For authorisation Role-based access control (RBAC) is implemented.

### 5.1 Authentication
The web application needs a way to register customers and a way to login. However this poses a major security risk due to data storage like; names, emails, adresses, passwords, etc. 
The best way to mitigate this issue is by using a good third party for authentication.
In my app I used Google as said third party.

That workflow would look something like this:

![image](https://user-images.githubusercontent.com/46562627/191809734-66c3edff-a220-4374-8572-14e5532d2613.png)

After logging in to the google cloud platform, I was easily able to create the credentials needed for the [Laravel Socialite driver](https://laravel.com/docs/9.x/socialite):

![image](https://user-images.githubusercontent.com/46562627/191803868-6114039c-88c3-4a3d-a812-f1a9762d63d4.png)

After fumbling around with the callback URI for a bit, the third party login was functional:

![image](https://user-images.githubusercontent.com/46562627/191809529-eeb2965a-d215-4e48-baf9-2ff323596b06.png)
![image](https://user-images.githubusercontent.com/46562627/171625197-f7bd2213-0a83-4ae8-ae69-6d7b17643774.png)

And the user is logged in or registered and recieves a [Laravel Sanctum accessToken](https://laravel.com/docs/9.x/sanctum) with the fitting scopes for the user:

![image](https://user-images.githubusercontent.com/46562627/171625743-4529d79c-7989-4a1c-8c18-ed9991083eb4.png)

Besides the customers, moderators also need an account. To keep this fully internal a custom authentication method is built. Just like a customer, a moderator receives a certain scope or scopes that allow access to certain endpoints.

### 5.2 Authorisation
As mentioned in the Authentication portion, the logged in user recieves a token. This token is part of RBAC. It creates systematic repeatable assignment of permissions, quick addition and changes of roles, intergration of third party user by using pre-defined roles, and easily auditable user privileges. 
This is done in two parts: assigning the roles, and checking the roles. in this example a token is generated for a user which gives the user access to certain scopes; artist, album and music. This snippet was taken from OauthTrait.php where the keys for a user are made;
```php
return $newUser->createToken(IdTrait::requestTokenId(), ['artist', 'album', 'music'])->plainTextToken;
```

The second part is checking the roles. Within Routes/Api.php when trying to access the endpoint the scopes of the users are checked through the sanctum middleware;

```php
Route::get(ARTIST_ROUTE, [ArtistController::class, 'getArtist'])->middleware(['auth:sanctum', 'abilities:artist']);
```

For the current scale of this application, the token scopes and RBAC are sufficient. However in the future it would be essential to be prepared to deal with way more roles than the current application supports. This could be done in multiple ways. Within Laravel it could be handled within the controller which requires more logic which in turn is harder to upkeep. 

## 6. Risk Analysis
A risk analysis is a way to document internal and/or external risk factors within an organisation or project. In this analysis a chance of the risk becoming a reality is checked against  the possible effect this situation may cause. Based on this you can prioritize certain risks to mitigate.
After all of this a mitigation strategy or plan can be set up to determine how certai risks can be removed or reduced to acceptable levels.

### 6.1 Risks, impact and probability
As mentioned above, firstly the possible risks have to be determined.
![image](https://user-images.githubusercontent.com/46562627/198877604-adcd36ce-19f3-4073-b5dd-c05ab506e14a.png)

After establishing the risks, the impact and probability can be determined using a pre-determined scale.
![image](https://user-images.githubusercontent.com/46562627/198877676-526b9048-6116-410c-a108-a8ca52333a79.png)

### 6.2 Risk Level
Multiplying the impact with the probability will result in a risk level. This level can then be used to prioritize mitigations.
![image](https://user-images.githubusercontent.com/46562627/198877758-3791c57e-8ee1-4d67-a6c1-a485f367b6ea.png)
![image](https://user-images.githubusercontent.com/46562627/198877745-786ea1e9-339b-4833-a4b8-3d3a82d13c55.png)

### 6.3 Mitigation
Finally the mitigations for the threats can be determined.

![image](https://user-images.githubusercontent.com/46562627/198877791-dc9e0171-3a12-429a-bc5c-3cf52a35bbfb.png)


## 7. Reflection
I have learned a lot about designing a secure application and incorperating best practices in my development process.

Before the project started I created a risk analysis in which I establish possible vulnerabilities, the risks they pose, the risk level they have, and how I can prevent them by designing my application accordingly.

I have implemented the right to deletion which is part of the General Data Protection Regulation checklist which shows I know how to work throught the checklist so that I can create a fully compliant system (given the time and resources) in which risk of personal data leaking is minimized.

In addition I have used industry standard tool like SonarLint en Sonarcloud which help me scan my code for potential issues way before deployment. By adding Sonarcloud to my pipeline I can even prevent broken builds from even merging to master branches.

I have also made an OWASP top 10 document which shows I understand the most common security issues in web applications and how I mitigate these. Combined with an Acunetix test report I can check whether my application specifically has any vulnerabilities. By creating a mitigation report I can explain what the outstanding vulnerabilities are and how I can mitigate those.

Finally I also created an authentication service which uses Google Oauth to minimize security vulnerabilities I could introduce by bulding it myself. This combined with Role Based Access Control prevents unauthorized users from using my application

Althought there is a lot to be learned on the side of authorisation I do believe that the system that I built is a great foundation to build on in the future. This realisation, combined with the new concepts I have used (like OAuth, RBAC, risk analysis and GDPR), I believe that if I focus more on learning about authorisation, that in the future I can create more secure applications that implement best practices.

