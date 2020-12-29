# fundAPI
## overview
REST Api for transferring money between 2 accounts with different currencies.

the exchange rate is caclcuted using https://exchangeratesapi.io/

## PROJECT 
Technologies : `JAVA1.8, Spring, H2|MySQL, JUnit, Git`

Installation : `mvn spring-boot:run`

Test : `mvn test`

## Data Base
The project configured and ready to be started with in-memory db H2.

Since H2 is an experimental db, in Order to be able to run the concurrent tests provided in
TransferServiceTest we need to switch to a db that support multi-threading.

the project is prepared to run with mysql by doing the following :
 
- un-comment the first 5 lines in application.properties 
- adjust to the datasource url, username, password
- un-comment @TEST from transferMoneyConcurrent & transferMoneyConcurrentWrapper
- re-build and run the tests

## API
### Endpoints

### 1 - Show Accounts

***Method*** : `GET` 

***URL*** : `/api/v1/accounts/`

***curl*** : `curl -i 'http://localhost:8080/api/v1/accounts' `

***Success Response***

***Code*** : `200 OK` 

***Content*** : 
```
{   
    "responseCode":"OK",
    "payload":"[{id:1,owner:1,currency:eur,balance:100.00},
                {id:2,owner:2,currency:usd,balance:500.00},
                {id:3,owner:3,currency:eur,balance:303.00},
                {id:4,owner:4,currency:eur,balance:404.00}]",
    "timestamp":"29-12-2020 02:43:16"
}
```

### 2 - Show Account Detail

***Method*** : `GET`

***URL*** : `/api/v1/accounts/:id`

***curl*** : ```curl -i 'http://localhost:8080/api/v1/accounts/1' ```

***Success Response***

***Code*** : `200 OK`

***Content*** : ```{"responseCode":"OK","payload":"{id:1,owner:1,currency:eur,balance:100.00}","timestamp":"29-12-2020 02:45:32"}```

***Error Responses***
***Code*** : `400 BAD REQUEST`
```
# curl -i 'http://localhost:8080/api/v1/accounts/200
{"errorMessage":"No account exist with the given id: 200","errorCode":"Bad Request","timestamp":"29-12-2020 12:49:46"}
```


### 3 - Create Account

***Method*** : `POST`

***URL*** : `/api/v1/accounts

***curl*** : ```curl -H "Content-Type: application/json" -X POST -d '{"owner":"11","currency":"EUR","balance":1000}' http://localhost:8080/api/v1/accounts ```

***HEADER*** : `Content-Type: application/json`

***DATA*** : `{"owner":7,"currency":"EUR","balance":533.00}`

***Success Response***

***Code*** : `200 OK`

***Content*** : 
```
{
    "responseCode":"OK",
    "payload":"{id:5,owner:11,currency:EUR,balance:1000}",
    "timestamp":"29-12-2020 02:46:47"
}
```

***Error Responses***
***Code*** : `400 BAD REQUEST`
```
# curl -H "Content-Type: application/json" -X POST -d '{"owner":"","currency":"EUR","balance":1000}' http://localhost:8080/api/v1/accounts
{"errorMessage":"Given Owner null is not valid","errorCode":"Bad Request","timestamp":"29-12-2020 12:57:34"}

# curl -H "Content-Type: application/json" -X POST -d '{"owner":"11","currency":"GGG","balance":1000}' http://localhost:8080/api/v1/accounts
{"errorMessage":"Given currency GGG is not valid","errorCode":"Bad Request","timestamp":"29-12-2020 12:58:25"}

# curl -H "Content-Type: application/json" -X POST -d '{"owner":10,"currency":"EUR","balance":-10}' http://localhost:8080/api/v1/accounts
{"errorMessage":"Given Balance-10 is not valid","errorCode":"Bad Request","timestamp":"29-12-2020 12:53:53"}
```

### 4 - Delete Account

***Method*** : `DELETE`

***URL*** : `/api/v1/accounts/:id

***curl*** : ```curl -i -X DELETE  http://localhost:8080/api/v1/accounts/1 ```

***Success Response***

***Code*** : `200 OK`

***Error Response***
***Code*** : `400 BAD REQUEST`
```
# curl -i -X DELETE  http://localhost:8080/api/v1/accounts/200 
  {"errorMessage":"No account exist with the given id: 200","errorCode":"Bad Request","timestamp":"29-12-2020 01:00:43"}%
```


### 5 - Transfer Money

***Method*** : `POST`

***URL*** : `/api/v1/transfer

***curl*** : ```curl -i -H "Content-Type: application/json" -X POST  -d '{"source":"2","target":"3","amount":"100"}'  http://localhost:8080/api/v1/transfer ```

***HEADER*** : `Content-Type: application/json`

***DATA*** : ```{"source":"2","target":"199","amount":"10"}```

***Success Response***

***Code*** : `200 OK`

***Error Responses***
***Code*** : `400 BAD REQUEST`
```
# curl -i -H "Content-Type: application/json" -X POST  -d '{"source":"222","target":"3","amount":"100"}'  http://localhost:8080/api/v1/transfer
{"errorMessage":"No account exist with the given id: 222","errorCode":"Bad Request","timestamp":"29-12-2020 01:09:27"}

# curl -i -H "Content-Type: application/json" -X POST  -d '{"source":"2","target":"333","amount":"100"}'  http://localhost:8080/api/v1/transfer
{"errorMessage":"No account exist with the given id: 333","errorCode":"Bad Request","timestamp":"29-12-2020 01:09:57"}

# curl -i -H "Content-Type: application/json" -X POST  -d '{"source":"2","target":"3","amount":"-20"}'  http://localhost:8080/api/v1/transfer
{"errorMessage":"Given amount -20 is not valid","errorCode":"Bad Request","timestamp":"29-12-2020 01:10:54"}%

# curl -i -H "Content-Type: application/json" -X POST  -d '{"source":"2","target":"3","amount":"100"}'  http://localhost:8080/api/v1/transfer
{"errorMessage":"Insufficient balance in source account","errorCode":"Bad Request","timestamp":"29-12-2020 01:06:17"}

```
## Handle method not found
***General Responses***

***Code*** : `404 NOT FOUND`
```
# curl -i 'http://localhost:8080/api/v1/accounts/get'
{"timestamp":"2020-12-29T03:33:02.589+00:00","status":400,"error":"Bad Request","message":"","path":"/api/v1/accounts/get"}
```

## TEST
Integration TESTS provided for AccountService and TransferService.

TransferServiceTest : there is 2 commented TESTs to be run against multithreaded db in order
to verify the behaviour in case of deadlocks caused by thread concurrency.


