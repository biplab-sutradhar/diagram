POST /auth/signup - Register New User
```mermaid

flowchart TD
%% API Flow: POST /auth/signup
Start[POST /auth/signup<br/>Request Body: name + email + password] --> ValidateInput[Validate Request Body]

ValidateInput --> InputValid{Input Valid?}
InputValid -->|No| Input422[Return 422 Invalid Input Data]
InputValid -->|Yes| CheckEmail[Check if Email Exists]

CheckEmail --> EmailExists{Email Already Exists?}
EmailExists -->|Yes| Email400[Return 400 Email Already Exists]
EmailExists -->|No| CreateSubscription[Create New Subscription]

CreateSubscription --> SubCreated{Subscription Created?}
SubCreated -->|No| Server500[Return 500 Internal Server Error]
SubCreated -->|Yes| CreateUser[Create New User]

CreateUser --> UserCreated{User Created?}
UserCreated -->|No| Server500
UserCreated -->|Yes| CreateToken[Create Verification Token]

CreateToken --> TokenCreated{Token Created?}
TokenCreated -->|No| Success201[Return 201 Something Went Wrong]
TokenCreated -->|Yes| SendEmail[Send Verification Email]

SendEmail --> Success200[Return 200 OK]

classDef startEnd fill:#81C8FF,stroke:#4682B4,stroke-width:2px,color:#000;
classDef decision fill:#FFD54F,stroke:#FFB300,stroke-width:2px,color:#000;
classDef success fill:#A5D6A7,stroke:#388E3C,stroke-width:2px,color:#000;
classDef error fill:#EF9A9A,stroke:#D32F2F,stroke-width:2px,color:#000;
classDef warning fill:#FFCC80,stroke:#F57C00,stroke-width:2px,color:#000;

class Start,Success200 startEnd
class InputValid,EmailExists,SubCreated,UserCreated,TokenCreated decision
class Success200 success
class Input422,Email400,Server500 error
class Success201 warning

```

POST /auth/signin - User Login
```mermaid
flowchart TD
%% API Flow: POST /auth/signin
Start[POST /auth/signin<br/>Request Body: email + password] --> GuestCheck{Guest User?}

GuestCheck -->|Yes| GetGuestUser[Get Guest User Data]
GuestCheck -->|No| ValidateInput[Validate Request Body]

ValidateInput --> InputValid{Input Valid?}
InputValid -->|No| Input422[Return 422 Invalid Input Data]
InputValid -->|Yes| CheckUser[Check if User Exists]

CheckUser --> UserExists{User Exists?}
UserExists -->|No| User404[Return 404 Incorrect Email/Password]
UserExists -->|Yes| CheckVerified{Email Verified?}

CheckVerified -->|No| Verify403[Return 403 Email Not Verified]
CheckVerified -->|Yes| VerifyPassword[Match Password]

VerifyPassword --> PasswordMatch{Password Matches?}
PasswordMatch -->|No| User404
PasswordMatch -->|Yes| SetCookie[Set JWT Cookie]

GetGuestUser --> SetCookie
SetCookie --> Success200[Return 200 OK]

classDef startEnd fill:#81C8FF,stroke:#4682B4,stroke-width:2px,color:#000;
classDef decision fill:#FFD54F,stroke:#FFB300,stroke-width:2px,color:#000;
classDef success fill:#A5D6A7,stroke:#388E3C,stroke-width:2px,color:#000;
classDef error fill:#EF9A9A,stroke:#D32F2F,stroke-width:2px,color:#000;
classDef warning fill:#FFCC80,stroke:#F57C00,stroke-width:2px,color:#000;

class Start,Success200 startEnd
class GuestCheck,InputValid,UserExists,CheckVerified,PasswordMatch decision
class Success200 success
class Input422,User404,Verify403 error

```

POST /auth/signout - User Logout
```mermaid
flowchart TD
%% API Flow: POST /auth/signout
Start[POST /auth/signout] --> CheckCookie{JWT Cookie Present?}

CheckCookie -->|No| Cookie400[Return 400 Session Failed]
CheckCookie -->|Yes| ClearCookie[Clear JWT Cookie]

ClearCookie --> Success205[Return 205 Reset Content]

classDef startEnd fill:#81C8FF,stroke:#4682B4,stroke-width:2px,color:#000;
classDef decision fill:#FFD54F,stroke:#FFB300,stroke-width:2px,color:#000;
classDef success fill:#A5D6A7,stroke:#388E3C,stroke-width:2px,color:#000;
classDef error fill:#EF9A9A,stroke:#D32F2F,stroke-width:2px,color:#000;
classDef warning fill:#FFCC80,stroke:#F57C00,stroke-width:2px,color:#000;

class Start,Success205 startEnd
class CheckCookie decision
class Success205 success
class Cookie400 error

```

