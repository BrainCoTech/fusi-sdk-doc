## Focus Now Server

> Version: 0.60.1
>
> Author: Juewei Dong

> **Warning**: This document may contains inaccurate information, please contact developer if any misleading sections are found 
>
> **Info**: APIs that are marked with ~~strikethrough~~ means that these APIs are deprecated. Deprecated APIs will still be supported, but will no longer be supported in the "near" future. Please be advised to use non-deprecated APIs for better support and simplified experience 

### Overview

- [Account related APIs](#account-related-apis)
  - [Register User ](#register-user)
  - [Verify Registration ](#verify-registration)
  - [Update user information](#update-user-information)
  - ~~[User Login](#user-login)~~
  - [User Login with login name](#user-login-with-login-name)
  - [Get user information](#get-user-information)
  - [delete user account ](#delete-user-account)
  - [Forget password (send verification code) ](#forget-password)
  - [Forget password (verify code and get token) ](#forget-password-verification)
  - [Change password with token](#change-password-with-token)
  - [Change password without token](#change-password-without-token)
  - [Renew Token](#renew-token)
- [Data Storage APIs](#data-storage-apis)
  - [Create Track](#create-track)
- Application version and contents
  - [Application Version Management](#application-version-management)
- [Third Party Login APIs](#third-party-login-api)
  - Register new wechat account
    - [Link email send code](#send-verification-code-when-registering-new-wechat-user-and-bind-email-address)
    - [Link phone number send code]()
    - [Link phone number verify code](#verify-code-when-register-new-wechat-user-and-bind-phone-number)
    - [Link email address verify code](#verify-code-when-register-new-wechat-user-and-bind-email-address)
  - Link Email / Phone
    - [Link email address send code](#link-email-account-by-sending-verification-code-when-user-is-authenticated)
    - [Link phone number send code](#link-phone-number-by-sending-verification-code-when-user-is-authenticated)
    - [Link email address verify code](#link-email-address-verify-code)
    - [Link phone nuber verify code](#link-phone-number-verify-code)
- [Lecture module APIs](#lecture-module-apis)
  - [Get lecture information](#get-lecture-information)



#### Fetch API Domain Name and server status

- Path: `https://statistics.brainco.tech/dns/api_info.json`

- Method: `GET` 

- Response:

  *Example 1*

  ```json
  {
  	"regionCN": {
  		"apiDomainName": "server.edu.brainco.cn",
  		"scheme": "https",
  		"status": "running",
  		"scheduledMaintainance": {
  			"time": null,
  			"maintainanceNotes": null
  		}
  	}
  }
  ```

  | Key                   | Description                                                  | Type   | Nullable |
  | --------------------- | ------------------------------------------------------------ | ------ | -------- |
  | apiDomainName         | Current API server's domain name                             | String | False    |
  | scheme                | `http` or `https`                                            | String | False    |
  | status                | Enum for server status, must be one of`running`, `maintaining`, `down` | String | False    |
  | scheduledMaintainance | Scheduled maintainance info                                  | JSON   | False    |
  | Time                  | Timestamp for scheduled maintainance (nullable)              | Long   | True     |
  | maintainanceNotes     | Maintainance notes.                                          | String | True     |


### Account related APIs

#### Register user 

##### (send / resend verification code)

- Path: `/family/user`

- Method: `POST`

- Request Header:

  | Header name  | Content          |
  | ------------ | ---------------- |
  | Content-type | application/json |

- Request Body:

  *Example 1*

  > Register with email address

  ```json
  {
      "email": "juewei.dong@brainco.tech",
      "appLanguage": "en"
  }
  ```

  *Example 2*

  > Register with phone number

  ```json
  {
      "phone": "13500000000",
      "appLanguage": "zh"
  }
  ```

  *Parameters*

  | Key         | Value                                                        | Optional                       |
  | ----------- | ------------------------------------------------------------ | ------------------------------ |
  | email       | Email address used in registration                           | true, under certain condition* |
  | appLanguage | Language code that follows [IETF BCP 47](https://tools.ietf.org/html/bcp47) , combines [IETF RFC 5646](https://tools.ietf.org/html/rfc5646) and [IETF RFC 4647](https://tools.ietf.org/html/rfc4647) , for example `zh-Hans`  or in `ll-Xxxx` form | false                          |
  | phone       | Phone number without nation code, currently only support Chinese mobile carriers | true, under certain condition* |

  > ***Notice**: One of `email` and `phone`  field must be specified. But both fields cannot be **simutaneously** specified. 

- Response:

  *Example 1:*

  - Response Code: `200 OK`

  - Response Body:

    ```Json
    {
        "id": "245b0fa7-55ec-44f1-8c67-5651aa50b525",
        "username": "Family-User-d39a6b59-72dd-48bc-9720-284125249df4",
        "token": null,
        "active": false
    }
    ```

    *Parameters*

    | Key      | Value                                                        |
    | -------- | ------------------------------------------------------------ |
    | id       | Server postgres primary key                                  |
    | username | Server generated username, begins with `Family-User`         |
    | token    | Authentication token, `null` here                            |
    | active   | User activation status, `false` here (user not activated via email verification) |


  *Example 2:*

  > When user has been activated, attempt to register or send verification code will result in `409 Conflict`  response

  - Response Code: `409 Conflict`

  - Response Body:

    ```Json
    {
        "error": "Email address has already been registered"
    }
    ```

    *Parameters*

    | Key   | Value         |
    | ----- | ------------- |
    | error | Error message |

  

#### Verify Registration
- Path: `/family/user`

- Method: `POST`

- Request Header:

  | Header name  | Content          |
  | ------------ | ---------------- |
  | Content-type | application/json |

- Request Body:

  *Example 1*

  > Verify with email address

  ```json
  { 
  	"username": "zhaoyi",
  	"email": "zhaoyi.yang@brainco.tech",
  	"password": "123456",
  	"code": "337636",
  	"deviceId": "AB:CD:EF:12:34:56",
      "appLanguage": "zh-Hans"
  }
  ```

  *Parameters*

  > Verify with phone number

  ```json
  { 
  	"username": "zhaoyi",
  	"phone": "18804657278",
  	"password": "123456",
  	"code": "337636",
  	"deviceId": "AB:CD:EF:12:34:56",
      "appLanguage": "zh-Hans"
  }
  ```

  

  | Key         | Value                                                        | Optional                       |
  | ----------- | ------------------------------------------------------------ | ------------------------------ |
  | username    | User's username                                              | false                          |
  | code        | Verification code received by Email                          | false                          |
  | email       | Email address used in registration                           | true, under certain condition* |
  | password    | User's password                                              | false                          |
  | deviceId    | Device id, mac address strongly recommended                  | false                          |
  | phone       | Phone number without nation code, currently only support Chinese mobile carriers | true, under certain condition* |
  | appLanguage | Application Language                                         | false                          |

  > ***Notice**: One of `email` and `phone`  field must be specified. But both fields cannot be **simutaneously** specified. 

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "id": "245b0fa7-55ec-44f1-8c67-5651aa50b525",
        "username": "minakoto",
        "token": "eyJhbGciOiJIUzI1NiJ9.eyJqdGki****jbz_KgM",
        "active": true
    }
    ```

    *Parameters*

    | Key      | Value                                                        |
    | -------- | ------------------------------------------------------------ |
    | id       | Server postgres primary key                                  |
    | username | User's user name                                             |
    | token    | Authentication token, Json Web Token here                    |
    | active   | User activation status, `true` here (user already been activated via email verification) |

  *Example 2*

  - Response Code: `401 Unauthorized`

  - Response Body:

    ```Json
    {
        "error": "Verification code does not match"
    }
    ```

    *Parameters*

    | Key   | Value         |
    | ----- | ------------- |
    | Error | Error message |

#### Update user information

- Path: `family/user/info`

- Method: `POST`

- Authentication: **Required** (See headers)

- Request Header:

  | Header name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *token*   |

- Request Body:

  ```json
  {
  	"handedness": 1,
  	"gender": 1,
  	"username": "Umisono",
  	"birthYear": 1994,
      "localCreated": "1527287118750",
      "localUpdated": "1527287118750",
      "attentionHistogram": [1.00, 2.00, 3.00],
      "videoProgress": [
          {
              "videoType": 0,
              "videoIndex": 0,
              "videoProgress": 36,
              "isCompleted": true
          }
      ]
  }
  ```

  *Parameters*

  | Key                | Value                                           | Optional |
  | ------------------ | ----------------------------------------------- | -------- |
  | handedness         | 0 to 10                                         | true     |
  | gender             | 0 or 1                                          | true     |
  | username           | Username                                        | true     |
  | birthYear          | 1900 to 2018                                    | True     |
  | localCreated       | `Long` type Date representation in milliseconds | true     |
  | localUpdated       | `Long` type Date representation in milliseconds | true     |
  | attentionHistogram | Json Array of `Double` type                     | True     |
  | videoProgress      | User's course and tutorial video progress       | True     |

  ***videoProgress*** *Parameters*

  > Optional flags below represents that if the field is optional under the condition that if **videoProgress** presents in the request body

  | Key           | Description                                                  | Type    | Optional |
  | ------------- | ------------------------------------------------------------ | ------- | -------- |
  | videoType     | Video type flag representing the type of the video, `0` is the tutorial video type, `1` is the course video type | Integer | False    |
  | videoIndex    | Video index flag representing the index of the video, it's `0` based | Integer | False    |
  | videoProgress | Time frame in seconds representing when the video has been paused at | Long    | False    |
  | isCompleted   | Flag that represents if the video has been completed by the user | Boolean | False    |

- Response:

  *Example 1*

  - Response Code: `200 OK`

  - Response Body

    ```json
    {
    	"id": "687afbbb-1536-4935-909b-38945cb5fc4a",
        "email": "focus@brainco.tech",
        "username": "bug",
        "handedness": 0,
        "gender": 0,
        "birthYear": 1994,
        "appLanguage": "zh-Hans-CN",
        "localId": null,
        "created": 1530234511716,
        "updated": 1530234511716,
        "localCreated": null,
        "localUpdated": null,
        "avatarUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/user/avatars/738BF4FF64C35C73700B1C2E96F35FF1/F8CDFDA2828FC7857E665B97CBC0B1C5.jpg",
        "mainTagArray": [],
        "reasonArray": [],
        "externalUsers": [
            {
                "name": "David Sonoda",
                "externalUserId": "oVUUo0kFum-tLXcLOgDcJryOjvbU",
                "identityProvider": "WECHAT"
            }
        ],
        "active": true,
        "attentionHistogram": [1.00, 2.00, 3.00],
        "videoProgress": [
            {
                "videoType": 0,
                "videoIndex": 0,
                "videoProgress": 36,
                "isCompleted": true
            }
        ]
    }
    ```


  *Example 2*

  - Response Code: `400 Bad Request`

  - Response Body:

    ```json
    [
        {
            "invalidKey": "birthYear",
            "invalidValue": 2025,
            "errorMessage": "Birth year should range from 1900 to 2018"
        },
        {
            "invalidKey": "gender",
            "invalidValue": 3,
            "errorMessage": "Invalid gender value"
        }
    ]
    ```


#### ~~User Login~~

> See [User Login with login name](#user-login-with-login-name)

##### (with email address)

- Path: `family/auth`

- Method: `POST`

- Request Header:

    | Header name  | Content          |
    | ------------ | ---------------- |
    | Content-type | application/json |

- Request Body

    ```Json
    {
    	"email": "juewei.dong@brainco.tech",
    	"password": "794613852",
    	"deviceId": "AB:CD:EF:12:34:56"
    }
    ```

    *Parameters*

    | Key      | Value                                       | Optional |
    | -------- | ------------------------------------------- | -------- |
    | email    | Email address                               | false    |
    | password | Users password                              | false    |
    | deviceId | Device ID, mac address strongly recommended | False    |

- Response

    *Example 1*

    - Response Code: `200 OK`

    - Response Body:

      ```json
      {
          "id": "245b0fa7-55ec-44f1-8c67-5651aa50b525",
          "loginName": "Family-User-d39a6b59-72dd-48bc-9720-284125249df4",
          "accountType": "FAMILY",
          "token": "eyJhbGciOiJIUzI1NiJ9.eyJ***Q6n93KThc6jbpwULeibaQTfZMrcw"
      }
      ```

      *Parameters*

      | Key         | Value                                                        |
      | ----------- | ------------------------------------------------------------ |
      | id          | Server postgres primary key                                  |
      | loginName   | Server generated loginName, begins with `Family-User` (Not used for login) |
      | accountType | Account Type, `FAMILY` here                                  |
      | **Token**   | Authentication token, Json Web Token here                    |

    *Example 2*

    - Response Code: `401 Unauthorized`

    - Response Body:

      ```json
      {
          "error": "Password is incorrect or account does not exist."
      }
      ```

      *Parameters*

      | Key   | Value         |
      | ----- | ------------- |
      | error | Error Message |

#### User Login with login name
- Path: `family/auth/login` 

- Method: `POST`

- Request Header:

  | Header Name  | Content          |
  | ------------ | ---------------- |
  | Content-type | application/json |

- Request Body:

  ```json
  {
      "loginName": "jueweid@andrew.cmu.edu",
      "password": "12345678",
      "deviceId": "AB:CD:EF:12:34:56"
  }
  ```

- Response:

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

      ```json
      {
          "id": "9136fc9f-22ba-42e9-845e-676cbbcf676d",
          "loginName": "18804657278",
          "accountType": "FAMILY",
          "token": "eyJhbGci****AIMdKdos",
          "accountDetails": {
              "id": "9136fc9f-22ba-42e9-845e-676cbbcf676d",
              "email": null,
              "phone": "18804657278",
              "username": "zhaoyi",
              "handedness": null,
              "gender": null,
              "birthYear": null,
              "appLanguage": "zh-Hans",
              "localId": null,
              "created": 1533165193101,
              "updated": 1533165193101,
              "localCreated": null,
              "localUpdated": null,
              "avatarUri": null,
              "mainTagArray": [],
              "reasonArray": [],
              "registration": false,
              "externalUsers": [],
              "active": true
          },
          "wechatUserInfo": null,
          "videoProgress": [
              {
                  "videoType": 0,
                  "videoIndex": 0,
                  "videoProgress": 36,
                  "isCompleted": true
              }
          ],
      }
      ```
      *Parameters*

      | Key            | Value                                                        |
      | -------------- | ------------------------------------------------------------ |
      | id             | Server generated UUID                                        |
      | loginName      | User's loginName, could be email and phone number            |
      | accountType    | Type of Account                                              |
      | token          | Bearer *Token*                                               |
      | accountDetails | Detailed infomation of account                               |
      | videoProgress  | Video progress Json array representing the video progress data |

  *Example 2*

  - Response Code: `401 Unauthorized`

  - Response Body:

    ```json
    {
        "error": "Password is incorrect or account does not exist."
    }
    ```


#### Get User information

- Path: `family/user/info`

- Method: `GET`

- Authentication: **Required**

- Request Header:

  | Content          | Header name    |
  | ---------------- | -------------- |
  | application/json | Content-type   |
  | Authorization    | Bearer *token* |

- Request Body: Empty

- Response

  *Example 1*

  - Response Code: `200 Ok`

  - Response Body:

    ```json
    {
    	"id": "687afbbb-1536-4935-909b-38945cb5fc4a",
        "email": "focus@brainco.tech",
        "username": "bug",
        "handedness": 0,
        "gender": 0,
        "birthYear": 1994,
        "appLanguage": "zh-Hans-CN",
        "localId": null,
        "created": 1530234511716,
        "updated": 1530234511716,
        "localCreated": null,
        "localUpdated": null,
        "avatarUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/user/avatars/738BF4FF64C35C73700B1C2E96F35FF1/F8CDFDA2828FC7857E665B97CBC0B1C5.jpg",
        "mainTagArray": [],
        "reasonArray": [],
        "externalUsers": [
            {
                "name": "David Sonoda",
                "externalUserId": "oVUUo0kFum-tLXcLOgDcJryOjvbU",
                "identityProvider": "WECHAT"
            }
        ],
        "active": true,
        "attentionHistogram": [1.00, 2.00, 3.00],
        "videoProgress": [
            {
                "videoType": 0,
                "videoIndex": 0,
                "videoProgress": 36,
                "isCompleted": true
            }
        ]
    }
    ```

  *Example 2*

  - Response Code: `401 Unauthorized`
  - Response Body: Empty



#### Delete User Account

- Path: `family/user`

- Method: `DELETE`

- Authentication: **Required**

- Request Header:

  | Content          | Header name    |
  | ---------------- | -------------- |
  | application/json | Content-type   |
  | Authorization    | Bearer *Token* |

- Request Body: Empty

- Response

  *Example 1*

  - Response Code: `204 No Content`

  - Response Body:

    ```json
    {
        "message": "User successfully deleted. (email juewei.dong@brainco.tech)"
    }
    ```

    *Parameters*

    | Key     | Value            |
    | ------- | ---------------- |
    | message | Response message |


#### Forget Password 

##### (send verification code)

- Path: `/family/user/password`

- Method: `PUT`

- Request Header:

  | Content          | Header name  |
  | ---------------- | ------------ |
  | application/json | Content-type |

- Request Body:

  *Example 1*

  ```json
  {
  	"email": "juewei.dong@brainco.tech"
  }
  ```

  *Example 2*

  ```json
  {
  	"loginName": "juewei.dong@brainco.tech"
  }
  ```

  *Example 3*

  ```json
  {
  	"loginName": "13500000000"
  }
  ```

  *Parameters*

  | Key       | Value                                     | Optional                        |
  | --------- | ----------------------------------------- | ------------------------------- |
  | email     | Email address used for reset password     | True, under certain conditions* |
  | loginName | login name, email address or phone number | True, under certain conditions* |

  > ***Notice**: One of `email` and `phone`  field must be specified. But both fields cannot be **simutaneously** specified. 

- Response

  *Example 1*

  - Response Code: `200 Ok`

  - Response Body:

    ```json
    {
        "message": "Code sent"
    }
    ```

    *Parameters*

    | Key     | Value            |
    | ------- | ---------------- |
    | message | Response message |

  *Example 2*

  - Response Code: `429 Too many requests`

  - Response Body:

    ```json
    {
        "message": "Frequency of sending verification request is too high",
        "details": {
            "TTL": 52000
        }
    }
    ```

    *Parameters*

    | Key         | Description                                                  |
    | ----------- | ------------------------------------------------------------ |
    | message     | Response message                                             |
    | details.TTL | Time to live in milliseconds, for example `52000` means another request can be made in `52` seconds |

  *Example 3*

  - Response Code: `400 Bad Request`

  - Response Body:

    ```json
    {
        "error": "Email jueweidong@brainco.tech does not exist"
    }
    ```

    *Parameters*

    | Key   | Value         |
    | ----- | ------------- |
    | error | Error message |

  *Example 3*

  - Response Code: `403 Forbidden`

  - Response Body:

    ```json
    {
        "error": "User has not been activated"
    }
    ```


#### Forget Password Verification

#####  (verify code and get short authentication token)

- Path: `/family/user/password`

- Method: `PUT`

- Request Header:

  | Content          | Header name  |
  | ---------------- | ------------ |
  | application/json | Content-type |

- Request Body: 

  *Example 1*

  ```json
  {
  	"email": "juewei.dong@brainco.tech",
  	"code": "870551",
  	"deviceId": "ABCD"
  }
  ```

  *Example 2*

  ```json
  {
  	"loginName": "13500000000",
  	"code": "870551",
  	"deviceId": "ABCD"
  }
  ```

  

  *Parameters*

  | Key       | Value                               | Optional                        |
  | --------- | ----------------------------------- | ------------------------------- |
  | email     | Users email address                 | true, under certain conditions* |
  | code      | Verification code received in email | false                           |
  | deviceId  | MAC address                         | false                           |
  | loginName | Login name of                       | true, under certain conditions* |

  > ***Notice**: One of `email` and `phone`  field must be specified. But both fields cannot be **simutaneously** specified. 

- Response

  *Example 1*

  - Response Code; `200 OK`

  - Response Body:

    ```json
    {
        "id": 31,
        "username": "Umisono",
        "token": "eyJhbGciOiJIUzI***82rPF7gUPsaeOv8yo1U",
        "active": true
    }
    ```

    | Key      | Value                                                        |
    | -------- | ------------------------------------------------------------ |
    | id       | Server generated primary key                                 |
    | username | User's username                                              |
    | token    | A short term Json Web Token, different from token obtaining from authentication api, this token will expire in **15 minutes**. |
    | active   | User's activation status                                     |

  *Example 2*

  - Response Code: `401 Unauthorized`

  - Response Body: 

    ```Json
    {
        "error": "Verification code does not match"
    }
    ```

    *Parameters*

    | Key   | Value         |
    | ----- | ------------- |
    | error | Error message |



#### Change password with token

##### (when user is shortly authenticated after calling forget password API)

- Path: `/family/user/password`

- Method: `PATCH`

- Authentication: **Required**

- Request Header: 

  | Content          | Header name    |
  | ---------------- | -------------- |
  | application/json | Content-type   |
  | Authorization    | Bearer *Token* |

- Request Body:

  ```json
  {
      "newPassword": "123456"
  }
  ```

  *Parameters*

  | Key         | Value                             |
  | ----------- | --------------------------------- |
  | newPassword | New password the user want to set |

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "message": "Password successfully changed"
    }
    ```

    | Key     | Value            |
    | ------- | ---------------- |
    | message | Response message |

  *Example 2*

  - Response Code: `403 Forbidden`

  - Response Body:

    ```json
    {
        "error": "Token expired"
    }
    ```

    *Parameters*

    | Key   | Value         |
    | ----- | ------------- |
    | error | Error message |


  *Example 3*

  - Response Code:  `401 Unauthorized`
  - Response Body: Empty



#### Change password without token

##### (Authentication not required)

- Path: `/family/user/password`

- Method: `POST`

- Authentication: **Not Required**

- Request Header: 

  | Content          | Header name  |
  | ---------------- | ------------ |
  | application/json | Content-type |

- Request Body:

  *Example 1*

  ```json
  {
      "email": "juewei.dong@brainco.tech",
      "oldPassword": "794613852",
      "newPassword": "123456",
      "deviceId": "12:34:56:AB:CD:EF"
  }
  ```

  *Example 2*

  ```json
  {
      "loginName": "13800000000",
      "oldPassword": "794613852",
      "newPassword": "123456",
      "deviceId": "12:34:56:AB:CD:EF"
  }
  ```

  

  *Parameters*

  | Key         | Value                                              | Optional                        |
  | ----------- | -------------------------------------------------- | ------------------------------- |
  | newPassword | New password the user want to set                  | false                           |
  | email       | User's email address                               | true, under certain conditions* |
  | oldPassword | User's old password                                | false                           |
  | devideId    | MAC address                                        | false                           |
  | loginName   | login name, could be email address or phone number | true, under certain conditions* |

  > ***Notice**: One of `email` and `phone`  field must be specified. But both fields cannot be **simutaneously** specified. 

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "id": 37,
        "loginName": "Family-User-63b0bbdf-9cce-4992-97d3-feb79b0d2e19",
        "accountType": "FAMILY",
        "token": "eyJhbGciOiJIUz****gxM8xVo14fM-IGkic54gPDt_X84rT4"
    }
    ```

    | Key         | Value                                   |
    | ----------- | --------------------------------------- |
    | id          | Postgres generated PK                   |
    | loginName   | Server generated loginName (Not useful) |
    | accountType | Account type, `FAMILY` here             |
    | token       | Newly issued token                      |

  *Example 2*

  - Response Code:  `401 Unauthorized`
  - Response Body: Empty

  *Example 3*

  - Response Code: `400 Bad Request`

  - Response Body:

    ```json
    [
        {
            "invalidKey": "newPassword",
            "invalidValue": "1234",
            "errorMessage": "Login name should contain 5 - 15 characters."
        }
    ]
    ```



#### Renew Token

- Path: `family/auth/token`

- Method: `GET`

- Authentication: **Required**

- Request Header: 

  | Content          | Header name    |
  | ---------------- | -------------- |
  | application/json | Content-type   |
  | Authorization    | Bearer *Token* |

- Request Body: Empty

- Response:

  *Example 1*

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    {
        "token": "eyJhbGciOiJIU***zrW5fM5fOecO-59WA"
    }
    ```

    *Parameters*

    | Key   | Value                                          |
    | ----- | ---------------------------------------------- |
    | token | Refreshed token, the old token will be invalid |

  *Example 2*

  - Response Code: `401 Unauthorized`
  - Response Body: Empty




### Data storage APIs

#### Create Track

- Path: `family/track`

- Method: `POST`

- Authentication: **Required**

- Request Header: 

  | Content          | Header name    |
  | ---------------- | -------------- |
  | application/json | Content-type   |
  | Authorization    | Bearer *Token* |

- Request Body: 

  ```json
  [
  		{
  		"Track": {
  			"localId": "<UUID>",
  			"localCreated": 1528491829351,
  			"localUpdated": 1528491829351,
  			"startTimestamp": 1.11,
  			"endTimestamp": 2.33,	
  			"reasonArray": [{"title": "sleepy", "type": 0, "timestamp": 3.44}, {"title": "pee", "type": 1, "timestamp": 4.55}],
  			"tags": {
  				"mainTag": "aaa",
  				"subTags": ["culture", "literature"]
  			}
  		},
  		"EEG": {
  			"localId": "<UUID>",
  			"localCreated": 1528491829351,
  			"localUpdated": 1528491829351,
  			"attentionData": [1.22, 2.33, 3.44, 4.55],
  			"alphaData": [1.22, 2.33, 3.44, 4.55],
  			"betaData": [1.22, 2.33, 3.44, 4.55],
  			"dataTimestamps": [1.22, 2.33, 3.44, 4.55]
  		}
  		
  	},
  	{
  		"Track": {
  			"localId": "<UUID>",
  			"localCreated": 1528491829351,
  			"localUpdated": 1528491829351,
  			"startTimestamp": 1.11,
  			"endTimestamp": 2.33,
  			"reasonArray": [{"title": "sleepy", "type": 0, "timestamp": 3.44}, {"title": "pee", "type": 1, "timestamp": 4.55}],
  			"tags": {
  				"mainTag": "aaa",
  				"subTags": ["culture", "literature"]
  			}
  		},
  		"EEG": {
  			"localId": "<UUID>",
  			"localCreated": 1528491829351,
  			"localUpdated": 1528491829351,
  			"attentionData": [1.22, 2.33, 3.44, 4.55],
  			"alphaData": [1.22, 2.33, 3.44, 4.55],
  			"betaData": [1.22, 2.33, 3.44, 4.55],
  			"dataTimestamps": [1.22, 2.33, 3.44, 4.55]
  		}
  		
  	}
  ]
  ```

- Response

  *Example 1*:

  - Response Code: `200 OK`

  - Response Body:

    *Sample Response Body (1)*

    ```json
    {
        "Success": [
            {
                "Track": {
                    "localId": "1478",
                    "serverId": "174"
                },
                "EEG": {
                    "localId": "2345",
                    "serverId": "190"
                }
            },
            {
                "Track": {
                    "localId": "9876",
                    "serverId": "177"
                },
                "EEG": {
                    "localId": "8765",
                    "serverId": "193"
                }
            }
        ],
        "Failure": []
    }
    ```

    *Sample Response Body (2)*

    ```json
    {
        "Success": [
            {
                "Track": {
                    "localId": "123456",
                    "serverId": "178"
                },
                "EEG": {
                    "localId": "123456",
                    "serverId": "194"
                }
            },
            {
                "Track": {
                    "localId": "234567",
                    "serverId": "179"
                },
                "EEG": {
                    "localId": "345678",
                    "serverId": "195"
                }
            }
        ],
        "Failure": [
            {
                "Track": {
                    "localId": "456789",
                    "serverId": "null"
                },
                "EEG": {
                    "localId": "987654",
                    "serverId": "null"
                }
            },
            {
                "Track": {
                    "localId": "9876",
                    "serverId": "177"
                },
                "EEG": {
                    "localId": "8765",
                    "serverId": "193"
                }
            }
        ]
    }
    ```

    *Parameters*

    | Key     | Value                                                        |
    | ------- | ------------------------------------------------------------ |
    | Success | Array of`JSON` object of server and client mapping of successfully persisted entities |
    | Failure | Array of`JSON` object of server and client mapping of failed persisted entities |

     

#### Get Track

- Path: `family/track`

- Method: `GET`

- Authorization: **Required**

- Request Header:

  | Content          | Header name    |
  | ---------------- | -------------- |
  | application/json | Content-type   |
  | Authorization    | Bearer *Token* |

- Request Parameter:

  | Name      | Value                                |
  | --------- | ------------------------------------ |
  | timestamp | Long value timestamp in milliseconds |

- Request Example:

  *(bash)*

  ```bash
  curl -X GET \
    'http://server.edu.brainco.cn/family/track?timestamp=1528491829351' \
    -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ***VlKysSUgpag' \
    -H 'Cache-Control: no-cache' \
    -H 'Content-Type: application/json' \
  ```

- Response

  - Response Code: `200 OK`

  - Response Body:

    *Example 1*

    ```json
    [
        {
            "Track": {
                "serverId": "182",
                "localId": "123456",
                "localCreated": 1528491829351,
                "localUpdated": 1528491829351,
                "startTimestamp": 1.11,
                "endTimestamp": 2.33,
                "reasonArray": [
                    {
                        "title": "sleepy",
                        "type": 0,
                        "timestamp": 3.44
                    },
                    {
                        "title": "pee",
                        "type": 1,
                        "timestamp": 4.55
                    }
                ],
                "tags": {
                    "mainTag": "aaa",
                    "subTags": [
                        "literature",
                        "culture"
                    ]
                }
            },
            "EEG": {
                "serverId": "203",
                "localId": "123456",
                "localCreated": 1528491829351,
                "localUpdated": 1528491829351,
                "alphaData": [
                    1.22,
                    2.33,
                    3.44,
                    4.55
                ],
                "betaData": [
                    1.22,
                    2.33,
                    3.44,
                    4.55
                ],
                "attentionData": [
                    1.22,
                    2.33,
                    3.44,
                    4.55
                ],
                "dataTimestamps": [
                    1.22,
                    2.33,
                    3.44,
                    4.55
                ]
            }
        },
        {
            "Track": {
                "serverId": "175",
                ......
                    3.44,
                    4.55
                ]
            }
        }
    ]
    ```

    | Key   | Value      |
    | ----- | ---------- |
    | Track | Track data |
    | EEG   | EEG data   |

#### Delete track

- Path: `family/track`

- Method: `DELETE`

- Authentication: **Required**

- Request Header:

  | Content          | Header name    |
  | ---------------- | -------------- |
  | application/json | Content-type   |
  | Authorization    | Bearer *Token* |

- Request Body:

  ```json
  [
      "174",
      "195",
      "196"
  ]
  ```

  *Explanation*

  Above parameters is a list of **server id** of track records.

- Response:

  - Response Code: `204 No Content`

  - Response Body: 

    ```json
    [
        "174"
    ]
    ```

    *Explanation*

    Above pareameters is a list of **successfully deleted** id of track records.



#### Create challenge 

- Path: `family/challenge`

- Method: `POST`

- Authentication: **Required**

- Request Header:

  | Content          | Header name    |
  | ---------------- | -------------- |
  | application/json | Content-type   |
  | Authorization    | Bearer *Token* |

- Request Body:

  *(JSON Array)*

  ```json
      [
       {
        "Challenge":{
            "localId": "c6",
            "localCreated": 1528491829351,
            "localUpdated": 1528491829351,
            "gameIndex": 0,
            "type": 1,
            "startTimestamp": 456,
            "endTimestamp": 123,
            "scoreData": [99.0, 98.0],
            "dataTimestamps": [1.0, 2.0, 3.0],
            "combineScore": 98.0
        },
        "EEG":{
         "localId": "e6",
            "localCreated": 1528491829351,
            "localUpdated": 1528491829351,
      	   "alphaData": [1.0, 2.0, 3.0, 4.0],
            "betaData": [1.0, 2.0, 3.0, 4.0],
            "attentionData": [1.0, 2.0, 3.0, 4.0],
            "dataTimestamps": [1.0, 2.0, 3.0]
        }
       },
        {
        "Challenge":{
            "localId": "c7",
            ......
        },
            "EEG": {
                ......
            "dataTimestamps": [1.0, 2.0, 3.0]
        }
       }
      ]
  ```
  *Parameters*

  | Key       | Value                                               | Optional |
  | --------- | --------------------------------------------------- | -------- |
  | Challenge | Challenge data to save                              | False    |
  | EEG       | EEG data associated with the Challenge data to save | False    |

- Response

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "Success": [
            {
                "Challenge": {
                    "localId": "c6",
                    "serverId": "59"
                },
                "EEG": {
                    "localId": "e6",
                    "serverId": "198"
                }
            },
            {
                "Challenge": {
                    "localId": "c10",
                    "serverId": "63"
                },
                "EEG": {
                    "localId": "e10",
                    "serverId": "202"
                }
            }
        ],
        "Failure": []
    }
    ```

    *Parameters*

    | Key     | Value                                                        |
    | ------- | ------------------------------------------------------------ |
    | Success | Array of`JSON` object of server and client mapping of successfully persisted entities |
    | Failure | Array of`JSON` object of server and client mapping of failed persisted entities |



#### Delete Track

- Path: `family/challenge`

- Method: `DELETE`

- Authentication: **Required**

- Request Header:

  | Content          | Header name    |
  | ---------------- | -------------- |
  | application/json | Content-type   |
  | Authorization    | Bearer *Token* |

- Request Body:

  ```json
  [
      "174",
      "195",
      "196"
  ]
  ```

  *Explanation*

  Above parameters is a list of **server id** of track records.

- Response:

  - Response Code: `204 No Content`

  - Response Body: 

    ```json
    [
        "174"
    ]
    ```

    *Explanation*

    Above pareameters is a list of **successfully deleted** id of track records.



#### Get challenge

- Path: `family/challenge`

- Method: `GET`

- Authorization: **Required**

- Request Headers:

  | Content          | Header name    |
  | ---------------- | -------------- |
  | application/json | Content-type   |
  | Authorization    | Bearer *Token* |

- Request Parameter:

  | Name      | Value                                |
  | --------- | ------------------------------------ |
  | timestamp | Long value timestamp in milliseconds |

- Request Example:

  *(bash)*

  ```bash
  curl -X GET \
    'http://server.edu.brainco.cn/family/track?timestamp=1528491829351' \
    -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ***VlKysSUgpag' \
    -H 'Cache-Control: no-cache' \
    -H 'Content-Type: application/json' \
  ```

- Response:

  - Response Code: `200 OK`

  - Response Body:

    ```json
    [
        {
            "Challenge": {
                "serverId": "53",
                "localId": "c1",
                "localCreated": 1528491829351,
                "localUpdated": 1528491829351,
                "gameIndex": 0,
                "type": 1,
                "startTimestamp": 456,
                "endTimestamp": 123,
                "scoreData": [
                    99,
                    98
                ],
                "dataTimestamps": [
                    99,
                    98
                ],
                "combineScore": 98
            },
            "EEG": {
                "serverId": "185",
                "localId": "e1",
                "localCreated": 1528491829351,
                "localUpdated": 1528491829351,
                "alphaData": [
                    1,
                    2,
                    3,
                    4
                ],
                "betaData": [
                    1,
                    2,
                    3,
                    4
                ],
                "attentionData": [
                    1,
                    2,
                    3,
                    4
                ],
                "dataTimestamps": [
                    1,
                    2,
                    3
                ]
            }
        },
        ...
    ]
    ```

    | Key       | Value          |
    | --------- | -------------- |
    | Challenge | Challenge data |
    | EEG       | EEG data       |


#### Create Challenge Report

- Path: `family/challengereport`

- Method: `POST`

- Authorization: **Required**

- Request Headers:

  | Content          | Header name    |
  | ---------------- | -------------- |
  | application/json | Content-type   |
  | Authorization    | Bearer *Token* |

- Request Body:

  ```json
  {
  	"challenge": ["c1", "c2", "c3", "c4", "c5"],
  	"suggestion": ["abcd", "abcd", "abcd", "abcd", "efgh"],
      "rankEnabled": true
  }
  ```

  | Key         | Value                                                        | Optional |
  | ----------- | ------------------------------------------------------------ | -------- |
  | challenge   | Challenge local Id                                           | False    |
  | suggestion  | Suggestion local id                                          | False    |
  | rankEnabled | Enable rank for this challenge report or not, see *Leaderboard api*  for more information | True     |

- Response 

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "serverId": "20aa4e99-3363-4eff-aa32-bec4d54351a6"
    }
    ```

    *Parameters*

    | Key      | Value                                   |
    | -------- | --------------------------------------- |
    | serverId | Server generated id of Challenge report |


#### Get Challenge Report

- Path: `family/challengereport`

- Method: `GET`

- Authentication: **Required**

- Request Headers:

  | Content          | Header name    |
  | ---------------- | -------------- |
  | application/json | Content-type   |
  | Authorization    | Bearer *Token* |

- Request Parameter:

  | Name      | Value                                |
  | --------- | ------------------------------------ |
  | timestamp | Long value timestamp in milliseconds |

- Request Example:

  *(bash)*

  ```bash
  curl -X GET \
    'http://server.edu.brainco.cn/family/track?timestamp=1528491829351' \
    -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ***VlKysSUgpag' \
    -H 'Cache-Control: no-cache' \
    -H 'Content-Type: application/json' \
  ```

- Response

  - Response Code: `200 OK`

  - Response Body:

    ```json
    [
        {
            "id": "<UUID>",
            "challengeId": ["<UUID1>", "<UUID2>", "<UUID3>", "<UUID4>", "<UUID5>"],
            "suggestionId": ["<ID6>", "<ID7>", "<ID8>", "<ID9>", "<ID10>"]
        },
        {
            "id": "<UUID>",
            "challengeId": ["<UUID1>", "<UUID2>", "<UUID3>", "<UUID4>", "<UUID5>"],
            "suggestionId": ["<ID6>", "<ID7>", "<ID8>", "<ID9>", "<ID10>"]
        }
    ]
    ```

    *Parameters*

    | Key          | Description                                       |
    | ------------ | ------------------------------------------------- |
    | id           | Server generated id for a challenge report record |
    | challengeId  | Local chanllenge Ids                              |
    | suggestionId | Local suggestion Ids                              |

#### Get All Suggestions

- Path: `family/suggestion`

- Method: `GET`

- Authorization: **Required**

- Request Header:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body: Empty

- Response:

  - Response Code: `200 OK`

  - Response Body:

    ```json
    [
        {
            "serverId": "31",
            "created": 1528500484116,
            "updated": 1528500484116,
            "type": 0,
            "localId": "0",
            "localCreated": 1528500468829,
            "localUpdated": 1528500468829,
            "level": 0,
            "textIndex": 0,
            "textEN": "Your strongest ability is acuity which leads to excellent selective attention. You can easily distinguish between your goals and distractions and remain focused on the task at hand.",
            "textZH": "你的敏锐度非常高，有很优秀的选择决断能力。你能够非常轻松的鉴别目标和干扰，并且能够在执行任务时保持高度的集中。"
        },
        {
            "serverId": "32",
            "created": 1528500484208,
            "updated": 1528500484208,
            "type": 0,
            "localId": "1",
            "localCreated": 1528500468829,
            "localUpdated": 1528500468829,
            "level": 0,
            "textIndex": 1,
            "textEN": "Your strongest ability is acuity which leads to excellent selective attention. Amid the distractions, you can easily make the right choice and stay focused very well. In short, you are very productive.",
            "textZH": "你的敏锐度非常高，有很优秀的选择决断能力。在纷繁的干扰中，你能够轻易的做出正确的选择，并且保持高度的集中，在学习和工作中你常常会很富有成效。"
        },
        {
           ......
        },
            ......
    ]
    ```

#### Create Meditation Data

- Path: `family/meditation`

- Method: `POST`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body:

  ```json
  {
      "startTimestamp": 1500000000000,
      "endTimestamp": 1500000000000,
      "localId": "<UUID>",
      "localCreated": 1500000000000,
      "localUpdated": 1500000000000,
      "EEG": {
          "localId": "<UUID>",
          "localCreated": 1,
          "localUpdated": 1,
          "alphaData": [0.0, 1.0, 2.0],
          "betaData": [0.0, 1.0, 2.0],
          "attentionData": [0.0, 1.0, 2.0],
          "dataTimestamps": [0.0, 1.0, 2.0]
      }
  }
  ```

  *Parameters*

  | Key            | Description                                                  | Type        | Optional |
  | -------------- | ------------------------------------------------------------ | ----------- | -------- |
  | startTimestamp | Start timestamp of meditation, which is epoch time in milliseconds | Long        | False    |
  | endTimestamp   | End timestamp of meditation, which is epoch time in milliseconds | Long        | False    |
  | localId        | Local id for meditation data, should be `UUID`               | String      | False    |
  | localCreated   | Timestamp for local creation time                            | Long        | False    |
  | localUpdated   | Timtestamp for local update time                             | Long        | False    |
  | EEG            | JSON Object that represents                                  | Json Object | False    |

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    {
        "serverId": "20",
        "localId": "<UUID>",
        "EEG": {
            "serverId": "30",
            "localId": "<UUID>"
        }
    }
    ```

    > Explanation:
    >
    > This response body represents a mapping relation between server id and local id of Meditation data and EEG data that affiliates with Meditation data



#### Get meditation data

- Path: `family/meditation`

- Method: `GET`

- Authentication: **Required**

- Request Header:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Parameters

  | Key       | Description                                                  | Type | Optional |
  | --------- | ------------------------------------------------------------ | ---- | -------- |
  | timestamp | Timestamp for fetching users meditation data. if provided, user's meditation data after this timestamp will be fetched. Note the timestamp is epoch time in milliseconds | Long | True     |

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "serverId": "50",
        "localId": "<UUID>",
        "startTimestamp": 1500000000000,
        "endTimestamp": 1500000001000,
        "isActive": true,
        "localUpdated": 1400000000000,
        "localCreated": 1400000000000,
        "EEG": {
            "serverId": "60",
            "localId": "<UUID>",
            "localCreated": 1400000000000,
            "localUpdated": 1400000000000,
            "alphaData": [0.1, 0.2, 0.3],
            "betaData": [0.3, 0.1, 0.2],
            "attentionData": 1400000000000,
            "dataTimestamps": 1400000000000,
            "isActive": true
        }
    )
    ```



#### Create or get neurofeedback data

- Path: `family/neurofeedback`

- Method: `GET` / `POST`

  *Please see the above API **Create or get meditation data** for more details, the format of this 2 APIs and parameters are exactly the same as the mentioned API above*

#### Create / Update Training Progress

- Path: `family/progress`

- Method: `POST`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body:

  ```json
  {
      "level": 0,
      "tag": 0,
      "ecosystemNum": 0,
      "totalCount": 1,
      "gifFrameCount": 100,
      "currentAnimalId": "1",
      "localUpdated": 1500000000000,
      "localCreated": 1500000000000,
      "localId": "<UUID>"
  }
  ```

  | Key             | Description                                              | Type                             | Optional |
  | --------------- | -------------------------------------------------------- | -------------------------------- | -------- |
  | level           | Level of the training progress                           | Integer                          | False    |
  | tag             | Tag of the training progress                             | Integer                          | False    |
  | ecosystemNum    | Ecosystem number of the training progress                | Integer                          | False    |
  | gifFrameCount   | Current gif frame of the training progress               | Integer                          | False    |
  | localUpdated    | Local updated timestamp                                  | Integer                          | False    |
  | localCreated    | Local created timestamp                                  | Integer                          | False    |
  | localId         | Local id                                                 | Integer                          | False    |
  | currentAnimalId | ID of the animal get in this standalone training session | String representation of Integer | True     |

  > Notice:
  >
  > If the API calling is intended to update training progress, local Id should be value return by the server.

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    {
        "level": 0,
        "tag": 0,
        "ecosystemNum": 0,
        "totalCount": 1,
        "gifFrameCount": 100,
        "currentAnimalId": "1",
        "localUpdated": 1500000000000,
        "localCreated": 1500000000000,
        "localId": "<UUID>",
        "animalIds": {
            "1": 8,
            "2": 6
        },
        "isActive": true
    }
    ```

#### Create / Update Training Progress Report

- Path: `family/progress/report`

- Method: `POST`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body:

  ```json
  [
  	{
  		"trainingProgress": {
  			"level": 0,
  			"tag": 0,
  			"ecosystemNum": 2,
  			"gifFrameCount": 0,
  			"totalCount": 0,
              "currentAnimalId": "1",
  			"localUpdated": 1000000000000,
  			"localCreated": 1000000000000,
  			"localId": "<UUID>"
  		},
  		"trainingProgressReport": {
  			"attentionStability": 80.0,
  			"attentionAverage": 20.0,
  			"immersionSpeed": 10.0,
  			"localCreated": 0,
  			"localUpdated": 0,
              "expValue": 10.0,
              "suggestionId": "<Local Suggestion Id>",
  			"localId": "<UUID>",
  			"animalId": "1",
  			"EEG": {
  				"localId": "<UUID>",
  				"localCreated": 1,
  				"localUpdated": 1,
  				"alphaData": [0.0, 1.0, 2.0],
  				"betaData": [0.0, 1.0, 2.0],
  				"attentionData": [0.0, 1.0, 2.0],
  				"dataTimestamps": [0.0, 1.0, 2.0]
  			}
  		},
  		"meditation": {
  			"startTimestamp": 0,
  			"endTimestamp": 0,
  			"localId": "<UUID>",
  			"localCreated": 0,
  			"localUpdated": 0,
  			"EEG": {
  				"localId": "<UUID>",
  				"localCreated": 1,
  				"localUpdated": 1,
  				"alphaData": [0.0, 1.0, 2.0],
  				"betaData": [0.0, 1.0, 2.0],
  				"attentionData": [0.0, 1.0, 2.0],
  				"dataTimestamps": [0.0, 1.0, 2.0]
  			}
  		},
  		"neurofeedback": {
  			"startTimestamp": 0,
  			"endTimestamp": 0,
  			"localId": "<UUID>",
  			"localCreated": 0,
  			"localUpdated": 0,
  			"EEG": {
  				"localId": "<UUID>",
  				"localCreated": 1,
  				"localUpdated": 1,
  				"alphaData": [0.0, 1.0, 2.0],
  				"betaData": [0.0, 1.0, 2.0],
  				"attentionData": [0.0, 1.0, 2.0],
  				"dataTimestamps": [0.0, 1.0, 2.0]
  			}
  		}
  	}, {
  		"trainingProgress": {
  			"level": 0,
  			"tag": 0,
  			"ecosystemNum": 2,
  			"gifFrameCount": 2,
  			"totalCount": 3,
              "currentAnimalId": "3",
  			"localUpdated": 1,
  			"localCreated": 1,
  			"localId": "<UUID>"
  		},
  		"trainingProgressReport": {
  			"attentionStability": 80.0,
  			"attentionAverage": 20.0,
  			"immersionSpeed": 10.0,
  			"localCreated": 0,
  			"localUpdated": 0,
              "expValue": 10.0,
              "suggestionId": "<Local Suggestion Id>",
  			"localId": "<UUID>",
  			"animalId": "1",
  			"EEG": {
  				"localId": "<UUID>",
  				"localCreated": 1,
  				"localUpdated": 1,
  				"alphaData": [0.0, 1.0, 2.0],
  				"betaData": [0.0, 1.0, 2.0],
  				"attentionData": [0.0, 1.0, 2.0],
  				"dataTimestamps": [0.0, 1.0, 2.0]
  			}
  		},
  		"meditation": {
  			"startTimestamp": 0,
  			"endTimestamp": 0,
  			"localId": "<UUID>",
  			"localCreated": 0,
  			"localUpdated": 0,
  			"EEG": {
  				"localId": "<UUID>",
  				"localCreated": 1,
  				"localUpdated": 1,
  				"alphaData": [0.0, 1.0, 2.0],
  				"betaData": [0.0, 1.0, 2.0],
  				"attentionData": [0.0, 1.0, 2.0],
  				"dataTimestamps": [0.0, 1.0, 2.0]
  			}
  		},
  		"neurofeedback": {
  			"startTimestamp": 0,
  			"endTimestamp": 0,
  			"localId": "<UUID>",
  			"localCreated": 0,
  			"localUpdated": 0,
  			"EEG": {
  				"localId": "<UUID>",
  				"localCreated": 1,
  				"localUpdated": 1,
  				"alphaData": [0.0, 1.0, 2.0],
  				"betaData": [0.0, 1.0, 2.0],
  				"attentionData": [0.0, 1.0, 2.0],
  				"dataTimestamps": [0.0, 1.0, 2.0]
  			}
  		}
  	}
  ]
  ```

  *Parameter Explanation*

  ***TrainingProgress*** 

  | Key              | Description                                                  | Optional |
  | ---------------- | ------------------------------------------------------------ | -------- |
  | trainingProgress | Training progress entity represents a position on a specific land where treee is grown. | False    |

  ***TrainingProgress*** fields

  | Key             | Description                                                  | Type    | Optional |
  | --------------- | ------------------------------------------------------------ | ------- | -------- |
  | level           | Level of training module                                     | Integer | False    |
  | tag             | Position on a land in training module                        | Integer | Fasle    |
  | ecosystemNum    | Ordinal number of ecosystem in.training module               | Integer | False    |
  | gifFrameCount   | Current gif frame number of tree growth animation of position `(ecosystemNum, level, tag)` | Integer | False    |
  | totalCount      | Total count of gif frame of tree growth animation of position `ecosystemNum, level, tag)` | Integer | False    |
  | localCreated    | Client side created timestamp                                | Long    | False    |
  | localUpdated    | Client side created timestamp                                | Long    | False    |
  | localId         | Training progress local id                                   | Long    | False    |
  | currentAnimalId | Current Training Progress Animal Id for Animal replacement   | String  | True     |

  ***TrainingProgressReport*** 

  | Optional               | Description                                                  | Optional |
  | ---------------------- | ------------------------------------------------------------ | -------- |
  | trainingProgressReport | Training progress report entity representing a training progress report. | False    |

  ***TrainingProgressReport*** fields

  | Key                | Description                                                  | Type    | Optional |
  | ------------------ | ------------------------------------------------------------ | ------- | -------- |
  | attentionStability | Level of training module                                     | Double  | False    |
  | attentionAverage   | Position on a land in training module                        | Double  | Fasle    |
  | immersionSpeed     | Ordinal number of ecosystem in.training module               | Double  | False    |
  | gifFrameCount      | Current gif frame number of tree growth animation of position `(ecosystemNum, level, tag)` | Integer | False    |
  | localUpdated       | Total count of gif frame of tree growth animation of position `ecosystemNum, level, tag)` | Long    | False    |
  | localCreated       | Client side created timestamp                                | Long    | False    |
  | localId            | Local id of training progress report, must be `UUID`         | String  | False    |
  | animalId           | Animal server id, which represents an animal the user obtained in this training session | String  | False    |
  | eeg                | EEG object, see below                                        | String  | False    |
  | expValue           | Experience value associated with certain training progress report, default value is 0.0 | Double  | True     |
  | suggestionId       | Local suggestion id for training progress report suggestion, contact our lead *[iOS Developer](mailto:hsiao.kai-chung@brainco.tech)* for more details. | String  | False    |

  ***EEG***

  | Key            | Description                                                  | Type           | Optional |
  | -------------- | ------------------------------------------------------------ | -------------- | -------- |
  | alphaData      | Alpha data of EEG, should be an array of double              | `List<Double>` | False    |
  | betaData       | Beta data of EEG, should be an array of double               | `List<Double`  | False    |
  | dataTimestamps | Array of timestamps, each element of which is an x-coordinate of a EEG data point | `List<Double>` | False    |
  | attentionData  | Array of attention data, each element of which represnets the attention value at corresponding timestamp | `List<Double>` | False    |
  | localUpdated   | Total count of gif frame of tree growth animation of position `ecosystemNum, level, tag)` | Long           | False    |
  | localCreated   | Client side created timestamp                                | Long           | False    |
  | localId        | Local id of training progress report, must be `UUID`         | String         | False    |

  ***Meditation***

  | Key            | Description                   | Type | Optional |
  | -------------- | ----------------------------- | ---- | -------- |
  | startTimestamp | Start timestamp of meditation | Long | False    |
  | endMeditation  | End timestamp of meditation   | Long | False    |

  ***Neurofeedback***

  *Same as Meditation format above*


- Response:

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    [
        {
            "serverId": "79dc2bed-95ed-496f-96e8-e63e445893df",
            "localId": "CD6AE4E8",
            "trainingProgress": {
                "level": 0,
                "tag": 2,
                "ecosystemNum": 0,
                "gifFrameCount": 2,
                "totalCount": 1,
                "animalIds": {
                    "25": 3
                },
                "serverId": "72",
                "localId": "985364F8",
                "localCreated": 1537823353438,
                "localUpdated": 1537823364601,
                "currentAnimalId": "25",
                "active": true
            },
            "meditation": {
                "serverId": "28",
                "localId": "067A2F5E",
                "startTimestamp": 1537823354131,
                "endTimestamp": 1537823355448,
                "localCreated": 1537823353438,
                "localUpdated": 1537823356719,
                "EEG": {
                    "serverId": "446",
                    "localId": "9F60A515",
                    "localCreated": 1537823353438,
                    "localUpdated": 1537823356726,
                    "alphaData": [
                        0
                    ],
                    "betaData": [
                        0
                    ],
                    "attentionData": [
                        3
                    ],
                    "dataTimestamps": [
                        1537823355028
                    ],
                    "isActive": true
                },
                "active": true
            },
            "neurofeedback": {
                "serverId": "18",
                "localId": "AA1C97AA",
                "startTimestamp": 1537823356500,
                "endTimestamp": 1537823358511,
                "localUpdated": 1537823360309,
                "localCreated": 1537823353438,
                "EEG": {
                    "serverId": "447",
                    "localId": "599BA7D3",
                    "localCreated": 1537823353438,
                    "localUpdated": 1537823360315,
                    "alphaData": [
                        0,
                        0
                    ],
                    "betaData": [
                        0,
                        0
                    ],
                    "attentionData": [
                        65,
                        0
                    ],
                    "dataTimestamps": [
                        1537823357028,
                        1537823358027
                    ],
                    "isActive": true
                },
                "active": true
            },
            "EEG": {
                "serverId": "458",
                "localId": "3A567CB3",
                "localCreated": 1537823353438,
                "localUpdated": 1537823353438,
                "alphaData": [
                    0,
                    0,
                    0
                ],
                "betaData": [
                    0,
                    0,
                    0
                ],
                "attentionData": [
                    78,
                    35,
                    64
                ],
                "dataTimestamps": [
                    1537823361027,
                    1537823362028,
                    1537823363027
                ],
                "isActive": true
            },
            "distributions": {
                "attentionStability": {
                    "0.0": 0,
                    "10.0": 0,
                    "20.0": 0,
                    "30.0": 0,
                    "40.0": 0,
                    "50.0": 0.3157894736842105,
                    "60.0": 0.3157894736842105,
                    "70.0": 0.3157894736842105,
                    "80.0": 0.9298245614035088,
                    "90.0": 1,
                    "100.0": 1
                },
                "attentionAverage": {
                    "0.0": 0,
                    "10.0": 0,
                    "20.0": 0.6140350877192983,
                    "30.0": 0.6140350877192983,
                    "40.0": 0.6140350877192983,
                    "50.0": 0.6140350877192983,
                    "60.0": 0.9298245614035088,
                    "70.0": 0.9298245614035088,
                    "80.0": 1,
                    "90.0": 1,
                    "100.0": 1
                },
                "immersionSpeed": {
                    "0.0": 0,
                    "2.0": 0,
                    "4.0": 0,
                    "6.0": 0,
                    "8.0": 0,
                    "10.0": 0.6140350877192983,
                    "12.0": 0.6140350877192983,
                    "14.0": 0.6140350877192983,
                    "16.0": 0.6842105263157895,
                    "18.0": 0.6842105263157895,
                    "20.0": 0.6842105263157895,
                    "22.0": 0.6842105263157895,
                    "24.0": 0.6842105263157895,
                    "26.0": 0.6842105263157895,
                    "28.0": 0.6842105263157895,
                    "30.0": 0.6842105263157895
                }
            },
            "ranks": {
                "attentionStability": 0.2982456140350877,
                "attentionAverage": 0.9122807017543859,
                "immersionSpeed": 0.9824561403508771
            },
            "suggestionId": "101",
            "expValue": 207,
            "animalId": "25",
            "attentionStability": 49,
            "attentionAverage": 59,
            "immersionSpeed": 99,
            "isActive": true
        },
        {
            "serverId": "9596a9c9-2c92-4149-b2f9-80bec9b1c6d0",
            "localId": "CD6AE4E8-E7A3-4DE8-BEFB-6E6FE704B87B",
            "trainingProgress": {
                "level": 0,
                "tag": 2,
                "ecosystemNum": 0,
                "gifFrameCount": 2,
                "totalCount": 1,
                "animalIds": {
                    "25": 3
                },
                "serverId": "72",
                "localId": "985364F8",
                "localCreated": 1537823353438,
                "localUpdated": 1537823364601,
                "currentAnimalId": "25",
                "active": true
            },
            "meditation": {
                "serverId": "27",
                "localId": "067A2F5E-0D90-4AB1-BC99-E498C3FDEDA1",
                "startTimestamp": 1537823354131,
                "endTimestamp": 1537823355448,
                "localCreated": 1537823353438,
                "localUpdated": 1537823356719,
                "EEG": {
                    "serverId": "443",
                    "localId": "9F60A515-BC1A-43DB-860D-7AEF7F9983CF",
                    "localCreated": 1537823353438,
                    "localUpdated": 1537823356726,
                    "alphaData": [
                        0
                    ],
                    "betaData": [
                        0
                    ],
                    "attentionData": [
                        3
                    ],
                    "dataTimestamps": [
                        1537823355028
                    ],
                    "isActive": true
                },
                "active": true
            },
            "neurofeedback": {
                "serverId": "17",
                "localId": "AA1C97AA-3BC8-4155-8DD9-0BE609A8CE8A",
                "startTimestamp": 1537823356500,
                "endTimestamp": 1537823358511,
                "localUpdated": 1537823360309,
                "localCreated": 1537823353438,
                "EEG": {
                    "serverId": "444",
                    "localId": "599BA7D3-3CA7-4EDF-8C19-A36665DCA89F",
                    "localCreated": 1537823353438,
                    "localUpdated": 1537823360315,
                    "alphaData": [
                        0,
                        0
                    ],
                    "betaData": [
                        0,
                        0
                    ],
                    "attentionData": [
                        65,
                        0
                    ],
                    "dataTimestamps": [
                        1537823357028,
                        1537823358027
                    ],
                    "isActive": true
                },
                "active": true
            },
            "EEG": {
                "serverId": "445",
                "localId": "3A567CB3-CBAF-4B03-879B-6F5089F5BB25",
                "localCreated": 1537823353438,
                "localUpdated": 1537823353438,
                "alphaData": [
                    0,
                    0,
                    0
                ],
                "betaData": [
                    0,
                    0,
                    0
                ],
                "attentionData": [
                    78,
                    35,
                    64
                ],
                "dataTimestamps": [
                    1537823361027,
                    1537823362028,
                    1537823363027
                ],
                "isActive": true
            },
            "distributions": {
                "attentionStability": {
                    "0.0": 0,
                    "10.0": 0,
                    "20.0": 0,
                    "30.0": 0,
                    "40.0": 0,
                    "50.0": 0.09302325581395349,
                    "60.0": 0.09302325581395349,
                    "70.0": 0.09302325581395349,
                    "80.0": 0.9069767441860465,
                    "90.0": 1,
                    "100.0": 1
                },
                "attentionAverage": {
                    "0.0": 0,
                    "10.0": 0,
                    "20.0": 0.813953488372093,
                    "30.0": 0.813953488372093,
                    "40.0": 0.813953488372093,
                    "50.0": 0.813953488372093,
                    "60.0": 0.9069767441860465,
                    "70.0": 0.9069767441860465,
                    "80.0": 1,
                    "90.0": 1,
                    "100.0": 1
                },
                "immersionSpeed": {
                    "0.0": 0,
                    "2.0": 0,
                    "4.0": 0,
                    "6.0": 0,
                    "8.0": 0,
                    "10.0": 0.813953488372093,
                    "12.0": 0.813953488372093,
                    "14.0": 0.813953488372093,
                    "16.0": 0.9069767441860465,
                    "18.0": 0.9069767441860465,
                    "20.0": 0.9069767441860465,
                    "22.0": 0.9069767441860465,
                    "24.0": 0.9069767441860465,
                    "26.0": 0.9069767441860465,
                    "28.0": 0.9069767441860465,
                    "30.0": 0.9069767441860465
                }
            },
            "ranks": {
                "attentionStability": 0.06976744186046512,
                "attentionAverage": 0.8837209302325582,
                "immersionSpeed": 0.9767441860465116
            },
            "suggestionId": "101",
            "expValue": 207,
            "animalId": "25",
            "attentionStability": 49,
            "attentionAverage": 59,
            "immersionSpeed": 99,
            "isActive": true
        }
    ]
    ```

    ***Parameters Explanation*** 

    The response is a JSON array containing multiple JSON Objects.

    | Key                              | Description                                                  |
    | -------------------------------- | ------------------------------------------------------------ |
    | distributions.attentionStability | Attention stability distribution data. Note this distribution is a discrete [*cumulative distribution function*](https://zh.wikipedia.org/wiki/累积分布函数). The client side has to parse the CDF fo [*probability density function*](https://zh.wikipedia.org/wiki/機率密度函數). |
    | distributions.attentionAverage   | Attention average value CDF.                                 |
    | distributions.immersionSpeed     | Immersion speed CDF                                          |
    | ranks.attentionStability         | The rank in percentage of attention stability, the higher the better. |
    | ranks.attentionAverage           | The rank in percetage of attention average, the higher the better. |
    | ranks.immersionSpeed             | The rank in percentage of immersion speed, the lower the better. |
    | localId                          | Local id of training progress.                               |
    | localCreated                     | Local created timestamp                                      |
    | localUpdated                     | Local updated timestamp                                      |
    | serverId                         | Server side id of training progress report.                  |

  *Example 2*

  - Response Code: `409 Conflict`

  - Response Body:

    ```json
    {
     	"error": "Training progress report for localId <UUID> has existed"   
    }
    ```

  *Example 3*

  - Response Code: `409 Conflict`

  - Response Body: 

    ```json
    {
        "error": "Meditation data already linked to existed training progress report for localId <UUID>"
    }
    ```

#### Get training progress report

- Path: `family/progress/report`

- Method: `GET`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Parameter:

  | Key       | Description                                                  | Type | Optional |
  | --------- | ------------------------------------------------------------ | ---- | -------- |
  | timestamp | Timestamp for fetching users historical training progress report data, if provided, user's training progress after this timestamp will be fetched. Note the timestamp is epoch time in milliseconds | Long | False    |

- Request Example

  ```bash
  curl -X GET \
    https://server.edu.brainco.cn/family/progress/report \
    -H 'Authorization: Bearer eyJhbGciOiJIU****tTZnt0' \
    -H 'Cache-Control: no-cache' \
    -H 'Content-Type: application/json' \
  ```

- Response

  *Example 1*

  - Response Code: `200 Ok`
  - Response Body: *See the response body for create training progress report*

#### Get all training progress

- Path: `family/progress`

- Method: `GET`

- Authentication Required: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body: empty

- Response

  *Example 1*

  - Response Code: `200 OK`
  - Response Body:

  ```json
  [
      {
          "level": 0,
          "tag": 0,
          "ecosystemNum": 0,
          "gifFrameCount": 0,
          "totalCount": 0,
          "animalIds": {
              "1": 4,
              "2": 7
          },
          "localUpdated": 0,
          "localCreated": 0,
          "serverId": 0,
          "localId": "<UUID>",
          "currentAnimalId": "2",
          "isActive": true
      }, {
          "level": 1,
          "tag": 0,
          "ecosystemNum": 0,
          "gifFrameCount": 0,
          "totalCount": 0,
          "animalIds": {
              "1": 4,
              "2": 7
          },
          "localUpdated": 0,
          "localCreated": 0,
          "serverId": 0,
          "localId": "<UUID>",
          "currentAnimalId": "2",
          "isActive": true
      }
  ]
  ```

  *Parameters*

  | Key          | Description                       |
  | ------------ | --------------------------------- |
  | level        | Level of training progress        |
  | tag          | Position on the training progress |
  | ecosystemNum | Ecosystem ordinal number          |
  | totalCount   | Total count of gif frames.        |
  | animaiIds    | Animal frequency map, key is      |

#### Get number of users reached certain ecosystem and level in training progress

- Path: `family/progress/stats`

- Method: `GET`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Parameters: 

  | Key           | Description                       | Type | Optional |
  | ------------- | --------------------------------- | ---- | -------- |
  | level         | Level of training progress        | Int  | False    |
  | tag           | Position on the training progress | Int  | False    |
  | ecosystem_num | Ecosystem ordinal number          | Int  | True     |

- Response:

  - Response Code: `200 No Content`

  - Response Body:

    ```json
    {
        "ecosystemNum": 0,
        "level": 0,
        "numberReached": 20
    }
    ```

    *Parameters*

    | Key           | Description                                  | Type    |
    | ------------- | -------------------------------------------- | ------- |
    | ecosystemNum  | Ecosystem number in Focus Planet             | Integer |
    | level         | Level number in Focus Planet training module | Integer |
    | numberReached | Number of users reached certain criteria     | Integer |


#### Get attention data distributions

- Path: `family/progress/distributions`

- Method: `GET`

- Authentication: **Not Required**

- Request Headers:

  | Header Name  | Content          |
  | ------------ | ---------------- |
  | Content-type | application/json |

- Request Parameters: empty

- Response:

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    {
        "attentionStabilityDist": {
            "0.0": 0,
            "10.0": 0,
            "20.0": 0,
            "30.0": 0,
            "40.0": 0,
            "50.0": 0,
            "60.0": 0,
            "70.0": 0,
            "80.0": 0,
            "90.0": 1,
            "100.0": 1
        },
        "attentionAverageDist": {
            "0.0": 0,
            "10.0": 0,
            "20.0": 0,
            "30.0": 0,
            "40.0": 0,
            "50.0": 0,
            "60.0": 0,
            "70.0": 0,
            "80.0": 1,
            "90.0": 1,
            "100.0": 1
        },
        "immersionSpeedDist": {
            "0.0": 0,
            "2.0": 0,
            "4.0": 0,
            "6.0": 0,
            "8.0": 0,
            "10.0": 0,
            "12.0": 0,
            "14.0": 0,
            "16.0": 1,
            "18.0": 1,
            "20.0": 1,
            "22.0": 1,
            "24.0": 1,
            "26.0": 1,
            "28.0": 1,
            "30.0": 1
        }
    }
    ```

    | Key                    | Description                                                  |
    | ---------------------- | ------------------------------------------------------------ |
    | attentionStabilityDist | Distribution data of attention distribution in discrete CDF. |
    | attentionAverageDist   | Distribution data of attention average in discrete CDF.      |
    | immersionSpeedDist     | Distribution data of immersion speed in discrete CDF.        |

#### Upload badge

- Path: `family/badge`

- Method: `POST`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body:

  ```json
  {
  	"transactionId": "<UUID>",
  	"badgeId": "2"
  }
  ```

  *Parameters*

  | Key           | Description                                    | Optional |
  | ------------- | ---------------------------------------------- | -------- |
  | transactionId | The unique identifier of a single transaction. | False    |
  | badgeId       | The server Id of the badge                     | False    |

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    {
        "transactionId": "<UUID>",
        "created": 1536868767667,
    }
    ```

  *Example 2*

  - Response Code: `409 Conflict`

  - Response Body:

    ```json
    {
        "error": UNREPEATABLE
    }
    ```

    > **Explanation**
    >
    > Some of the badges can only be unlocked once and can not be obtained multiple times. If the user try to upload or increment a badge of this type, the server would respond with `409`

  *Example 3*

  - Response Code: `400 Bad Request`

  - Response Body:

    ```json
    {
        "error": "Invalid BadgeId 0"
    }
    ```

    > **Explanation**
    >
    > Badge id range on the server is currently 1 - 16, any out-of-range badge id would result in this response

#### Get badge freqency map of user

- Path: `family/badge`

- Method: `GET`

- Authentication: **Required**

- Request Headers: 

  | Key           | Description                                    | Optional |
  | ------------- | ---------------------------------------------- | -------- |
  | transactionId | The unique identifier of a single transaction. | False    |
  | badgeId       | The server Id of the badge                     | False    |

- Request Parameter: empty

- Response Body:

  ```json
  {
      "1": 1,
      "2": 5
  }
  ```

  > **Explanation**
  >
  > Above is a frequency map of badge frequency. For example `"2": 5` represents that  the user has obtained 5 badges whose `id` is `2`



#### Upload experience value transaction

- Path: `family/transaction/exp`

- Method: `POST`

- Authentication:  **Required**

- Request Headers:

  | Key           | Description                                    | Optional |
  | ------------- | ---------------------------------------------- | -------- |
  | transactionId | The unique identifier of a single transaction. | False    |
  | badgeId       | The server Id of the badge                     | False    |

- Request Body:

  ```json
  {
      "transactionId": "<UUID>",
      "operationType": "FOCUS_PLANET",
      "expValue": 10.0
  }
  ```

  *Parameter*

  | Key           | Description                                                  | Type        | Optional |
  | ------------- | ------------------------------------------------------------ | ----------- | -------- |
  | transactionId | A transaction ID.generated locally to uniquely identify a transaction | UUID        | False    |
  | operationType | The operation type of this transaction, could be `FOCUS_PLANET`, `FOCUS_TRACK`, `FOCUS_ELEVATE`, `FOCUS_SHARING`, `FOCUS_LOGIN` | Enum String | False    |
  | expValue      | The experience value the user get in this transaction        | Double      | False    |

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "id": "<UUID>",
        "transactionId": "<UUID>",
        "expValue": 10.0,
        "operationType": "FOCUS_TRACK",
        "created": 1500000000000,
        "updated": 1500000000000
    }
    ```

    *Parameters*

    | Key           | Description                                         | Type        |
    | ------------- | --------------------------------------------------- | ----------- |
    | id            | Server id of experience                             | UUID string |
    | transactionId | Transaction id that uniquely identify a transaction | UUID String |
    | operationType | Operation type mentioned above                      | Enum String |
    | created       | Transaction server creation type                    | Long        |
    | updated       | Transaction server update time                      | Long        |

  *Example 2*

  - Response Code: `429 Too Many Requests`

  - Response Body:

    ```json
    {
        "error": "Login bonus has already been granted today, try tomorrow."
    }
    ```

    > **Explation**
    >
    > If the operation type is `FOCUS_LOGIN`, this type of transaction can only be uploaded once per day according to the App Definition



#### Get experience value transactions

- Path: `family/transaction/exp`

- Method: `GET`

- Authentication:  **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Parameters:

  | Key       | Description                                                  | Type | Optional |
  | --------- | ------------------------------------------------------------ | ---- | -------- |
  | timestamp | The start timestamp of client for fetching experience value transaction history, should be milliseconds form of epoch time | Long | True     |

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    [
        {
            "id": "<UUID>",
            "transactionId": "<UUID>",
            "expValue": 10.0,
            "operationType": "FOCUS_TRACK",
            "created": 1500000000000,
            "updated": 1500000000000
        },
        {
            "id": "<UUID>",
            "transactionId": "<UUID>",
            "expValue": 10.0,
            "operationType": "FOCUS_TRACK",
            "created": 1500000000000,
            "updated": 1500000000000
        }
    ]
    ```


### Application Version management

> ***Warning***:
>
>  Following APIs are for development only and should not be used in any released and distributed version of client application

#### Add or update application version (Focus EDU Training system / Focus Now)

- Path: `family/appversion`

- Method: `POST`

- Authentication: **Required**

- **Roles Allowed**; **`SYSTEM_ADMIN`**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

> ***Info***: 
>
> `SYSTEM_ADMIN` role is granted to certain administrative account. In above `SYSTEM_ADMIN` only allowed case, you should firsly authenticate the server with your administrator account. Then use the token the server responded to access the apis and resources.
>
> Apart from admin access only resources and apis. Admin account can also access other apis and resources as other family / edu training system users
>
> For administrative account details, contact developer (Juewei Dong) for futher details and credentials.

> ***Warning***
>
> Please be advised that only to create new version record of application when you are ready to release your newer version of application. 

- Request Body:

  ```json
  {
      "version": "1.1.2",
      "clientType": "FOCUS_NOW",
      "releaseDate": 1530630616417,
      "releaseNotes": "Sample release notes"
  }
  ```

  *Parameters*

  | Key          | Description                                             | Optional |
  | ------------ | ------------------------------------------------------- | -------- |
  | version      | Version string representation of application            | False    |
  | clientType   | Client type, `FOCUS_EDU_TRAINING` or `FOCUS_NOW` and `` | False    |
  | releaseDate  | Release date `Long` representation in milliseconds      | False    |
  | releaseNotes | Release Notes of this application version               | True     |

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "id": "df7d36ac-f6cd-43c7-a172-81b0ac5d886a",
        "clientType": "FOCUS_EDU_TRAINING",
        "version": "0.1.2",
        "releaseDate": 1530630616417,
        "releaseNotes": "Sample release notes"
    }
    ```

#### Get App Version

- Path: `family/appversion`

- Method: `GET`

- Request Headers:

  | Header Name  | Content          |
  | ------------ | ---------------- |
  | Content-type | application/json |

- Request Parameter:

  | Name        | Value                                                        |
  | ----------- | ------------------------------------------------------------ |
  | client_type | `FOCUS_EDU_TRAINING` or `FOCUS_NOW` or `FOCUS_NOW_ANDROID`   |
  | latest      | Boolean value of whether getting the latest app version information, `true` of `false` |

- Response

  *Example 1*

  - Request Sample:

    > Getting ***all*** version information of `FOCUS_EDU_TRAINING` system

    ```bash
    curl -X GET \
      'https://server.edu.brainco.cn/family/appversion?client_type=FOCUS_EDU_TRAINING' \
      -H 'Cache-Control: no-cache' \
      -H 'Content-Type: application/json' 
    ```

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    [
        {
            "id": "93782e02-8816-4b4e-9718-077708d4302f",
            "clientType": "FOCUS_EDU_TRAINING",
            "version": "0.1.3",
            "releaseDate": 1530582176581,
            "releaseNotes": null
        },
        {
            "id": "df7d36ac-f6cd-43c7-a172-81b0ac5d886a",
            "clientType": "FOCUS_EDU_TRAINING",
            "version": "0.1.2",
            "releaseDate": 1530630616417,
            "releaseNotes": null
        },
        {
            "id": "67d74e1b-6ad7-4d82-8ba8-38884617388f",
            "clientType": "FOCUS_EDU_TRAINING",
            "version": "0.2.1",
            "releaseDate": 1533319391726,
            "releaseNotes": "sample release notes"
        }
    ]
    ```

  *Example 2*

  - Request Sample: 

  - > Getting the ***lateset*** application version of `FOCUS_EDU_TRAINING` 

    ```bash
    curl -X GET \
      'https://server.edu.brainco.cn/family/appversion?client_type=FOCUS_EDU_TRAINING&latest=true' \
      -H 'Cache-Control: no-cache' \
      -H 'Content-Type: application/json'
    ```

    - Response Code: `200 Ok`
    - Response Body: 

      ```json
      {
          "id": "67d74e1b-6ad7-4d82-8ba8-38884617388f",
          "clientType": "FOCUS_EDU_TRAINING",
          "version": "0.2.1",
          "releaseDate": 1533319391726,
          "releaseNotes": "sample release notes"
      }
      ```

  *Example 3*

  - Response Code: `400 Bad Request`

  - Response Body:

    ```json
    {
        "error": "No enum constant brainco.focus.family.server.entity.ClientType.FOCUS_EDU_TRAININ"
    }
    ```




### Course Video Management

#### Get course video url list from server

- Path: `family/course/info`

- Method: `GET`

- Authentication: **NOT Required**

- Request Header: 

  | Header Name  | Content          |
  | :----------- | ---------------- |
  | Content-type | application/json |

- Request Body: Empty

- Request Parameter: 

  | Key         | Description                                                  | Optional |
  | ----------- | ------------------------------------------------------------ | -------- |
  | version     | Version string for *Focus EDU Traning System* and *Focus Now* | False    |
  | client_type | Client type identifier, `FOCUS_EDU_TRAINING`, `FOCUS_EDU_TEACHING`, `FOCUS_NOW` | False    |

  *Example*

  ```bash
  curl -X GET https://server.edu.brainco.cn/family/course/info?version=0.2.0&client_type=FOCUS_EDU_TRAINING
  ```

- Request Body:

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "[course1,part1,video1]": {
            "id": "e7e7b3e5-fc12-4607-85e6-49b4dbd1cc12",
            "courseName": "why_we_need_focus1.1",
            "fileName": "82fe49e1-bd9d-4931-9116-e4fed8150401",
            "fileType": "mp4",
            "url": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToB/course/82fe49e1-bd9d-4931-9116-e4fed8150401.mp4",
            "clientType": "FOCUS_EDU_TRAINING",
            "fileStatus": "COMPLETED",
            "created": 1531203294731,
            "updated": 1531203420525
        },
        "[course1,part1,video2]": {
            "id": "6fe06ccc-fb66-41fb-acdf-170a76e76204",
            "courseName": "why_we_need_focus1.2",
            "fileName": "b63b4b5d-87fb-4009-bfbc-e24c4ad2135c",
            "fileType": "mp4",
            "url": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToB/course/b63b4b5d-87fb-4009-bfbc-e24c4ad2135c.mp4",
            "clientType": "FOCUS_EDU_TRAINING",
            "fileStatus": "COMPLETED",
            "created": 1531203472868,
            "updated": 1531203581406T
        },
        "[course1,part1,video3]": {
            "id": "d7551c99-cfe9-4fa9-b923-a32ea1bb5c78",
            "courseName": "why_we_need_focus1.3",
            "fileName": "1f5095b7-6cda-411e-8e5c-999ed02a9fc5",
            "fileType": "mp4",
            "url": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToB/course/1f5095b7-6cda-411e-8e5c-999ed02a9fc5.mp4",
            "clientType": "FOCUS_EDU_TRAINING",
            "fileStatus": "COMPLETED",
            "created": 1531203601566,
            "updated": 1531203679831
        },
        "[course1,part2,video1]": {
            "id": "04f90d48-967e-4836-8c19-26b01b2a34c4",
            "courseName": "what_is_meditation1.2.1",
            "fileName": "40a4a75d-2d8f-445a-ad37-aad48665b803",
            "fileType": "mp4",
            "url": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToB/course/40a4a75d-2d8f-445a-ad37-aad48665b803.mp4",
            "clientType": "FOCUS_EDU_TRAINING",
            "fileStatus": "COMPLETED",
            "created": 1531203707288,
            "updated": 1531203741646
        }
    }
    ```

    > Notice: 
    >
    > Response here is a map, the key of the map is string representation of course index array. 
    >
    > For example, `"[course1, part2, video1]"` represents that this course information is for course1, part2, video1

    | Key        | Description                                                  |
    | ---------- | ------------------------------------------------------------ |
    | id         | Server generated course record id                            |
    | courseName | Course video name                                            |
    | fileName   | Course file name on cloud                                    |
    | fileType   | Course video type                                            |
    | uri        | Course video url for downloading or streaming                |
    | clientType | Client type of video, for example `FOCUS_EDU_TRAINING`, `FOCUS_EDU_TEACHING`, `FOCUS_NOW` |
    | fileStatus | FIle status of the video, must check the file status to be `COMPLETED` before downloading using the url |

    > Notice:
    >
    > Contact server admin for software version info update.

### Avatar Service

#### Request Permission for uploading new avatar

- Path: `family/user/avatar`

- Method: `POST`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body: Empty

- Response:

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "accessKeyId": "<AccessKeyId>",
        "accessKeySecret": "<AccessKeySecret>",
        "securityToken": "<SecurityToken>",
        "bucketName": "focus-resources",
        "resourcesUri": "universal/user/avatars/<md5_value_of_user_id>/<md5_value_of_datetime>.jpg",
        "fileName": "<md5_value_of_datetime>",
        "ticket": "ey****.*****"
    }
    ```

    *Parameter*

    | Key             | Description                                                  |
    | --------------- | ------------------------------------------------------------ |
    | accessKeyId     | Aliyun OSS Security Token Service generated credential id, valid for `900` seconds |
    | accessKeySecret | Aliyun OSS Security Token Service generated credential key, valid for `900` seconds |
    | securityToken   | Aliyun OSS Security Token Service generated security token, valid for `900` seconds |
    | bucketName      | Allowed upload bucket name of aliyun OSS                     |
    | resourceUri     | Resource URI based on designated aliyun OSS bucket           |
    | fileName        | Server generated file name for avatar picture file           |
    | ticket          | Server generated ticket, similar to ticket in Wechat Authentication Service, but carrying different information |

#### Avatar uploading callback

- Path: `family/user/avatar/callback`

- Method: `POST`

- Authentication: **Required**

- Ticket Authentication: **Required**

- Request Headers: 

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body:

  ```json
  {
      "ticket": "eyJ****.*****"
  }
  ```

  *Parameter*

  | Key    | Description                                                  | Optional |
  | ------ | ------------------------------------------------------------ | -------- |
  | ticket | Server generated ticket carrying uri info of newly uploaded avatar | False    |

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    {
        "avatarUri": "https://focus-resources.oss-cn-beijing.aliyuncs.com/universal/avatar/<md5_val>/<another_md5_val>.jpg"
    }
    ```

    | Key       | Description                                 |
    | --------- | ------------------------------------------- |
    | avatarUri | Avatar Uri for newly uploaded user's avatar |

  *Example 1*

  - Response Code: `401 OK`

  - Response Body: 

    ```json
    {
        "error": "This ticket is not issued under this user's authorization, aborting"
    }
    ```






### Challenge leaderboard service

> ***Notice***:
>
> Currently, the leaderboard will be reset every Thursday day at 00:00:00 (Timezone: GMT +8, Beijing, Hongkong, Taipei)

#### ~~Create rank data (standalone)~~

- Path: `family/ranking`

- Method: `POST`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body:

  ```json
  {
      "score": 89.9
  }
  ```

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    {
        "message": "Succesfully updated ranking info"
    }
    ```


#### Create ranking data (when creating challenge report)

See create challenge report api.

> ***Notice:*** 
>
> If you wish to create ranking data for newly posted challenge report, you must specify the `rankEnabled` to be `ture` 

 #### Get global ranking data

- Path: `family/ranking/global`

- Method: `GET`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body: Empty

- Request Parameter:

  | Key   | Value                                               | Optional |
  | ----- | --------------------------------------------------- | -------- |
  | start | Start value for the leaderboard, for example `5000` | False    |
  | end   | End value for the leaderboard, for example `5050`   | False    |

  *Example 1*

  ```
  https://server.edu.brainco.cn/family/ranking/global?start=0&end=2
  ```

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    [
        {
            "rank": 0,
            "score": 98.98,
            "name": "test2552",
            "loginName": "test2552@brainco.tech",
            "date": 1531168372831
        },
        {
            "rank": 1,
            "score": 98.98,
            "name": "test9928",
            "loginName": "test9928@brainco.tech",
            "date": 1531168381983
        },
        {
            "rank": 2,
            "score": 98.98,
            "name": "test22353",
            "loginName": "test22353@brainco.tech",
            "date": 1531168397135
        }
    ]
    ```

  *Example 2*

  - Response Code: `400 Bad Request`

  - Response Body:

    ```json
    {
        "error": "Start and end rank should be positive or 0"
    }
    ```

#### Get ranking by score

- Path: `family/ranking`

- Method: `GET`

- Authentication: **Required**

- Request Headers: 

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Parameter: 

  | Key   | Description                                                  | Optional |
  | ----- | ------------------------------------------------------------ | -------- |
  | score | Score (double) that you want to get the ranking in current leaderboard | False    |

  *Example 1*

  ```Bash
  https://server.edu.brainco.cn/family/ranking?score=92.0
  ```

  For example, the ranking data on the server is,

  ```json
  [
      {
          "rank": 0,
          "score": 98,
          "name": "BrianCo Focus",
          "avatarUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/user/avatars/661627AB1739A85D77F18DFCD2F3470E/4BCD3D455E32E3A85ACEB584CD8F5D72.jpg",
          "userId": "6aebbcc2-d256-48bb-b918-3fc066407409",
          "date": 1531506170495
      },
      {
          "rank": 1,
          "score": 90,
          "name": "juewei",
          "avatarUri": null,
          "userId": "820ac0c2-846c-4638-b994-d67c4cb5fdd8",
          "date": 1531843234495
      }
  ]
  ```

  so, the ranking would be 

  | Ranking | Name          | Score |
  | ------- | ------------- | ----- |
  | 0       | BrainCo Focus | 98    |
  | 1       | juewei        | 90    |

- Response

  *Example 1* `score=90`

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    {
        "ranking": 2
    }
    ```

    > Notice: The ranking data is zero based

  *Example 2* `score=91`

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "ranking": 1
    }
    ```

  *Example 3* `score=99`

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "ranking": 0
    }
    ```



#### Get leaderboard refreshed timestamp

- Path: `family/ranking/refresh-timestamp`

- Method: `GET`

- Request Headers: empty

- Response:

  *Example 1*

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    {
        "refreshTimestamp": 150000000000
    }
    ```

#### Get login user ranking data

- Path: `family/ranking/self`

- Method: `GET`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body: Empty

- Request Parameter: Empty

  *Example 1*

  ```
  https://server.edu.brainco.cn/family/ranking/self
  ```

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "rank": 0,
        "score": 98.98,
        "name": "test2552",
        "loginName": "test2552@brainco.tech",
        "date": 1531168372831
    }
    ```

    *Parameters*

    | Key       | Description                                                  |
    | --------- | ------------------------------------------------------------ |
    | rank      | Rank of the user                                             |
    | score     | Highest challenge average score of current user in current challenge leaderboard |
    | name      | Name of the user                                             |
    | loginName | LoginName (email) of the current user                        |
    | date      | Date of  the challenge ranking data being created (in milliseconds starting 1970) |

  *Example 2*

  - Response Code: `404 Not Found`

  - Response Body:

    ```json
    {
        "error": "Ranking data not found for current user"
    }
    ```


### Third party login API

> ***Warning:***
>
> Third party developer accounts including Wechat developer account, weibo developer account, facebook developer account etc provide API keys for developers to access oauth0/2 apis. Please only use api access keys and secret for testing purpose, **Do not ** write it in your configuration files or hard code it in your code. All access keys should be kept on the server. 

#### Wechat login / Create wechat login account

> Description: 

- Path: `family/oauth2/wechat/login`

- Method: `POST`

- Authentication: Not required

- Request Headers:

  | Header Name  | Content          |
  | ------------ | ---------------- |
  | Content-type | application/json |

- Request Body:

  ```json
  {
      "code": "<Wechat verification code>",
      "lang": "zh-Hans",
      "deviceId": "AB:CD:EF:12:34:56",
      "createNewAccount": false
  }
  ```

  *Parameters*

  | Key              | Description                                                  | Optional |
  | ---------------- | ------------------------------------------------------------ | -------- |
  | code             | Wechat oauth api code parameter, see [*WeChat Mobile Application Login Development Guide*](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419317851&token=&lang=en_US). `code` **is not needed** when `createNewAccount` is true. | False    |
  | lang             | Language code that follows [IETF BCP 47](https://tools.ietf.org/html/bcp47) , combines [IETF RFC 5646](https://tools.ietf.org/html/rfc5646) and [IETF RFC 4647](https://tools.ietf.org/html/rfc4647) , for example `zh-Hans` in `ll-Xxxx` form | False    |
  | deviceId         | MAC address                                                  | False    |
  | createNewAccount | Boolean value, if specified and the value is `true`, server will create new account directly if fetched Wechat `OpenID` has not linked to any specific account or registered as an account in the server. If not specified, default value will be `false` and server will not automatically create user if `OpenID` does not exist. | True     |
  | ticket           | Server generated ticket, if createNewAccount specified, ticket should be specified either | True     |

- Response:

  *Example 1*

  - Response Code: `404 Not found`

  - Response Message:

    ```json
    {
        "message": "Wechat User not found, specify 'createNewAccount' parameter to create new account or use ticket to send email",
        "ticket": "eyJub2RlM****EFgE=",
        "expire": 1527287118750
    }
    ```

    *Parameters*

    | Key     | Description                                                  |
    | ------- | ------------------------------------------------------------ |
    | Message | Error message for wechat user not found                      |
    | ticket  | A ticket for accessing a list of access restricted api without bearer authorization. Specifically, now *Register wechat user with linkage to email address - send email verification code* API need ticket authorization. Ticket contains information fetched from third party apps. See *Info* part of *Send verification code when register new wechat user & bind email address* API |
    | expire  | Expiration time for ticket in milliseconds starting from 1970 |

  *Example 2*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
    	"id": "245b0fa7-55ec-44f1-8c67-5651aa50b525",
        "loginName": "Wechat-User-d39a6b59-72dd-48bc-9720-284125249df4",
        "accountType": "FAMILY",
        "token": "eyJhbGciOiJIUzI1NiJ9.eyJqdGki****jbz_KgM",
    	"registration": true,    
        "accountDetails": {
            "handedness": 0,
            "gender": 1,
            "birthYear": 1992,
            "email": null,
            "name": "DavidSonoda",
            "avatarUri": "http://...."
        },
        "userInfo": { 
            "access_token":"ACCESS_TOKEN", 
            "expires_in":7200, 
            "refresh_token":"REFRESH_TOKEN",
            "openid":"OPENID", 
            "scope":"SCOPE",
            "unionid":"o6_bmasdasdsad6_2sgVt7hMZOPfL"
        }
    }
    ```

    *Parameter*

    | Key            | Description                                                  |
    | -------------- | ------------------------------------------------------------ |
    | id             | Server generated primary key for account                     |
    | loginName      | Temporary login name when user has not set any loginName     |
    | accountType    | Account Type, `FAMILY` here                                  |
    | token          | Json web token                                               |
    | registration   | Boolean flag that specifies if the API action is registration or login. |
    | accountDetails | Account details of newly created wechat user                 |
    | userInfo       | User information json from WeChat, see [*WeChat Mobile Application Login Development Guide*](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419317851&token=&lang=en_US) |

  *Example 3*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
    	"id": "245b0fa7-55ec-44f1-8c67-5651aa50b525",
        "loginName": "Juewei",
        "accountType": "FAMILY",
        "token": "eyJhbGciOiJIUzI1NiJ9.eyJqdGki****jbz_KgM",
    	"registration": false,    
        "accountDetails": {
            "handedness": 0,
            "gender": 1,
            "birthYear": 1992,
            "email": null,
            "name": "DavidSonoda",
            "avatarUri": "http://...."
        },
        "userInfo": { 
            "access_token":"ACCESS_TOKEN", 
            "expires_in":7200, 
            "refresh_token":"REFRESH_TOKEN",
            "openid":"OPENID", 
            "scope":"SCOPE",
            "unionid":"o6_bmasdasdsad6_2sgVt7hMZOPfL"
        }
    }
    ```

    *Parameters are same as example 2*

    > Info: Notice that `registration` flag here is false, meaning that this api call is login, not register user.



#### Send verification code when registering new wechat user and bind email address

- Path: `family/oauth2/registration/email`

- Method: `POST`

- Authentication: **Not required**

- Ticket authentication: **Required**

  > Info:
  >
  > Server issued "ticket" is a server generated token which carries information of the user fetched from third party account, it has 2 parts, header and signature,
  >
  > | Part      | Description                                                  |
  > | --------- | ------------------------------------------------------------ |
  > | Header    | The header is BASE64 encoded user's information fetched from third party application by oauth2 apis. It's not **Encrypted**. |
  > | Signature | Signature is AES **Encrypted** of the header part. Used for self validation to ensure the header is unmodified since generated. |
  >
  > Designing of ticket is inspired by **Json Web Token**, but the *Claim* part is more flexible, can carries self defined json inside the ticket, other than standardized *Claim*.
  >
  > ***Warning***:
  >
  > Ticket is as important as user's authentication token. Don't disclose this credential to unauthorized users or parties.

- Request Header:

  | Header Name  | Content          |
  | ------------ | ---------------- |
  | Content-type | application/json |

- Request Body:

  ```json
  {
      "email": "jueweid@andrew.cmu.edu",
      "ticket": "eyJ*******=.sw+A***=",
      "lang": "zh-Hans",
  }
  ```

  *Parameters*

  | Key    | Description                                                  | Optional |
  | ------ | ------------------------------------------------------------ | -------- |
  | email  | Email address to bind                                        | False    |
  | ticket | Server generated ticket, see info above part of this API     | False    |
  | lang   | Language code that follows [IETF BCP 47](https://tools.ietf.org/html/bcp47) , combines [IETF RFC 5646](https://tools.ietf.org/html/rfc5646) and [IETF RFC 4647](https://tools.ietf.org/html/rfc4647) , for example `zh-Hans` in `ll-Xxxx` form | False    |

- Response: 

  *Example 1*

  - Response Code: `200 OK` 

  - Response Body:

    ```json
    {
        "message": "Verification code has been sent to email address jueweid@andrew.cmu.edu"
    }
    ```

  *Example 2*

  - Response Code: `401 Unauthorized`

  - Response Body:

    ```json
    {
        "error": "Ticket corrupted or not valid"
    }
    ```

    > Explanation: Signature check failed for this ticket

    ```json
    {
        "error": "Ticket expired"
    }
    ```

    > Explanation: Ticket expired

#### Send verification code when registering new wechat user and bind email address

- Path: `family/oauth2/registration/phone`

- Method: `POST`

- Authentication: **Not required**

- Ticket authentication: **Required**

- Request Headers:

  | Header Name  | Content          |
  | ------------ | ---------------- |
  | Content-type | application/json |

- Request Body:

  ```json
  {
      "phone": "13500000000",
      "ticket": "eyJ*******=.sw+A***=",
      "lang": "zh-Hans",
  }
  ```

- *Parameters*

  | Key    | Description                                                  | Optional |
  | ------ | ------------------------------------------------------------ | -------- |
  | phone  | Phone number to bind                                         | False    |
  | ticket | Server generated ticket, see info above part of this API     | False    |
  | lang   | Language code that follows [IETF BCP 47](https://tools.ietf.org/html/bcp47) , combines [IETF RFC 5646](https://tools.ietf.org/html/rfc5646) and [IETF RFC 4647](https://tools.ietf.org/html/rfc4647) , for example `zh-Hans` in `ll-Xxxx` form | False    |

- Response: 

  *Example 1*

  - Response Code: `200 OK` 

  - Response Body:

    ```json
    {
        "message": "Verification code has been sent to phone number 13500000000"
    }
    ```

  *Example 2*

  - Response Code: `401 Unauthorized`

  - Response Body:

    ```json
    {
        "error": "Ticket corrupted or not valid"
    }
    ```

    > Explanation: Signature check failed for this ticket

    ```json
    {
        "error": "Ticket expired"
    }
    ```

    > Explanation: Ticket expired


#### Verify code when register new wechat user and bind phone number

- Path: `family/oauth2/registration/phone/verify`

- Method: `POST`

- Authentication: **Not Required**

- Ticket authentication: **Required**

- Request Headers:

  | Header Name  | Content          |
  | ------------ | ---------------- |
  | Content-type | application/json |

- Request Body:

  ```json
  {
      "phone": "13500000000",
      "ticket": "eyJ******=.l*****==",
      "lang": "zh-Hans",
      "code": "485318",
      "deviceId": "AB:CD:EF:12:34:56",
      "password": "abc12345678"
  }
  ```

  *Parameters*

  | Key      | Description                                                  | Optional |
  | -------- | ------------------------------------------------------------ | -------- |
  | phone    | Phone number to be linked to newly registered account        | False    |
  | ticket   | Server issued ticket, issued at `family/oauth2/wechat/login` | False    |
  | lang     | IETF BCP 47 language code                                    | False    |
  | code     | Verification code that user received by Phone                | False    |
  | deviceId | MAC address                                                  | False    |
  | password | Password of the user                                         | False    |

- Response

  *Example 1*

  - Response Code: `401 Unauthorized`

  - Response Body: 

    ```json
    {
    	"error": "Verification code does not match"   
    }
    ```

  *Example 2*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "message": "Phone number 13500000000 has been succesfully linked to current account"
    }
    ```



#### Verify code when register new wechat user and bind email address

- Path: `family/oauth2/registration/email/verify`

- Method: `POST`

- Authentication: **Not Required**

- Ticket authentication: **Required**

- Request Headers:

  | Header Name  | Content          |
  | ------------ | ---------------- |
  | Content-type | application/json |

- Request Body:

  ```json
  {
      "email": "jueweid@andrew.cmu.edu",
      "ticket": "eyJ******=.l*****==",
      "lang": "zh-Hans",
      "code": "485318",
      "deviceId": "AB:CD:EF:12:34:56",
      "password": "abc12345678"
  }
  ```

  *Parameters*

  | Key      | Description                                                  | Optional |
  | -------- | ------------------------------------------------------------ | -------- |
  | email    | Email address to be linked to newly registered account       | False    |
  | ticket   | Server issued ticket, issued at `family/oauth2/wechat/login` | False    |
  | lang     | IETF BCP 47 language code                                    | False    |
  | code     | Verification code that user received by Email                | False    |
  | deviceId | MAC address                                                  | False    |
  | password | Password of the user                                         | False    |

- Response

  *Example 1*

  - Response Code: `401 Unauthorized`

  - Response Body: 

    ```json
    {
    	"error": "Verification code does not match"   
    }
    ```

  *Example 2*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "id": "00eb4fac-a56e-4dc2-9d6f-24c5fff32c17",
        "loginName": "15917058263@163.com",
        "accountType": "FAMILY",
        "token": "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiJiMWY1N2U0Ny1iZmQ2LTQzY2UtOGZiZC00MmQxZWNhOWZmMDkiLCJpc3MiOiJodHRwOi8vZXhhbXBsZS5vcmciLCJhdWQiOiJodHRwOi8vZXhhbXBsZS5vcmciLCJzdWIiOiJXZWNoYXQtVXNlci1iZmU3NTljMS1iYjRlLTQzOTgtOGVjMC1lOWI1OTE5NDlkM2IiLCJpYXQiOjE1MzM5MTg1MTksImV4cCI6MTUzNjUxMDUxOSwiYXV0aG9yaXRpZXMiOlsiRkFNSUxZIl0sInJlZnJlc2hDb3VudCI6MCwicmVmcmVzaExpbWl0IjoyMDAwfQ.CMUc2pYfze0n701Ck23aUfxwkB8YR54dYBtwL7FCYe0",
        "accountDetails": {
            "id": "00eb4fac-a56e-4dc2-9d6f-24c5fff32c17",
            "email": "15917058263@163.com",
            "phone": null,
            "username": "line",
            "handedness": null,
            "gender": 0,
            "birthYear": null,
            "appLanguage": "zh-Hans",
            "localId": null,
            "created": 1533918519284,
            "updated": 1533918519284,
            "localCreated": null,
            "localUpdated": null,
            "avatarUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/user/avatars/58DE4BAB4AE1DD56FC55E05D1C09225B/7214ECDDA0D5830987E06B920CAD006C.jpg",
            "mainTagArray": null,
            "reasonArray": null,
            "registration": true,
            "externalUsers": [
                {
                    "name": "line",
                    "externalUserId": "oVUUo0ras4bImO_71WgnBQbkaRRM",
                    "identityProvider": "WECHAT"
                }
            ],
            "active": true
        },
        "wechatUserInfo": {
            "openid": "oVUUo0ras4bImO_71WgnBQbkaRRM",
            "nickname": "line",
            "sex": 1,
            "language": "zh_CN",
            "city": "Shenzhen",
            "province": "Guangdong",
            "country": "China",
            "headimgurl": "http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKrEGU8yaic9mKJtdwRY3viaGAex1mv2OJ39QuVaDLhRQrWMMzJzqvlqrDUAUFdh7pVavc61F2TRFQg/132",
            "privilege": [],
            "unionid": "oJzdD0aK8gbvuKDk1hv1T1m2_dog"
        }
    }
    ```



#### Link wechat account

- Path: `family/oauth2/wechat/link`

- Method: `POST`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body:

  ````json
  {
      "code": "<Wechat code>",
  	"lang": "zh-Hans",
      "deviceId": "AB:CD:EF:12:34:56"
  }
  ````

- Response

  *Example 1*

  - Response Code: `409 Conflict`

  - Respoonse Body:

    ````json
    {
     	"error": "Wechat account has already existed, binding failed"   
    }
    ````

    > Explanation: This account has linked to wechat account before so linking operation failed

  *Example 2*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "id": "00eb4fac-a56e-4dc2-9d6f-24c5fff32c17",
        "loginName": "15917058263@163.com",
        "accountType": "FAMILY",
        "token": "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiJiMWY1N2U0Ny1iZmQ2LTQzY2UtOGZiZC00MmQxZWNhOWZmMDkiLCJpc3MiOiJodHRwOi8vZXhhbXBsZS5vcmciLCJhdWQiOiJodHRwOi8vZXhhbXBsZS5vcmciLCJzdWIiOiJXZWNoYXQtVXNlci1iZmU3NTljMS1iYjRlLTQzOTgtOGVjMC1lOWI1OTE5NDlkM2IiLCJpYXQiOjE1MzM5MTg1MTksImV4cCI6MTUzNjUxMDUxOSwiYXV0aG9yaXRpZXMiOlsiRkFNSUxZIl0sInJlZnJlc2hDb3VudCI6MCwicmVmcmVzaExpbWl0IjoyMDAwfQ.CMUc2pYfze0n701Ck23aUfxwkB8YR54dYBtwL7FCYe0",
        "accountDetails": {
            "id": "00eb4fac-a56e-4dc2-9d6f-24c5fff32c17",
            "email": "15917058263@163.com",
            "phone": null,
            "username": "line",
            "handedness": null,
            "gender": 0,
            "birthYear": null,
            "appLanguage": "zh-Hans",
            "localId": null,
            "created": 1533918519284,
            "updated": 1533918519284,
            "localCreated": null,
            "localUpdated": null,
            "avatarUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/universal/user/avatars/58DE4BAB4AE1DD56FC55E05D1C09225B/7214ECDDA0D5830987E06B920CAD006C.jpg",
            "mainTagArray": null,
            "reasonArray": null,
            "registration": true,
            "externalUsers": [
                {
                    "name": "line",
                    "externalUserId": "oVUUo0ras4bImO_71WgnBQbkaRRM",
                    "identityProvider": "WECHAT"
                }
            ],
            "active": true
        },
        "wechatUserInfo": {
            "openid": "oVUUo0ras4bImO_71WgnBQbkaRRM",
            "nickname": "line",
            "sex": 1,
            "language": "zh_CN",
            "city": "Shenzhen",
            "province": "Guangdong",
            "country": "China",
            "headimgurl": "http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKrEGU8yaic9mKJtdwRY3viaGAex1mv2OJ39QuVaDLhRQrWMMzJzqvlqrDUAUFdh7pVavc61F2TRFQg/132",
            "privilege": [],
            "unionid": "oJzdD0aK8gbvuKDk1hv1T1m2_dog"
        }
    }
    ```

    *Parameters*

    | Key            | Description                                                  |
    | -------------- | ------------------------------------------------------------ |
    | id             | Server generated primary key for account                     |
    | loginName      | Temporary login name when user has not set any loginName     |
    | accountType    | Account Type, `FAMILY` here                                  |
    | token          | Json web token                                               |
    | registration   | Boolean flag that specifies if the API action is registration or login. |
    | accountDetails | Account details of newly created wechat user                 |
    | userInfo       | User information json from WeChat, see [*WeChat Mobile Application Login Development Guide*](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419317851&token=&lang=en_US) |

    > Explanation: 
    >
    > When linking operation successful, server will respond with newly issued JWT token with user info fetched from WeChat open API
    >
    > Note that the `registration` flag is always false here, response format is same as wechat login API



#### Link Email address by sending verification code when user is authenticated

- Path: `family/oauth2/email`

- Method: `POST`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body:

  ```json
  {
      "email": "minakoto00@gmail.com"
  }
  ```

  *Paramters*

  | Key   | Description                         | Optional |
  | ----- | ----------------------------------- | -------- |
  | email | Email address the user want to link | False    |

- Response

  *Example 1*

  - Response Code: `403 Forbidden`

  - Response Body:

    ```json
    {
        "error": "User has already linked email address"
    }
    ```

  *Example 2*

  - Response Code: `403 Forbidden`

  - Response Body:

    ```json
    {
        "error": "Email address has already been linked to other account"
    }
    ```

  *Example 3*

  - Response Code: `200`

  - Response Body: 

    ```json
    {
        "message": "Verification code has been sent to email address minakoto00@gmail.com"
    }
    ```

#### Link Phone number by sending verification code when user is authenticated

- Path: `family/oauth2/phone`

- Method: `POST`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body:

  ```json
  {
      "phone": "13500000000"
  }
  ```

  *Paramters*

  | Key   | Description                         | Optional |
  | ----- | ----------------------------------- | -------- |
  | email | Email address the user want to link | False    |

- Response

  *Example 1*

  - Response Code: `403 Forbidden`

  - Response Body:

    ```json
    {
        "error": "User has already linked phone number"
    }
    ```

  *Example 2*

  - Response Code: `403 Forbidden`

  - Response Body:

    ```json
    {
        "error": "Phone number has already been linked to other account"
    }
    ```

  *Example 3*

  - Response Code: `200`

  - Response Body: 

    ```json
    {
        "message": "Verification code has been sent to phone number minakoto00@gmail.com"
    }
    ```


#### Link Email address verify code 

- Path: `family/oauth2/email`

- Method: `PUT`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body:

  ```json
  {
      "email": "example@example.org",
   	"code": "123456",
      "password": "abcd123456"
  }
  ```

- Response:

  *Example 1*

  - Response Code: `401 Unauthorized`

  - Resposne Body: 

    ```json
    {
        "error": "Verification code does not match"
    }
    ```

  *Example 2*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "message": "Email address example@example.org has been successfully linked to current account"
    }
    ```

#### Link phone number verify code

- Path: `family/oauth2/phone`

- Method: `PUT`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body:

  ```json
  {
      "phone": "13500000000",
   	"code": "123456",
      "password": "abcd123456"
  }
  ```

- Response:

  *Example 1*

  - Response Code: `401 Unauthorized`

  - Resposne Body: 

    ```json
    {
        "error": "Verification code does not match"
    }
    ```

  *Example 2*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "message": "Phone number 13500000000 has been successfully linked to current account"
    }
    ```



#### Unlink wechat account

- Path: `family/oauth2/wechat`

- Method: `DELETE`

- Authentication: **Required** 

- Request Headers: 

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Body: Empty

- Response

  *Example 1*

  - Response Code: `403 Forbidden	`

  - Response Body: 

    ```json
    {
        "error": "Unlinked wechat account not permitted, this user has not linked email address"
    }
    ```

  *Example 2*

  - Response Code: `204 No Content`

  - Response Body: 

    ````json
    {
        "message": "Successfully unlinked wechat account"
    }
    ````




### Lecture module APIs

#### Get lecture information

- Path: `family/lecture`

- Method: `GET`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Parameters: Empty

- Response:

  *Example 1*

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "active": [
            {
                "bannerUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToC/lectures/DB1A8FBEA7261A4A8F3F634D1D950E24/lecture_banner.png",
                "hostAvatarUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToC/lectures/DB1A8FBEA7261A4A8F3F634D1D950E24/host_avatar.png",
                "hostAvatarRectUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToC/lectures/DB1A8FBEA7261A4A8F3F634D1D950E24/host_avatar_rect.png",
                "recapVideoUri": null,
                "sharingPicUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToC/lectures/DB1A8FBEA7261A4A8F3F634D1D950E24/sharing_pic.png",
                "recapVideoTimeElapsed": null,
                "id": "1eea6c0c-3066-499e-aacb-2cf1b76f814a",
                "lectureName": "哥大学长：做一个有情怀有理想的学霸",
                "hostName": "赖文鹏",
                "hostTitle": "哥伦比亚大学国际与公共事务学院研究生\n共同未来公益项目副执行长",
                "hostTag": "哥大学长",
                "lectureTags": [
                    "学习效率",
                    "课外活动",
                    "公益",
                    "哥大"
                ],
                "hostIntro": "哥伦比亚大学国际与公共事务学院研究生在读，北京第二外国语学院管理学学士，以年级第一的成绩获得北京市优秀毕业生。他曾任中国首个获公募资格并致力于帮扶叙利亚难民的公益项目“共同未来”副秘书长，也曾作为中国代表赴韩国参与联合国年会，在非洲肯尼亚第二大贫民窟Mathare进行志愿服务，在以色列、约旦、土耳其的非营利组织、国际组织、难民营等实地调研。他曾作为TEDxLujiazui史上最年轻的讲者分享经历，也在联合国“一带一路”志愿服务论坛作为嘉宾分享经验。",
                "lectureIntro": "是否每次定下远大目标都踌躇满志，到了执行的时候却难以坚持？是否理想美好而遥不可及，制定实施步骤时却寸步难行？本期分享，来自哥伦比亚大国际与公共事务学院的学长赖文鹏，也是四年投身与中国与全球公益事业的公益人，将为我们讲述他是如何在保持高效的学习力的同时，一步一步实现自己的公益理想的。作为同龄人中的佼佼者，他也将独家分享在常青藤名校哥伦比亚大学，如何做一个既有高效学习力，又有情怀和理想的学霸。",
                "startTime": 1533960000000,
                "ongoing": true,
                "livePasscode": "Focus310"
            }
        ],
        "past": [
            {
                "bannerUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToC/lectures/D16BBCA7367B5A2349EC8EDBDE9374D6/lecture_banner.png",
                "hostAvatarUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToC/lectures/D16BBCA7367B5A2349EC8EDBDE9374D6/host_avatar.png",
                "hostAvatarRectUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToC/lectures/D16BBCA7367B5A2349EC8EDBDE9374D6/host_avatar_rect.png",
                "recapVideoUri": null,
                "sharingPicUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToC/lectures/D16BBCA7367B5A2349EC8EDBDE9374D6/sharing_pic.png",
                "recapVideoTimeElapsed": null,
                "id": "3fa74807-3bf2-4eac-8414-e4661d21eded",
                "lectureName": "非典型哈佛学霸的高效学习力",
                "hostName": "鲁林希",
                "hostTitle": "哈佛大学人类发展心理学硕士\n哈佛中国教育论坛主题故事负责人",
                "hostTag": "哈佛学姐",
                "lectureTags": [
                    "习惯",
                    "学习力",
                    "时间管理",
                    "效率"
                ],
                "hostIntro": "她是哈佛大学人类发展心理学硕士，精通中英德韩四国语言，中国青少年作家协会最年轻理事；她也是钢琴十级的乐队键盘手，精通十八般武艺，在中国进行上千场精彩演讲的独立演讲者；她更是公平教育的执着践行者，为美国麻省女子监狱的服役囚犯上心理课和学习辅导，帮助他们通过高中学历考试，重新收获自己的人生价值。",
                "lectureIntro": "好的学习习惯的养成是提高学习效率和学习成绩的关键。如何才能做到事半功倍，用最少的时间做最高效的事情？如何找准你的生理节律，正确地休息和学习？如何提高学习时间的有效性，巧妙运用碎片化与整体化的时间？如何对抗困扰你多年的拖延癌？在本节分享会中，哈佛学霸林希学姐将会为我们一一回答这些问题，并和我们独家分享，是怎样的高效率和高学习力，让她能够三年读完本科、以最小的年龄入读哈佛硕士学位并全A毕业。",
                "startTime": 1532750400000,
                "ongoing": false,
                "livePasscode": "Focus876"
            }
        ]
    }
    ```

    *Parameters*

    | Key    | Description                                                  |
    | ------ | ------------------------------------------------------------ |
    | active | Active incoming lectures. It's a Json array with one (or multiple) json objects |
    | past   | Past lectures (inactive). It's a Json array with one (or multiple) json objects |

    | Key                   | Description                                                  | Nullable | Type        |
    | --------------------- | ------------------------------------------------------------ | -------- | ----------- |
    | bannerUri             | URL of lecture banner picture                                | true     | `String`    |
    | hostAvatarUri         | URL of host's avatar (circular)                              | true     | `String`    |
    | hostAvatarRectUri     | URL of host's avatar (rectangular)                           | true     | `String`    |
    | recapVideoUri         | URL of recap video (on Aliyun OSS)                           | true     | `String`    |
    | sharingPicUri         | URL of lecture picture template which will be used for sharing picture generation | true     | `String`    |
    | recapVideoTimeElapsed | Time elapsed for recap video, will only be presented only after the recap video is updated | true     | `Long`      |
    | id                    | Server side id of Lecture Info record. Checking presentation of this field is **not required**. This field is mainly used for future Lecture Management web GUI implementation | false    | `String`    |
    | lectureName           | Name of the lecture                                          | false    | `String`    |
    | hostName              | Name of the Lecturer                                         | false    | `String`    |
    | hostTitile            | Titile of the host                                           | false    | `String`    |
    | hostTag               | Tag of the host, e.g. `哈佛学姐`                             | false    | `String`    |
    | lectureTags           | Tags of the lecture. The array will not be empty.            | false    | `JsonArray` |
    | hostIntro             | Introduction of the host                                     | false    | `String`    |
    | lectureIntro          | Introduction of the lecture                                  | false    | `String`    |
    | startTime             | Start time in millisenconds starting from 1970. Client side should parse this `Long` value in the Time Zone of GMT +8 (Beijing, Honkong, Taipei) | false    | `String`    |
    | ongoing               | Boolean flag that specifies if this lecutre is active        | false    | `Boolean`   |
    | livePasscode          | Live passcode of lecture on *千聊* (qlchat) live chat room   | true     | `Boolean`   |




### User feedback management

#### Post User Feedback

- Path: `family/feedback`

- Method: `POST`

- Authentication: **Required**

- Request Headers:

  | Header Name   | Content        |
  | ------------- | -------------- |
  | Authorization | Bearer *Token* |

- Request MIME Type: `multipart/form-data`

  > Info:
  >
  > This API includes picture uploading, so it's a multipart form

- Request Parameters:

  | Key             | Description                                                  | Optional | Type    |
  | --------------- | ------------------------------------------------------------ | -------- | ------- |
  | feedbackText    | Feedback text content                                        | False    | Text    |
  | feedbackPic     | Feedback picture file                                        | True     | Picture |
  | feedbackType    | Feedback type ordinal, see *Get feedback types* API for more information | False    | Int     |
  | contacts        | Contacts of the feedback sender                              | True     | Text    |
  | version         | Application version, for example `1.3.0`                     | False    | Text    |
  | firmwareVersion | Firmware version of Focus Headband                           | False    | Text    |
  | osVersion       | Operating system version, for example `iOS 11.4.1`           | False    | Text    |

  *Example*

  ```bash
  curl -X POST \
    http://localhost:8080/family/feedback \
    -H 'Authorization: Bearer eyJhbGciOiJI**H-O2T6Y' \
    -H 'Cache-Control: no-cache' \
    -H 'Content-Type: application/json' \
    -H 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \
    -F 'feedbackText=有bug' \
    -F feedbackPic=@/Users/brainco/Downloads/132.jpg \
    -F feedbackType=1 \
    -F contacts=minakoto00@gmail.com \
    -F version=1.3.0 \
    -F firmwareVersion=0.9.0
  ```

- Response

  *Example 1*

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    {
        "id": 12,
        "accountId": "6aebbcc2-d256-48bb-b918-3fc066407409",
        "feedbackText": "有bug",
        "feedbackPicUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToC/feedback/C20AD4D76FE97759AA27A0C99BFF6710.jpg",
        "receiptNumber": "FN02000012",
        "feedbackType": "OTHERS",
        "contacts": "minakoto00@gmail.com",
        "version": "1.3.0",
        "osVersion": "iOS 11.4.1",
        "firmwareVersion": "0.9.0"
    }
    ```




#### Get feedback types

- Path: `family/feedback/types`

- Method: `GET`

- Authentication: **Not requried**

- Request Headers:

  | Header Name  | Content          |
  | ------------ | ---------------- |
  | Content-type | application/json |

- Request Body: Empty

- Request Parameters: Empty

- Response

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    {
        "0": {
            "en": "SUGGESTIONS",
            "zh": "意见和建议"
        },
        "1": {
            "en": "BUGS",
            "zh": "程序错误"
        },
        "2": {
            "en": "OTHERS",
            "zh": "其他反馈"
        }
    }
    ```

    > Info:
    >
    > The response represents a map of ordinal numbers and its corresponding feedback types (including localized messages), The `0`, `1` or `2` should be used as parameters in *Post User feedback* API



#### Get All user feedbacks

- Path: `family/feedback`

- Method: `GET`

- Authentication: **Required**

- Roles Allowed: `SYSTEM_ADMIN`

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Parameters: empty

- Response:

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "statusOptions": [
            "OPEN",
            "IN_PROGRESS",
            "RESOLVED",
            "CLOSED"
        ],
        "feedbacks": [
            {
                "id": 14,
                "accountId": "6aebbcc2-d256-48bb-b918-3fc066407409",
                "feedbackText": "有bug",
                "feedbackPicUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToC/feedback/AAB3238922BCC25A6F606EB525FFDC56.jpg",
                "receiptNumber": "FN01000014",
                "feedbackType": "BUGS",
                "contacts": "minakoto00@gmail.com",
                "version": "1.3.0",
                "firmwareVersion": "0.9.0",
                "osVersion": "iOS 11.4.1",
                "status": "OPEN",
                "created": 1534793614998,
                "updated": 1534793620496
            },
            {
                "id": 15,
                "accountId": "6aebbcc2-d256-48bb-b918-3fc066407409",
                "feedbackText": "有bug",
                "feedbackPicUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToC/feedback/9BF31C7FF062936A96D3C8BD1F8F2FF3.jpg",
                "receiptNumber": "FN01000015",
                "feedbackType": "BUGS",
                "contacts": "minakoto00@gmail.com",
                "version": "1.3.0",
                "firmwareVersion": "0.9.0",
                "osVersion": "iOS 11.4.1",
                "status": "OPEN",
                "created": 1534793773808,
                "updated": 1534793778253
            }
        ]
    }
    ```

    *Parameters*

    | Key           | Description                                                  | Nullable |
    | ------------- | ------------------------------------------------------------ | -------- |
    | statusOptions | Feedback resolve status options, can be passed as parameter in updating feedback status API | False    |
    | feedbacks     | JSON array of all user submitted feedbacks                   | False    |



#### Update user feedback status

- Path: `family/feedback`

- Method: `PATCH`

- Authentication: **Required**

- Roles Allowed: `SYSTEM_ADMIN`

- Request Headers:

  | Header Name   | Content          |
  | ------------- | ---------------- |
  | Content-type  | application/json |
  | Authorization | Bearer *Token*   |

- Request Parameters:

  | Name   | Description                                                  | Type         |
  | ------ | ------------------------------------------------------------ | ------------ |
  | id     | Server id used to identify certain feedback record.          | Integer      |
  | status | Feedback type options return in *Get all user feedback* API, for example`OPEN`, `CLOSED`, `RESOLVED`, `IN_PROGRESS` | String, Enum |

- Response

  - Response Code: `200 OK`

  - Response Body: 

    ```json
    {
                "id": 14,
                "accountId": "6aebbcc2-d256-48bb-b918-3fc066407409",
                "feedbackText": "有bug",
                "feedbackPicUri": "https://focus-resource.oss-cn-beijing.aliyuncs.com/ToC/feedback/AAB3238922BCC25A6F606EB525FFDC56.jpg",
                "receiptNumber": "FN01000014",
                "feedbackType": "BUGS",
                "contacts": "minakoto00@gmail.com",
                "version": "1.3.0",
                "firmwareVersion": "0.9.0",
                "osVersion": "iOS 11.4.1",
                "status": "CLOSED",
                "created": 1534793614998,
                "updated": 1534793620496
            }
    ```





### Push notification service (iOS)

#### Register device token when launching

- Path: `family/notification/device`

- Method: `PUT`

- Authentication: **Not Required**

- Request header:

  | Header Name  | Content          |
  | ------------ | ---------------- |
  | Content-type | application/json |

- Request Body

  ```json
  {
      "deviceToken": "..."
  }
  ```

  *Parameters*

  | Key         | Description                                                  | Type   | Optional |
  | ----------- | ------------------------------------------------------------ | ------ | -------- |
  | deviceToken | Token string that an iOS device fetched from Apple's APNS server | String | False    |

- Response:

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "message": "Updated Successfully"
    }
    ```



#### Register device token to certain user (Currently support 1 device per user)

- Path: `family/notification/device`

- Method: `POST`

- Authentication: **Required**

- Request Header:

  | Header Name    | Content          |
  | -------------- | ---------------- |
  | Content-type   | application/json |
  | Authentication | Bearer *Token*   |

- Request Body

  ```json
  {
      "deviceToken": "..."
  }
  ```

  *Parameters*

  | Key         | Description                                                  | Type   | Optional |
  | ----------- | ------------------------------------------------------------ | ------ | -------- |
  | deviceToken | Token string that an iOS device fetched from Apple's APNS server | String | False    |

- Response:

  - Response Code: `200 OK`

  - Response Body:

    ```json
    {
        "message": "Updated Successfully"
    }
    ```



#### Broadcast notification to registered devices

- Path: `family/notification/broadcast`

- Method: `POST`

- Authentication: **Required**

- Roles Allowed: `SYSTEM_ADMIN`

- Request Headers:

  | Header Name    | Content          |
  | -------------- | ---------------- |
  | Content-type   | application/json |
  | Authentication | Bearer *Token*   |

- Request Body

  ```json
  {
      "title": "Test title",
      "body": "Test body",
      "subtitle": "Test subtitle"
  }
  ```

  *Parameters*

  | Key      | Description                | Type   | Optional |
  | -------- | -------------------------- | ------ | -------- |
  | title    | Notification title text    | String | True     |
  | body     | Notification body text     | String | False    |
  | subtitle | Notification subtitle text | String | True     |

- Response

  - Response Code: `200 OK`
  - Response Body: Empty