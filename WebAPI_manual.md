# QUBO solver Web API manual 

- [QUBO solver Web API manual](#qubo-solver-web-api-manual)
- [Outline](#outline)
- [QUBO problem and JSON data format.](#qubo-problem-and-json-data-format)
  - [QUBO problem](#qubo-problem)
    - [Example of QUBO problem](#example-of-qubo-problem)
  - [JSON data format of QUBO matrices](#json-data-format-of-qubo-matrices)
- [QUBO solutions and JSON data format](#qubo-solutions-and-json-data-format)
- [Basic Operations of the Web API](#basic-operations-of-the-web-api)
  - [Service endpoints of the Web API](#service-endpoints-of-the-web-api)
  - [Status codes returned by the Web API](#status-codes-returned-by-the-web-api)
  - [Status check](#status-check)
- [The basic flow of the Web API usage](#the-basic-flow-of-the-web-api-usage)
  - [User registration](#user-registration)
  - [Access token retrieval](#access-token-retrieval)
  - [Solving QUBO problems](#solving-qubo-problems)
  - [Supported methods and operations of URIs.](#supported-methods-and-operations-of-uris)
- [User registration](#user-registration-1)
- [Account management](#account-management)
  - [Getting registered user information](#getting-registered-user-information)
  - [Different ways to submit key-values](#different-ways-to-submit-key-values)
    - [Query parameters](#query-parameters)
    - [JSON data format](#json-data-format)
  - [Getting new password](#getting-new-password)
  - [Getting username](#getting-username)
  - [Changing password](#changing-password)
  - [Deleting user account](#deleting-user-account)
- [Retrieving access token](#retrieving-access-token)
- [Posting and managing QUBO matrices](#posting-and-managing-qubo-matrices)
  - [Posting a QUBO matrix](#posting-a-qubo-matrix)
  - [Alternative way for posting a QUBO matrix](#alternative-way-for-posting-a-qubo-matrix)
    - [Example](#example)
  - [Getting QUBO matrix information](#getting-qubo-matrix-information)
  - [Getting a list of problems](#getting-a-list-of-problems)
  - [Deleting a QUBO matrix](#deleting-a-qubo-matrix)
  - [Deleting all QUBO matrices](#deleting-all-qubo-matrices)
- [Job registration and management](#job-registration-and-management)
  - [Posting a job](#posting-a-job)
  - [Job parameters](#job-parameters)
  - [Getting job information](#getting-job-information)
  - [Getting a list of all jobs](#getting-a-list-of-all-jobs)
  - [Deleting a job](#deleting-a-job)
  - [Deleting all unexecuted jobs](#deleting-all-unexecuted-jobs)
- [Getting and managing solutions](#getting-and-managing-solutions)
  - [Getting a lit of all solutions](#getting-a-lit-of-all-solutions)
  - [Getting a solution](#getting-a-solution)
  - [Deleting a solution](#deleting-a-solution)
  - [Deleting all solutions](#deleting-all-solutions)

# Outline
* This Web service provides API access for solving QUBO problems using GPUs.
* Users can upload QUBO matrices and jobs for solving them.
* The QUBO solver finds solutions for problems specified by submitted jobs. 
* The format of QUBO matrices and solutions is JSON (JavaScript Object Notation).

# QUBO problem and JSON data format.

## QUBO problem
* Quadratic Unconstrained Binary Optimization problem
* Input: an $n\times n$ QUBO matrix $W=(W_{i,j})$ $(0\leq i,j\leq n-1)$．All elements must be integers.
* Output: $n$-bit vector $X=(x_i)$ $(0\leq i\leq n-1)$．
* Objective function: The energy $E(X)=\sum_{0\leq i,j\leq n-1}W_{i,j}x_ix_j$ of $X$.
* A QUBO problem is a problem to find an $n$-bit vector $X$ with minimum energy $E(X)$ over all $2^n$ vectors $X$.
* We call QUBO problems with this definition **base** 0, because the indices of $W$ and $X$ start from 0.
* The base is 0 if these indices start from 1 such that $i$ and $j$ take value in $[1,n]$.  
* This Web API accepts QUBO problems with base 0 and 1.

### Example of QUBO problem
* Example of QUBO problem of size $5\times 5$
  
$
W=
\begin{pmatrix} 
2 & 2 & -3& 0& 4&\\ 
 & 1 & -2& -2& 0&\\ 
 &  & -2& -4& 0&\\ 
 &\text{\huge{0}}  & & 5& 1&\\ 
 &  & & & -4& 
\end{pmatrix}
$
* The optimal solution is $X=[0,1,1,0,1]$ with energy $E(X)=-7$.

## JSON data format of QUBO matrices
* It has three key-values below.

| key    | value                                                  | limitation                    |
| ------ | ------------------------------------------------------ | ----------------------------- |
| "nbit" | The number $n$ of bits                                 | must be from 32 to 64k(65536) |
| "base" | The base of the QUBO problem                           | 0 or 1                        |
| "qubo" | a list of all non-zero elements of the QUBO matrix $Q$ |                               |

* Elements of "qubo" are 3-tuples [$i$,$j$,$W_{i,j}$].
* For example, the QUBO problem $W$ above is represented by JSON data format as follows:
```JSON
{
  "nbit":5,
  "base":0,
  "qubo":[
    [0,0,2],
    [0,1,2],
    [0,2,-3],
    [0,4,4],
    [1,1,1],
    [1,2,-2],
    [1,3,-2],
    [2,2,-2],
    [2,3,-4],
    [3,3,5],
    [3,4,1],
    [4,4,-4]
  ]
}
```
* If the JSON data format includes keys other than "nbit", "base", and "qubo", the Web API ignores them.
* QUBO matrix files with JSON data format must have file name extension ".json".
* This Web API service accepts QUBO matrix files compressed by gzip with file name extension ".json.gz".

# QUBO solutions and JSON data format
* Solutions of QUBO problems given by this Web API service are JSON data format.
* For example, a solution of the QUBO problem $W$ above is represented by the JSON data format as follows:
```JSON
{
    "energy": -7,
    "solution": [0,1,1,0,1]
}
```
* Key "solution" has the bit values of $n$-bit solution vector as a list of $n$ 0/1 integers.
* Additional keys such as "tts" (Time-To-Solution) may be included.

# Basic Operations of the Web API
* User registration, QUBO matrices upload, job registration, and solution vector retrieval can be done by sending HTTP requests to service endpoints of the Web API.
  
## Service endpoints of the Web API
* The service endpoints have the following format: 
```
https://qubosolver.cs.hiroshima-u.ac.jp/v1/{directory}
```

The Web API provides the following service endpoints:
* Status check(**root**)： https://qubosolver.cs.hiroshima-u.ac.jp/v1/
* User registration(**signup**)：https://qubosolver.cs.hiroshima-u.ac.jp/v1/signup
* Account management(**account**)：https://qubosolver.cs.hiroshima-u.ac.jp/v1/account
* Access token issue(**token**)：https://qubosolver.cs.hiroshima-u.ac.jp/v1/token
* QUBO matrix registration(**problems**)：https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems
* Job registration(**jobs**)：https://qubosolver.cs.hiroshima-u.ac.jp/v1/jobs
* Solution download(**solutions**)：https://qubosolver.cs.hiroshima-u.ac.jp/v1/solutions

## Status codes returned by the Web API
* The Web API returns the following HTTP status codes for access requests.

| Status codes                | success/failure | description                                            |
| --------------------------- | :-------------: | ------------------------------------------------------ |
| PROCESSING(102)             |     failure     | not started due to QUBO matrix verification in process |
| OK(200)                     |     success     | request completed correctly                            |
| CREATED(201)                |     success     | user registration completed correctly                  |
| ACCEPTED(202)               |     success     | file upload completed correctly and processing started |
| BAD_REQUEST(400)            |     failure     | malformed parameters                                   |
| UNAUTHORIZED(401)           |     failure     | wrong password or wrong access token                   |
| NOT_FOUND(404)              |     failure     | wrong URI or no resource (user or file)                |
| CONFLICT(409)               |     failure     | the same registered username or email found            |
| UNSUPPORTED_MEDIA_TYPE(415) |     failure     | non-JSON(.json or .json.gz) files uploaded             |
| INTERNAL_SERVER_ERROR(500)  |     failure     | unexpected error of the Web API server                 |
| SERVICE_UNAVAILABLE(503)    |     failure     | QUBO solver not running                                |

## Status check
One can see if QUBO solver is working now by the following cURL command.
```BASH
$  curl -X GET https://qubosolver.cs.hiroshima-u.ac.jp/v1/
```
The WEB API returns the following message if the QUBO solver is working:
```JSON
{
    "message": "QUBO solver is working",
    "active": true,
    "jobs_in_queue": 5,
    "total_time_limit": 1200,
    "uri_root": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/",
    "uri_signup": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/signup",
    "uri_account": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/account",
    "uri_token": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/token",
    "uri_problems": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems",
    "uri_jobs": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/jobs",
    "uri_solutions": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/solutions"
}
``` 
* "jobs_in_queue" is the total number of jobs to be executed in the queue.
* "total_time_limit" is the total time_limit of all jobs in the queue. If a new job is submitted then it must be wait at least "total_time_limit" seconds for execution.

HTTP status codes
  
| status codes             | success/failure | explanation                |
| ------------------------ | :-------------: | -------------------------- |
| OK(200)                  |     success     | QUBO solver is working     |
| SERVICE_UNAVAILABLE(503) |     failure     | QUBO solver is not working |


# The basic flow of the Web API usage

## User registration 
Users must signup to use the Web API service
1. **signup**: register user and receive password by email. The email will be sent from email address qubosolver@gmail.com.
2. **account**: register a new password using the received password.
   
## Access token retrieval
Retrieve access token to access URI directories problems, jobs, and solutions.
1. **token**: retrieve an access token from username and password.

The access token has an expiration time. If it is expired, users must obtain a new access token.
## Solving QUBO problems
Using access token authentication, one can access/operate files in three directories "problems", "jobs", and "solutions" to solve QUBO problems．
1. **problems**: register QUBO matrices of QUBO problems.
2. **jobs**: submit jobs for solving QUBO problems.
3. **solutions**: read solutions obtained by the QUBO solver, which solves QUBO problems according to the submitted jobs


## Supported methods and operations of URIs.

| URI directory    | method | operations                       | access token authentication |
| ---------------- | ------ | -------------------------------- | :-------------------------: |
| signup           | POST   | create user account              |                             |
| account          | POST   | get user account information     |                             |
| account          | PUT    | change password                  |                             |
| account          | DELETE | delete user account              |              x              |
| token            | POST   | retrieve access token            |                             |
| problems         | GET    | get list of all problems         |              x              |
| problems         | POST   | post a QUBO matrix               |              x              |
| problems         | DELETE | delete all QUBO matrices         |              x              |
| problems/{file}  | GET    | get information of a QUBO matrix |              x              |
| problems/{file}  | DELETE | delete a QUBO matrix             |              x              |
| jobs             | GET    | get list of unexecuted jobs      |              x              |
| jobs             | POST   | post a job                       |              x              |
| jobs             | DELETE | delete all unexecuted jobs       |              x              |
| jobs/{file}      | GET    | get an unexecuted job            |              x              |
| jobs/{file}      | DELETE | delete an unexecuted job         |              x              |
| solutions        | GET    | get a list of all solutions      |              x              |
| solutions        | DELETE | delete all solutions             |              x              |
| solutions/{file} | GET    | get a solution                   |              x              |
| solutions/{file} | DELETE | delete a solution                |              x              |

* We can think that "problems", "jobs", and "solutions" are directories of a file system and files are stored in these directories.
* Files with JSON data format (.json) are stored in directories "jobs" and "solutions".
* In addition to JSON data format (.json), directory "problems" can store gzip-compressed files with JSON data format (.json.gz).

# User registration
* URI: https://qubosolver.cs.hiroshima-u.ac.jp/v1/signup
* User account is created by submitting the following key-values.
  
| Key         | Value                   | limitation                               | example                      |
| ----------- | ----------------------- | ---------------------------------------- | ---------------------------- |
| username    | User name               | alphanumeric string with 5 to 32 letters | nakano                       |
| email       | Email address           | up to 256 letters                        | nakano@ cs.hiroshima-u.ac.jp |
| firstname   | First name              | up to 32 letters                         | Koji                         |
| lastname    | Last name               | up to 32 letters                         | Nakano                       |
| affiliation | university/company name | up to 64 letters                         | Hiroshima University         |

* After user registration is completed, password will be sent by email from qubosolover@gmail.com.
* For example, user registration can be done by the following cURL command:
```BASH
$ curl -X POST https://qubosolver.cs.hiroshima-u.ac.jp/v1/signup \
  -d "username=nakano" \
  -d "email=nakano@cs.hiroshima-u.ac.jp" \
  -d "firstname=Koji" \
  -d "lastname=Nakano" \
  -d "affiliation=Hiroshima University"
```
*  The Web API returns the following response if successful
```JSON
{
    "message": "<password> emailed"
}
``` 
* HTTP status codes
  
| status codes     | success/failure | description                              |
| ---------------- | :-------------: | ---------------------------------------- |
| CREATED(201)     |     success     | user account creation succeeded          |
| BAD_REQUEST(400) |     failure     | malformed parameters                     |
| CONFLICT(409)    |     failure     | username and/or email already registered |

# Account management
* URI: https://qubosolver.cs.hiroshima-u.ac.jp/v1/account
* Get user information
* Get username or password if they are lost
* Change password
  
| URI directory | Method | Key-values                      | Operation                                                            |
| ------------- | ------ | ------------------------------- | -------------------------------------------------------------------- |
| account       | POST   | username, password              | get registered user infomation(email,firstname,lastname,affiliation) |
| account       | POST   | username, email                 | new password emailed                                                 |
| account       | POST   | email                           | get username                                                         |
| account       | PUT    | username，password，newpassword | change password                                                      |

## Getting registered user information
* User account information can be retrieved by submitting username and password as follows.
```BASH
$ curl -X POST https://qubosolver.cs.hiroshima-u.ac.jp/v1/account \
  -d "username=nakano" \
  -d "password=xxxxxxxx"
```
* xxxxxxxx must be replaced with registered password.
* The Web API returns the following response.
```JSON
{
  "username": "nakano",
  "email": "nakano@cs.hiroshima-u.ac.jp",
  "firstname": "Koji",
  "lastname": "Nakano",
  "affiliation": "Hirohsima University",
  "jobs": 0
}
```
* The value of key "jobs" is the number of jobs submitted by the user so far.
* HTTP status codes
  
| status codes      | success/failure | description                              |
| ----------------- | :-------------: | ---------------------------------------- |
| OK(200)           |     success     | user information retrieved correctly     |
| BAD_REQUEST(400)  |     failure     | malformed parameters                     |
| UNAUTHORIZED(401) |     failure     | wrong password                           |
| NOT_FOUND(404)    |     failure     | no user account associated with username |

## Different ways to submit key-values
* This Web API accepts requests with query parameters and/or JSON data format.
* For example, user information can be retrieved by the following cURL commands:
### Query parameters
```BASH
$ curl -X POST "https://qubosolver.cs.hiroshima-u.ac.jp/v1/account?username=nakano&passwxxxxxxxx`
```
* Note that this method is not recommended because password in the query parameter can be recorded/cached in the client and/or the server.


### JSON data format
```BASH
$ curl -X POST https://qubosolver.cs.hiroshima-u.ac.jp/v1/account \
  -H "Content-type: application/json" \
  -d '{"username":"nakano","password":"xxxxxxxx"}```
```
## Getting new password
* If password is lost, new password can be obtained by email from registered username and email as follows:
```BASH
$ curl -X POST https://qubosolver.cs.hiroshima-u.ac.jp/v1/account \
  -d "username=nakano" \
  -d "email=nakano@cs.hiroshima-u.ac.jp"
```
* The Web API returns the following response and sends new password.
```JSON
{
    "message": "new <password> emailed"
}
```
* HTTP status codes

| status codes      | success/failure | description                      |
| ----------------- | :-------------: | -------------------------------- |
| OK(200)           |     success     | new password is emailed          |
| UNAUTHORIZED(401) |     failure     | email is wrong                   |
| NOT_FOUND(404)    |     failure     | no user associated with username |

## Getting username
* If username is lost, it can be retrieved by the registered email as follows:
```BASH
$ curl -X POST https://qubosolver.cs.hiroshima-u.ac.jp/v1/account \
  -d "email=nakano@cs.hiroshima-u.ac.jp"
```
* The Web API returns the following response with username.
```JSON
{
    "message": "found user account associated with <email>",
    "username": "nakano"
}
```
* HTTP status codes
  
| status codes     | success/failure | description                           |
| ---------------- | :-------------: | ------------------------------------- |
| OK(200)          |     success     | retrieve username                     |
| BAD_REQUEST(400) |     failure     | malformed parameters                  |
| NOT_FOUND(404)   |     failure     | no user account associated with email |

## Changing password
* Password is updated by newpassword by the cURL command as follows:
```BASH
$ curl -X PUT https://qubosolver.cs.hiroshima-u.ac.jp/v1/account \
  -d "username=nakano" \
  -d "password=xxxxxxxx" \
  -d "newpassword=xxxxxxxx"
```
* Note that the method is PUT. 
* The Web API returns the following response if successful.
```JSON
{
    "message": "<password> is updated by <newpassword> successfully"
}
```
* newpassword must satisfy the following conditions:
  * It must be an alphanumeric string with 8 to 32 letters.
  * It must contain at least one letter from all of three categories: uppercase(A-Z), lowercase(a-z)，and numeric(0-9) letters.
* HTTP status codes
  
| status codes      | success/failure | description                                              |
| ----------------- | :-------------: | -------------------------------------------------------- |
| OK(200)           |     success     | password changed successfully                            |
| BAD_REQUEST(400)  |     failure     | malformed parameters or newpassword violating conditions |
| UNAUTHORIZED(401) |     failure     | wrong password                                           |
| NOT_FOUND(404)    |     failure     | no user associated with username                         |

## Deleting user account
* User account deletion can be done by method DELETE with access token explained later as follows: 
```BASH
$ curl -X DELETE -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/account
```
* The Web API returns the following response if successful.
```JSON
{
    "message": "[nakano] deleted"
}
```
* HTTP status codes
  
| status codes      | success/failure | description                           |
| ----------------- | :-------------: | ------------------------------------- |
| OK(200)           |     success     | deletion of user account is completed |
| UNAUTHORIZED(401) |     failure     | wrong access token                    |

# Retrieving access token
* URI: https://qubosolver.cs.hiroshima-u.ac.jp/v1/token 
* Access token can be retrieved from registered username and password.
* Access token is necessary to access directories "problems", "jobs", and "solutions", and to delete user account.

| URI directory | Method | Key-values         | Operation             |
| ------------- | ------ | ------------------ | --------------------- |
| token         | POST   | username，password | retrieve access token |

```BASH
$ curl -X POST https://qubosolver.cs.hiroshima-u.ac.jp/v1/token \
  -d "username=nakano" \
  -d "password=xxxxxxxx"
```
* The Web API returns the following response with access_token if successful.
```JSON
{
  "message": "will expire in 24 hours",
  "access_token": "eyJ0eXAiOiJKV1QiLCJ...omitted...6as2PkikVprUHAYChy2h9T3XY",
}
```
* HTTP status codes
  
| status codes      | success/failure | description                      |
| ----------------- | :-------------: | -------------------------------- |
| OK(200)           |     success     | access token retrieved           |
| BAD_REQUEST(400)  |     failure     | malformed parameters             |
| UNAUTHORIZED(401) |     failure     | wrong password                   |
| NOT_FOUND(404)    |     failure     | no user associated with username |

* It is recommended to set access_token to environment variable JWT for security reason as follows.
```BASH
$ export JWT=eyJ0eXAiOiJKV1QiLCJ...omitted...6as2PkikVprUHAYChy2h9T3XY
```
* HTTP header "Authorization: Bearer $JWT" should be used to access directories that require access token authorization as follows:
```BASH
$ curl -X METHOD -H "Authorization: Bearer $JWT" \
 https://qubosolver.cs.hiroshima-u.ac.jp/v1/{directory} ....
```
# Posting and managing QUBO matrices
* URI: https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems
* Access token authorization is required

| URI directory   | method | description                      |
| --------------- | ------ | -------------------------------- |
| problems        | GET    | Get a list of all problem files  |
| problems        | POST   | POST a QUBO matrix               |
| problems        | DELETE | Delete all QUBO matrices         |
| problems/{file} | GET    | Get information of a QUBO matrix |
| problems/{file} | DELETE | Delete a QUBO matrix             |


## Posting a QUBO matrix
* A QUBO matrix file (.json or .json.gz) can be uploaded to directory "problems" by method POST as follows:
```BASH
$ curl -X POST -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems \
  -F "file=@nug30.json.gz"
```
* The Web API returns the following response if successful.
```JSON
{
   "message": "[nug30.json.gz] uploaded",
   "file": "nug30.json.gz",
   "uri_problem": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems/nug30.json.gz"
}
```
* After a QUBO matrix file is uploaded, the Web API returns a response and **verification** of the QUBO matrix starts. It will take a lot of time if the file is very large.
* The verification process checks if the uploaded file follows the JSON data format correctly, and converts it to a different file format that can be handled by the QUBO solver.
* Jobs for solving the QUBO problem can be submitted after the verification is completed.
* HTTP status codes
  
| status codes                | success/failure | description                                   |
| --------------------------- | :-------------: | --------------------------------------------- |
| ACCEPTED(202)               |     success     | QUBO matrix uploaded and verification started |
| BAD_REQUEST(400)            |     failure     | malformed parameters                          |
| UNAUTHORIZED(401)           |     failure     | wrong access token                            |
| UNSUPPORTED_MEDIA_TYPE(415) |     failure     | file is not JSON data format                  |

## Alternative way for posting a QUBO matrix
* You can upload a QUBO matrix by embedding the JSON of it in the body of a HTTP message.
* The JSON data must have key "file" to specify a file, which must be a json.

### Example
* Suppose that the following JSON data is stored in file "test.json". Key "file" is necessary to specify the name of a file written in directory problems.
```JSON
{
    "file": "k2000.json",
    "nbit": 2000,
    "base": 1,
    "qubo": [
        [1,1,61],
        [1,2,-2],
        [1,3,-2],
        [1,4,-2],
        ...omitted...
        [1999,2000,-2],
        [2000,2000,-49]
    ]
}
```
* By the following cURL command, the JSON data is embedded in the body of the HTTP message.
```BASH
$ curl -X POST -H "Authorization: Bearer $JWT" -H "Content-type: Application/json" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems \
  -d "@test.json"
```
* The JSON data is uploaded and written in file k2000.json (not test.json) of the directory **problem**.

## Getting QUBO matrix information
* The information of a QUBO matrix file can be retrieved by method GET as follows:
```BASH
$ curl -X GET -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems/nug30.json.gz
```
* The Web API returns the following response with information of the specified file if successful.
```JSON
{
    "file": "nug30.json.gz",
    "bytes": 820045,
    "time": "2022-03-21 12:48:50",
    "uri_problem": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems/nug30.json.gz",
    "verify": true,
    "nbit": 900,
    "nelement": 281910,
    "minval": -1000,
    "maxval": 1000,
    "parameters": {
        "problem": "QAP",
        "nbit": 900,
        "base": 0
    }
}
```
* If the verification is in progress, the Web API returns the following simple response.
```JSON
{
    "file": "nug30.json.gz",
    "bytes": 820045,
    "time": "2022-03-13 17:53:25",
    "uri_problem": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems/nug30.json.gz"
}
```
* The Web API returns the following response with error message if the verification is failed.
```JSON
{
    "file": "nug30.json.gz",
    "bytes": 820045,
    "time": "2022-03-21 12:48:50",
    "uri_problem": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems/nug30.json.gz",
    "verify": false,
    "message": "<qubo> has non-integers"
    }
}
```
* HTTP status codes

| status codes      | success | description                          |
| ----------------- | :-----: | ------------------------------------ |
| OK(200)           | success | information of QUBO matrix retrieved |
| UNAUTHORIZED(401) | failure | wrong access token                   |
| NOT_FOUND(404)    | failure | file not found                       |

* The status of the verification can be confirmed by the following table:

| value of "verify" | status of verification   |
| :---------------: | ------------------------ |
|       none        | verification in progress |
|       true        | verification succeeded   |
|       false       | verification failed      |

## Getting a list of problems
* A list of all QUBO matrix files can be obtained by the following cURL command:
```BASH
$ curl -X GET -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems
```
* For example, the Web API returns the following response with file name (file), file size (bytes), file creation time (time), and URI of uploaded files in the directory "problems" (uri_problem).
```JSON
[
    {
        "file": "8queen.json.gz",
        "bytes": 2250,
        "time": "2022-03-13 18:05:11",
        "uri_problem": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems/8queen.json.gz"
    },
    {
        "file": "nug30.json.gz",
        "bytes": 820045,
        "time": "2022-03-13 17:53:25",
        "uri_problem": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems/nug30.json.gz"
    }
]

```

* HTTP status codes
  
| status codes      | success/failure | description                  |
| ----------------- | :-------------: | ---------------------------- |
| OK(200)           |     success     | a list of all files obtained |
| UNAUTHORIZED(401) |     failure     | wrong access token           |

## Deleting a QUBO matrix
* A QUBO matrix can be deleted by method DELETE with QUBO matrix URI as follows:
```BASH
$ curl -X DELETE -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems/nug30.json.gz  
```

* The Web API returns the following response if successful.

```JSON
{
  "message": "[nug30.json.gz] deleted"
}
```
* It may result in incomplete deletion if the file is deleted during the verification process.
* HTTP status codes
  
| status codes      | success/failure | description        |
| ----------------- | :-------------: | ------------------ |
| OK(200)           |     success     | problem is deleted |
| UNAUTHORIZED(401) |     failure     | wrong access token |
| NOT_FOUND(404)    |     failure     | file not found     |


## Deleting all QUBO matrices
* All QUBO matrices are deleted by method DELETE as follows:
```BASH
$ curl -X DELETE -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems
```
* It may result in incomplete deletion if deletion is performed during the verification process.
* HTTP status codes
  
| status codes      | success/failure | description          |
| ----------------- | :-------------: | -------------------- |
| OK(200)           |     success     | all problems deleted |
| UNAUTHORIZED(401) |     failure     | wrong access token   |

# Job registration and management
* URI: https://qubosolver.cs.hiroshima-u.ac.jp/v1/jobs
* Access token authorization is required

| URI directory | method | description                   |
| ------------- | ------ | ----------------------------- |
| jobs          | GET    | get a list of unexecuted jobs |
| jobs          | POST   | post a job                    |
| jobs/{file}   | GET    | get the information of a job  |
| jobs/{file}   | DELETE | delete a job                  |

## Posting a job
* A job can be posted by method POST with parameters specifying a QUBO matrix file as follows:
```BASH
$ curl -X POST -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/jobs \
  -d "problem=nug30.json.gz" \
  -d "time_limit=30"
```
* A QUBO matrix file is specified by key "problem".
* "time_limit" is an example of a key to specify the time limit, which is passed to the QUBO solver.
* The Web API returns the following response if successful.
```JSON
{
    "message": "job for [nug30.json.gz] successfully submitted",
    "job": "nug30_0001.json",
    "uri_problem": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/problems/nug30.json.gz",
    "uri_job": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/jobs/nug30_0001.json",
    "uri_solution": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/solutions/nug30_0001.json"
}
```

* Posting a job is possible for problems that complete verification.
* Job file "nug30_0001.json" is stored in directory "jobs".
* Job files are passed in the order of posting, and the QUBO solver works for jobs in turn.
* When a job is assigned to the QUBO solver, the job file in directory "jobs" is moved to "solutions".
* The job file moved to "solution" is treated as a solution file.
* Intermediate and the final solutions obtained by the QUBO solver are overwritten in the solution file. 
* The QUBO solver updates the solution file when it finds a better solution.
* When the QUBO solver terminates, the solution file stores the best solution obtained by it.
* HTTP status codes
  
| status codes      | success/failure | description                                              |
| ----------------- | :-------------: | -------------------------------------------------------- |
| PROCESSING(102)   |     failure     | posting a job failed because verification is in progress |
| ACCEPTED(202)     |     success     | posting job succeeded.                                   |
| BAD_REQUEST(400)  |     failure     | malformed parameters                                     |
| UNAUTHORIZED(401) |     failure     | wrong access token                                       |
| NOT_FOUND(404)    |     failure     | no QUBO matrix found or verification failed              |

## Job parameters
* The following table includes parameters that can be passed to the QUBO solver.

| Key             | default value | minimum | maximum | description                                                                                                                                                         |
| --------------- | ------------- | ------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| time_limit      | 10            | 1       | 600    | maximum running time of the QUBO solver                                                                                                                             |
| target_energy   | 0             | -       | -       | The QUBO solver terminates if it finds a solution with target energy or less. The value is void if 0.                                                               |
| bfactor         | 2             | 0.001     | 20      | The factor that determines the minimum number of flip operation of each batch search. Each batch search terminates after bfactor*nbit flip operations is performed. |
| factor          | 0.2           | 0.001    | 1       | The factor that determines the minimum number of flip operations of each search. Each search terminates after factor*nbit flip operations is performed.             |
| nsolpool        | 40            | 10      | 1000    | The number of solutions stored in each solution pool.                                                                                                               |
| nisland_per_gpu | 1             | 1       | 5       | The number of island per GPU device                                                                                                                                 |
| arithmetic_bits | 32            | 32      | 64      | The number of bits used in arithmetic operations on GPUs                                                                                                            |

* Each batch search repeats specific search (MaxMin,PositiveMin,CyclicMin,RandomMin, etc) one or more times.
* After a specific search is terminated, the batch search is terminated if bfactor*nbit flip operations are performed so far. Thus, it makes sense to specify values of bfactor and factor satisfying bfactor$>$factor.


## Getting job information
* The information of an unexecuted job can be obtained by method GET for the URI specifying it as follows:
```BASH
$ curl -X GET -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/jobs/nug30_0001.json \
```
* The Web API returns the following response if successful:
```JSON
{
    "job": "nug30_0031.json",
    "problem": "nug30.json.gz",
    "nbit": 900,
    "minval": -1000,
    "maxval": 1000,
    "parameters": {
        "problem": "nug30.json.gz",
        "time_limit": "80"
    }
}
```
* HTTP status codes

| status codes      | success | description                  |
| ----------------- | :-----: | ---------------------------- |
| OK(200)           | success | getting information of a job |
| UNAUTHORIZED(401) | failure | wrong access token           |
| NOT_FOUND(404)    | failure | job file not found           |


## Getting a list of all jobs
* A list of all unexecuted jobs can be retrieved by method GET as follows:
```BASH
$ curl -X GET -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/jobs
```
* The Web API returns the list of unexecuted jobs as follows:
```JSON
[
    {
        "file": "8queen_0001.json",
        "bytes": 189,
        "time": "2022-03-14 16:29:09",
         "uri_job": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/jobs/8queen_0001.json"
    },
    {
        "file": "nug30_0002.json",
        "bytes": 186,
        "time": "2022-03-14 16:47:01",
         "uri_job": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/jobs/nug30_0002.json"
    }
]
```
* If a job is posted when the QUBO solver is waiting for a new job, it is executed immediately. So the posted job will not be shown in the list.
* HTTP status codes

| status codes      | success/failure | description                |
| ----------------- | :-------------: | -------------------------- |
| OK(200)           |     success     | get a list of all problems |
| UNAUTHORIZED(401) |     failure     | wrong access token         |


## Deleting a job
* An unexecuted job can be deleted by method DELETE with URI specifying it.
```BASH
$ curl -X DELETE -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/jobs/nug30_0002.json
```
* The Web API returns the following response if successful.
```JSON
{
    "message": "[nug30_0002.json] deleted"
}
```

| status codes      | success/failure | description        |
| ----------------- | :-------------: | ------------------ |
| OK(200)           |     success     | job file deleted   |
| UNAUTHORIZED(401) |     failure     | wrong access token |
| NOT_FOUND(404)    |     failure     | file not found     |

## Deleting all unexecuted jobs
* All unexecuted jobs can be deleted by method DELETE as follows:
```BASH
$ curl -X DELETE -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/jobs
```
* The Web API returns the following response if successful.
```JSON
{
    "message": "all files in <jobs> deleted"
}
```

* HTTP status codes
  
| status codes      | success/failure | description        |
| ----------------- | :-------------: | ------------------ |
| OK(200)           |     success     | all jobs deleted   |
| UNAUTHORIZED(401) |     failure     | wrong access token |


# Getting and managing solutions
* URI: https://qubosolver.cs.hiroshima-u.ac.jp/v1/solutions

| URI directory        | method | 動作                        |
| -------------------- | ------ | --------------------------- |
| solutions            | GET    | get a list of all solutions |
| solutions            | DELETE | delete all solutions        |
| solutions/{file} | GET    | get a solution              |
| solutions/{file} | DELETE | delete a solution file      |

## Getting a lit of all solutions
* A list of all solution can be retrieved by method GET as follows:
```BASH
$ curl -X GET -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/solutions
```
* For example, the Web API returns the following response if successful.
```JSON
[
    {
        "file": "8queen_0001.json",
        "bytes": 442,
        "time": "2022-10-19 14:54:41",
        "uri_solution": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/solutions/8queen_0001.json" 
    },
    {
        "file": "nug30_0002.json",
        "bytes": 2112,
        "time": "2022-10-19 14:55:03",
        "uri_solution": "https://qubosolver.cs.hiroshima-u.ac.jp/v1/solutions/nug30_0002.json"
    }
]
```
* HTTP status codes

| status codes      | success/failure | description                      |
| ----------------- | :-------------: | -------------------------------- |
| OK(200)           |     success     | a list of all solutions obtained |
| UNAUTHORIZED(401) |     failure     | wrong access token               |

## Getting a solution
* By method GET to the URI corresponding to a solution, the solution vector can be obtained.
```BASH
$ curl -X GET -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/solutions/8queen_0001.json
```
* The Web API returns the following response with the solution vector, the energy, etc.
```JSON
{
    "terminated": true,
    "success": true,
    "problem": "nug30.json.gz",
    "job": "nug30_0002.json",
    "energy": -23872,
    "tts": 57.719,
    "kernel_time": 100.015,
    "solution": [0,0,0,0,0,0,0,0, ...omitted... ,0,1,0,0,0,0,0,0,0,0,0],
    "parameters": {
        "time_limit": 80,
        "target_energy": 0,
        "bfactor": 2.0,
        "factor": 0.2,
        "nsolpool": 40,
        "ngpu": 5,
        "nisland_per_gpu": 1,
        "nisland": 5,
        "value_bits": 16,
        "arithmetic_bits": 32
    }
}

```
* The value of key "terminated" is true if the QUBO solver is terminated.
* The value of key "success" is false if the QUBO solver is abnormally terminated.
* HTTP status codes

| status codes      | success | description                                 |
| ----------------- | :-----: | ------------------------------------------- |
| OK(200)           | success | the solution vector, etc obtained correctly |
| UNAUTHORIZED(401) | failure | wrong access token                          |
| NOT_FOUND(404)    | failure | file not found                              |

## Deleting a solution
* A solution file can be deleted by method DELETE with the URI corresponding to it.  
```BASH
$ curl -X DELETE -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/solutions/nug30_0002.json
```
* The Web API returns the following response if successful.
```JSON
{
    "message": "[nug30_0002.json] deleted"
}

```
* It may results in incomplete deletion if the QUBO solver is working for the corresponding problem.
* HTTP status codes

| status codes      | success/failure | description             |
| ----------------- | :-------------: | ----------------------- |
| OK(200)           |     success     | solution file deleted   |
| UNAUTHORIZED(401) |     failure     | wrong access token      |
| NOT_FOUND(404)    |     failure     | solution file not found |

## Deleting all solutions
* All solutions can be deleted by method DELETE as follows.
```BASH
$ curl -X DELETE -H "Authorization: Bearer $JWT" \
  https://qubosolver.cs.hiroshima-u.ac.jp/v1/solutions
```
* It may result in incomplete deletion if the QUBO solver is working for some problem.
* HTTP status codes
  
| status codes      | success/failure | description                |
| ----------------- | :-------------: | -------------------------- |
| OK(200)           |     success     | all solution files deleted |
| UNAUTHORIZED(401) |     failure     | wrong access token         |