GET /auth/verify/:token - Email Verification
```mermaid
flowchart TD
%% API Flow: GET /auth/verify/:token
Start[GET /auth/verify/:token<br/>Param: token] --> ValidateToken{Token Present?}

ValidateToken -->|No| Token422[Return 422 Invalid Token]
ValidateToken -->|Yes| FetchToken[Fetch User Token]

FetchToken --> TokenExists{Token Exists?}
TokenExists -->|No| CheckDeleted[Check Deleted Token]
TokenExists -->|Yes| CheckExpiry{Token Expired?}

CheckDeleted --> DeletedExists{Deleted Token Found?}
DeletedExists -->|Yes| Already200[Return 200 Already Verified]
DeletedExists -->|No| Token400[Return 400 Invalid Token]

CheckExpiry -->|Yes| Expired403[Return 403 Expired Token]
CheckExpiry -->|No| GetUser[Get User Data]

GetUser --> UserExists{User Exists?}
UserExists -->|No| User404[Return 404 Invalid Token]
UserExists -->|Yes| AlreadyVerified{Already Verified?}

AlreadyVerified -->|Yes| DeleteToken[Delete Token]
DeleteToken --> Already200
AlreadyVerified -->|No| VerifyUser[Verify User Account]

VerifyUser --> VerifySuccess{Verification Successful?}
VerifySuccess -->|No| Verify500[Return 500 Verification Failed]
VerifySuccess -->|Yes| DeleteTokenSuccess[Delete Used Token]

DeleteTokenSuccess --> DeleteSuccess{Delete Successful?}
DeleteSuccess -->|No| Server500[Return 500 Internal Server Error]
DeleteSuccess -->|Yes| Success200[Return 200 Email Verified Successfully]

classDef startEnd fill:#81C8FF,stroke:#4682B4,stroke-width:2px,color:#000;
classDef decision fill:#FFD54F,stroke:#FFB300,stroke-width:2px,color:#000;
classDef success fill:#A5D6A7,stroke:#388E3C,stroke-width:2px,color:#000;
classDef error fill:#EF9A9A,stroke:#D32F2F,stroke-width:2px,color:#000;
classDef warning fill:#FFCC80,stroke:#F57C00,stroke-width:2px,color:#000;

class Start,Success200,Already200 startEnd
class ValidateToken,TokenExists,DeletedExists,CheckExpiry,UserExists,AlreadyVerified,VerifySuccess,DeleteSuccess decision
class Success200,Already200 success
class Token422,Token400,Expired403,User404,Verify500,Server500 error

```

POST /auth/password/forgot - Initiate Password Reset
```mermaid

flowchart TD
%% API Flow: POST /auth/password/forgot
Start[POST /auth/password/forgot<br/>Request Body: email] --> ValidateInput[Validate Request Body]

ValidateInput --> InputValid{Input Valid?}
InputValid -->|No| Input422[Return 422 Invalid Input Data]
InputValid -->|Yes| CheckUser[Check if User Exists]

CheckUser --> UserExists{User Exists?}
UserExists -->|No| User404[Return 404 Email Doesn't Exist]
UserExists -->|Yes| CreateToken[Create Forgot Password Token]

CreateToken --> TokenCreated{Token Created?}
TokenCreated -->|No| Server500[Return 500 Internal Server Error]
TokenCreated -->|Yes| SendEmail[Send Reset Email]

SendEmail --> Success200[Return 200 OK]

classDef startEnd fill:#81C8FF,stroke:#4682B4,stroke-width:2px,color:#000;
classDef decision fill:#FFD54F,stroke:#FFB300,stroke-width:2px,color:#000;
classDef success fill:#A5D6A7,stroke:#388E3C,stroke-width:2px,color:#000;
classDef error fill:#EF9A9A,stroke:#D32F2F,stroke-width:2px,color:#000;
classDef warning fill:#FFCC80,stroke:#F57C00,stroke-width:2px,color:#000;

class Start,Success200 startEnd
class InputValid,UserExists,TokenCreated decision
class Success200 success
class Input422,User404,Server500 error


```

GET /auth/password/forgot/:token - Get Reset Session
```mermaid

flowchart TD
%% API Flow: GET /auth/password/forgot/:token
Start[GET /auth/password/forgot/:token<br/>Param: token] --> ValidateToken{Token Present?}

ValidateToken -->|No| Token422[Return 422 Invalid Token]
ValidateToken -->|Yes| FetchToken[Fetch User Token]

FetchToken --> TokenExists{Token Exists?}
TokenExists -->|No| User404[Return 404 User Not Found]
TokenExists -->|Yes| CheckExpiry{Token Expired?}

CheckExpiry -->|Yes| Expired403[Return 403 Expired Token]
CheckExpiry -->|No| SetCookie[Set Reset Session Cookie]

SetCookie --> Success200[Return 200 OK]

classDef startEnd fill:#81C8FF,stroke:#4682B4,stroke-width:2px,color:#000;
classDef decision fill:#FFD54F,stroke:#FFB300,stroke-width:2px,color:#000;
classDef success fill:#A5D6A7,stroke:#388E3C,stroke-width:2px,color:#000;
classDef error fill:#EF9A9A,stroke:#D32F2F,stroke-width:2px,color:#000;
classDef warning fill:#FFCC80,stroke:#F57C00,stroke-width:2px,color:#000;

class Start,Success200 startEnd
class ValidateToken,TokenExists,CheckExpiry decision
class Success200 success
class Token422,User404,Expired403 error


```

PATCH /auth/password/reset - Reset Password

```mermaid
flowchart TD
%% API Flow: PATCH /auth/password/reset
Start[PATCH /auth/password/reset<br/>Request Body: token + newPassword] --> CheckSession{Reset Session Cookie?}

CheckSession -->|No| Session401[Return 401 Verification Failed]
CheckSession -->|Yes| ValidateInput[Validate Request Body]

ValidateInput --> InputValid{Input Valid?}
InputValid -->|No| Input422[Return 422 Invalid Input Data]
InputValid -->|Yes| GetUser[Get User Data]

GetUser --> UpdatePassword[Update User Password]
UpdatePassword --> UpdateSuccess{Update Successful?}

UpdateSuccess -->|No| Password400[Return 400 Password Failed]
UpdateSuccess -->|Yes| ValidateToken[Validate Provided Token]

ValidateToken --> TokenValid{Token Valid?}
TokenValid -->|No| Token403[Return 403 Password Failed]
TokenValid -->|Yes| DeleteToken[Delete Used Token]

DeleteToken --> Success200[Return 200 OK]

classDef startEnd fill:#81C8FF,stroke:#4682B4,stroke-width:2px,color:#000;
classDef decision fill:#FFD54F,stroke:#FFB300,stroke-width:2px,color:#000;
classDef success fill:#A5D6A7,stroke:#388E3C,stroke-width:2px,color:#000;
classDef error fill:#EF9A9A,stroke:#D32F2F,stroke-width:2px,color:#000;
classDef warning fill:#FFCC80,stroke:#F57C00,stroke-width:2px,color:#000;

class Start,Success200 startEnd
class CheckSession,InputValid,UpdateSuccess,TokenValid decision
class Success200 success
class Session401,Input422,Password400,Token403 error



```

GET /auth/validate-session - Validate Session
```mermaid

flowchart TD
%% API Flow: GET /auth/validate-session
Start[GET /auth/validate-session] --> Auth{Authorized?}

Auth -->|No| Auth401[Return 401 Unauthorized]
Auth -->|Yes| Success200[Return 200 OK + User Data]

classDef startEnd fill:#81C8FF,stroke:#4682B4,stroke-width:2px,color:#000;
classDef decision fill:#FFD54F,stroke:#FFB300,stroke-width:2px,color:#000;
classDef success fill:#A5D6A7,stroke:#388E3C,stroke-width:2px,color:#000;
classDef error fill:#EF9A9A,stroke:#D32F2F,stroke-width:2px,color:#000;
classDef warning fill:#FFCC80,stroke:#F57C00,stroke-width:2px,color:#000;

class Start,Success200 startEnd
class Auth decision
class Success200 success
class Auth401 error


```

```mermaid

```
