
# API Description

This API allow you to connect with Keno Revenda system.

The test environment URL is:

- https://revenda.kenominas.com.br/beta/ - Dashboard

- https://revenda.kenominas.com.br/beta/api - API Endpoint

The production environment URL is:

- https://revenda.kenominas.com.br/ - Dashboard

- https://revenda.kenominas.com.br/api - API Endpoint


*** IMPORTANT NOTICE ***
This API is still under development and currently are in beta testing. Production environments are not ready.


## Account Types

The API was built with 2 accounts type in mind:

- Distributor - Accounts which will register tickets in their own name. Those accounts work with a credit limit and once per week an invoice is generated with the balance between tickets cost and tickets prizes.
- Reseller - Accounts which will register tickets for customers. After sending a ticket purchase request to the API, those account will receive a PIX QR Code for payment, after the payment is identified by our system, the ticket is then generated and sent back to the seller. Prizes are automatically sent to the given PIX DICT Key.

__Regardless the account type, prizes which requires Tax Retention, will require manual review from our side. If you are a Distributor, at that point you will need to inform detailed information about the ticket owner for prize payment through our Back Office, this ticket prize will not count in the weekly invoice, but it's cost will.__


## Authentication

All requests to the API should be sent as a regular application/x-www-form-urlencoded POST request. Every request must inform the authentication details, as follow:

- account_id : Your account ID found at the Dashboard
- key : The API key found at the Dashboard

Endpoins which accept additional arguments, should use:
- $variable_name : $variable_value

All this data must be sent as a paylod in JSON format.

Besides the API Key authentication, you should also whitelist your origin IPs at the Dashboard. Doing so will guarantee that only your IPs can act at your behalf.

## API Limitations

Currently we don't impose limits or throttle requests ratio on our API. 

TBI/TBD - API Block and Throttling due to multiple requests.

## Endpoints

### /account_info
- Available for All

This endpoint will retrieve information regarding your own account, including credit limit and account status. This can be useful to predict what type of transactions you will be able to do.

**Sample Request**
```json
curl 'https://revenda.kenominas.com.br/beta/api/account_info/' \
  --data-raw '{"account_id":2,"key":"QcgA2.J8qz2vnzxIIco4U2m8M5z0gNmIbUM"}'
```

**Sample Response**
```json
{
    "id": 2,
    "status": "active",
    "allowed_ips": [
        "129.100.100.1",
        "2001:0db8:85a3:0000:0000:8a2e:0370:7334",
        "172.20.0.1"
    ],
    "credit": {
        "total": 0,
        "available": 0,
        "due": 0,
        "pastdue": 0
    }
}
```

### /ticket/new 
- Available for Distributor

This endpoint will allow the register of a new ticket.

**Expected Variables**
- numbers : *(required)* A string containing the desired numbers separated by comma, the list must have at least one number and at max 10 numbers, between 1 and 80.
- multiply : *(required)* An integer with the desired value to multiple the bet. Allowed values: 1, 2, 3, 4, 5, 6, 8, 10, 12, 20
- draws : *(required)* An integer with the desired value of draws which this bet should participate. Allowed values: 1, 2, 3, 4, 5, 6, 10, 20, 50, 100, 200
- golden : *(required)* A boolean value indicating if the bet will use Golden Ball/Bull's Eye. Any value besides *true* will not use it.
- price : *(optional)* An integer value representing the calculated cost of the ticket. If informed, the calculated price must match the real price which will be charged, otherwise the ticket will not be generated.

**Sample Request**
```json
curl 'https://revenda.kenominas.com.br/beta/api/ticket/new/' \
  --data-raw '{"account_id":2,"key":"QcgA2.J8qz2vnzxIIco4U2m8M5z0gNmIbUM","price":24,"numbers":"2,10,34,56","multiply":3,"draws":"2","golden":true}'
```
**Sample Response**
```json
{
    "serial": "000820193600007864320585689757",
    "registeredDate": "20/08/2022 12:39:52",
    "draws": {
        "amount": 1,
        "firstDraw": 1025242,
        "firstDrawTime": "20/08/2022 12:40:00",
        "lastDraw": 1025242,
        "lastDrawTime": "20/08/2022 12:40:00"
    },
    "bet": {
        "numbers": [
            2,
            10,
            34,
            56
        ],
        "multiply": 3,
        "golden": true
    },
    "price": 12,
    "award": {
        "settled": false,
        "prize": null,
        "tax": null,
        "prizeFinal": null,
        "paid": null
    }
}
```

### /ticket/new/batch
- Available for Distributor

This endpoint will allow the register of multiple new tickets. The tickets will be registered in the order they are sent. There is no guarantee that all the tickets will have the same start_draw. In the case where any error occurs, due to any reason, prior to all tickets being generated, the already registered tickets will not be canceled. This endpoint does not check for the total credit which will be used, in the case the credit isn't enough for a particular ticket in the middle of the process, that particular ticket will be skipped. The response includes the field `errors` which is an array with all the non blockable errors which happened during the process.

**Expected Variables**
- tickets : *(required)* An array containing an array with the following fields:
    - numbers : *(required)* A string containing the desired numbers separated by comma, the list must have at least one number and at max 10 numbers, between 1 and 80.
    - multiply : *(required)* An integer with the desired value to multiple the bet. Allowed values: 1, 2, 3, 4, 5, 6, 8, 10, 12, 20
    - draws : *(required)* An integer with the desired value of draws which this bet should participate. Allowed values: 1, 2, 3, 4, 5, 6, 10, 20, 50, 100, 200
    - golden : *(required)* A boolean value indicating if the bet will use Golden Ball/Bull's Eye. Any value besides *true* will not use it.
    - price : *(optional)* An integer value representing the calculated cost of the ticket. If informed, the calculated price must match the real price which will be charged, otherwise the ticket will not be generated.

**Sample Request**
```json
curl 'https://revenda.kenominas.com.br/beta/api/ticket/new/batch' \
  --data-raw '{"account_id":2,"key":"QcgA2.J8qz2vnzxIIco4U2m8M5z0gNmIbUM","tickets":[{"price":24,"numbers":"2,10,34,56","multiply":3,"draws":"2","golden":true}, {"price":12,"numbers":"1,2,3,4","multiply":3,"draws":"2","golden":true}, {"price":24,"numbers":"22,35,40,90","multiply":3,"draws":"2","golden":true}, {"price":24,"numbers":"22,35,40","multiply":3,"draws":"2","golden":true}]}'
```
**Sample Response**
```json
{
    "tickets": [
        {
            "serial": "365543424732212254722184096529",
            "registeredDate": "02/09/2022 20:56:59",
            "draws": {
                "amount": 2,
                "firstDraw": 1029059,
                "firstDrawTime": "02/09/2022 21:00:00",
                "lastDraw": 1029060,
                "lastDrawTime": "02/09/2022 21:04:00"
            },
            "bet": {
                "numbers": [
                    2,
                    10,
                    34,
                    56
                ],
                "multiply": 3,
                "golden": true
            },
            "price": 24,
            "award": {
                "settled": false,
                "prize": null,
                "tax": null,
                "prizeFinal": null,
                "paid": null
            }
        },
        {
            "serial": "201955787800052428802342573035",
            "registeredDate": "02/09/2022 20:56:59",
            "draws": {
                "amount": 2,
                "firstDraw": 1029059,
                "firstDrawTime": "02/09/2022 21:00:00",
                "lastDraw": 1029060,
                "lastDrawTime": "02/09/2022 21:04:00"
            },
            "bet": {
                "numbers": [
                    22,
                    35,
                    40
                ],
                "multiply": 3,
                "golden": true
            },
            "price": 24,
            "award": {
                "settled": false,
                "prize": null,
                "tax": null,
                "prizeFinal": null,
                "paid": null
            }
        }
    ],
    "errors": [
        "Error on ticket key {1}: Wrong ticket price. Calculated ticket price: 24",
        "Error on ticket key {2}: Invalid number selected: 90"
    ]
}
```

### /ticket/view
- Available for All

This endpoint will allow viewing all details about a ticket belonging to this account.

**Expected Variables**
- serial : *(required)* A string containing the ticket serial.

**Sample Request**
```json
curl 'https://revenda.kenominas.com.br/beta/api/ticket/new/' \
  --data-raw '{"account_id":2,"key":"QcgA2.J8qz2vnzxIIco4U2m8M5z0gNmIbUM","serial":"000820193600007864320585689757"}'
```
**Sample Response**
```json
{
    "serial": "000820193600007864320585689757",
    "registeredDate": "20/08/2022 12:39:52",
    "draws": {
        "amount": 1,
        "firstDraw": 1025242,
        "firstDrawTime": "20/08/2022 12:40:00",
        "lastDraw": 1025242,
        "lastDrawTime": "20/08/2022 12:40:00"
    },
    "bet": {
        "numbers": [
            2,
            10,
            34,
            56
        ],
        "multiply": 3,
        "golden": true
    },
    "price": 12,
    "award": {
        "settled": true,
        "prize": 0,
        "tax": 0,
        "prizeFinal": 0,
        "paid": null
    }
}
```

### /ticket/image
- Available for All

This endpoint will allow viewing the ticket in image format.

**Expected Variables**
- serial : *(required)* A string containing the ticket serial.

**Sample Request**
```json
curl 'https://revenda.kenominas.com.br/beta/api/ticket/new/' \
  --data-raw '{"account_id":2,"key":"QcgA2.J8qz2vnzxIIco4U2m8M5z0gNmIbUM","serial":"000820193600007864320585689757"}'
```

**Sample Response**
Response will be an image in PNG format.

### /draws
- Available for All

This endpoint will retrieve details about a specific draw or the last 10 draws.

**Expected Variables**
- draw : *(optional)* An integer, if informed will show the result for that draw

**Sample Request**
```json
curl 'https://revenda.kenominas.com.br/beta/api/draws/' \
  --data-raw '{"draw":973294}'
```
**Sample Response**
```json
[
    {
        "id": 973294,
        "date": "20/02/2022 07:00:00",
        "numbers": "17,50,04,09,73,13,72,56,36,24,43,71,29,42,22,67,54,27,75,68"
    }
]
```

### /ticket/order
- Available for Reseller

This endpoint will allow the register of a new pre-paid ticket for customers. The ticket will only be registered after payment. They payment can be done through a PIX QR Code or PIX Copy & Paste which will be provided in the response of this endpoint.

**Expected Variables**
- name : *(required)* A string containing the full customer name, including first and last name, limited to 255 characters.
- cpf : *(required)* A string containing customer CPF.
- cep : *(required)* A string containing customer CEP. It must be between 30000-000 and 39999-999.
- phone : *(required)* A string containing customer Cell Phone Number.
- pix : *(required)* A string with the customer PIX Key. If the generated ticket is awarded, the payment will be done through this PIX key. The Pix Key must be associated with the customer CPF informed above. This key should be first validated through the endpoint `/pix_info`
- numbers : *(required)* A string containing the desired numbers separated by comma, the list must have at least one number and at max 10 numbers, between 1 and 80.
- multiply : *(required)* An integer with the desired value to multiple the bet. Allowed values: 1, 2, 3, 4, 5, 6, 8, 10, 12, 20
- draws : *(required)* An integer with the desired value of draws which this bet should participate. Allowed values: 1, 2, 3, 4, 5, 6, 10, 20, 50, 100, 200
- golden : *(required)* A boolean value indicating if the bet will use Golden Ball/Bull's Eye. Any value besides *true* will not use it.
- price : *(optional)* An integer value representing the calculated cost of the ticket. If informed, the calculated price must match the real price which will be charged, otherwise the ticket will not be generated.
- sms : *(optional)* If equal to 1, an SMS will be sent to the user with the ticket and when the ticket is settled.

**Sample Request**
```json
curl 'https://revenda.kenominas.com.br/beta/api/ticket/new/' \
  --data-raw '{
  "account_id": 2,
  "key": "Ruaxz.fKZKWatbX93ExsmWlDiSlwcocJoq1",
  "name": "Jon Snow",
  "cpf": "12345678910",
  "cep": "39950000",
  "phone": "31999999999",
  "multiply": 1,
  "numbers": "13,15,24,26,35,46,47,71,72,73",
  "draws": 1,
  "golden": 1,
  "sms": "1"
}'
```

**Response Variables**
- order : An integer representing the Order ID created.
- expiration : Timestamp with the time limit to pay the QR Code.
- qrcode : The Text representation of the QR Code for payment.
- qrcode_img : A URL where you can get the image representing the above QR Code as image.

**Sample Response**
```json
{
  "order": 3,
  "expiration": "2022-11-07T23:08:54+00:00",
  "qrcode": "00020101021226790014br.gov.bcb.pix2557invoice.starkbank.com/v2/30432bac47b545d68f9bf815c62b939f5204000053039865802BR5925Intralot Do Brasil Comerc6014Belo Horizonte62070503***6304BED6",
  "qrcode_img": "https://revenda.kenominas.com.br/invoice-qrcode/6424804695474176"
}
```

### /ticket/order/view
- Available for Reseller

This endpoint will allow to check all the details about an order.

**Expected Variables**
- order_id : *(required)* The Order ID to look for.

**Sample Request**
```json
curl 'https://revenda.kenominas.com.br/beta/api/ticket/new/' \
  --data-raw '{
  "account_id": 2,
  "key": "Ruaxz.fKZKWatbX93ExsmWlDiSlwcocJoq1",
  "order_id": 3
}'
```

**Response Variables**
- order : An integer representing the Order ID created.
- value : The cost of the order
- date : Timestamp with the time when order was created.
- serial : The ticket serial number. It will be null until order is paid and ticket is generated.
- multiply : The multiplier selected
- numbers : The numbers selected
- draws : The draws amount selected
- golden : The golden selection

**Sample Response**
```json
{
  "id": 3,
  "value": 4,
  "date": "2022-11-07 20:02:54.000000",
  "status": "canceled",
  "serial": null,
  "multiply": 1,
  "numbers": [
    13,
    15,
    24,
    26,
    35,
    46,
    47,
    71,
    72,
    73
  ],
  "draws": 1,
  "golden": 1
}
```

### /ticket/pix
- Available for Reseller

This endpoint will allow to set the DICT key for a Prized order. After set, the order will enter in a queue for prize payment. If the payment cannot be done for any reason, but most likely due to bank issues reason, order will be marked as "manual-payment" and the ticket will need to pass through a soft manual review for receiving the payment.

**Expected Variables**
- cpf : *(required)* The user CPF. It must match the CPF which registered the ticket.
- pix_key : *(required)* The PIX DICT Key. It can be any valid PIX key, but it must belong to the same CPF which registered the ticket. Attention for phone numbers, which must be sent in the international format: +5511999999999
- serial : *(required)" The prized serial ticket.

**Sample Request**
```json
curl 'https://revenda.kenominas.com.br/beta/api/ticket/new/' \
  --data-raw '{
  "account_id": 2,
  "key": "Ruaxz.fKZKWatbX93ExsmWlDiSlwcocJoq1",
  "cpf": "12345678910",
  "pix_key": "pix@email.com",
  "serial": "415734172700060293120146413327"
}'
```

**Response Variables**
- serial : String with the Serial Number
- pix_key : String with the DICT Key
- bank : Bank name collected from the DICT Key
- success : Boolean to determine if the Pix is valid and if the ticket entered the queue for payment

**Sample Response**
```json
{
    "serial": "415734172700060293120146413327",
    "pix_key": "pix@email.com",
    "bank" : "ITAU UNIBANCO",
    "success": true
}
```

### /ticket/list
- Available for All

This endpoint will retrieve details about all tickets generated by your account

**Expected Variables**
- day : *(optional)* A single day in format YYYY-MM-DD to retrieve notifications from that day alone
- status : *(optional)" The status of the ticket. Can be any of:
-- all : return all tickets
-- pending : return all tickets which still have pending draws
-- finished : return all tickets which have already finished draws
-- manual-award : return all tickets with some prize with tax to receive
-- awarded : return all tickets with some prize to receive
-- unawarded : return all tickets without prizes
-- settled : return all tickets which prizes were paid
-- unsettled : return all tickets which prizes are still pending payment
- page : *(optional)* For paginated results. API will return 100 entries per page in ascending order by date

**Sample Request**
```json
curl 'https://revenda.kenominas.com.br/beta/api/ticket/new/' \
  --data-raw '{
  "account_id": 2,
  "key": "Ruaxz.fKZKWatbX93ExsmWlDiSlwcocJoq1",
  "status": "unawarded"
}'
```

**Response Variables**
- Array of:
-- serial : String with the Serial Number
-- date : Timestamp with the time when serial was created.
-- status : The current serial status

**Sample Response**
```json
[
    {
        "serial": "003348528004194304002342531931",
        "date": "2022-11-04 18:50:44.000000",
        "status": "unawarded"
    }
]
```

### /notifications/pending
- Available for All

This endpoint will retrieve details about all notifications which are still pending from your account.

**Expected Variables**
- day : *(optional)* A single day in format YYYY-MM-DD to retrieve notifications from that day alone
- type : *(optional)* Retrieve only notifications from that type. More information on Notifications section below.
- page : *(optional)* For paginated results. API will return 100 entries per page in ascending order by date

### /notifications/failed
- Available for All

This endpoint will retrieve details about all notifications which failed to be submitted from your account.

**Expected Variables**
- day : *(optional)* A single day in format YYYY-MM-DD to retrieve notifications from that day alone
- type : *(optional)* Retrieve only notifications from that type. More information on Notifications section below.
- page : *(optional)* For paginated results. API will return 100 entries per page in ascending order by date

### /notifications/sent
- Available for All

This endpoint will retrieve details about all notifications which were sent from your account.

**Expected Variables**
- day : *(optional)* A single day in format YYYY-MM-DD to retrieve notifications from that day alone
- type : *(optional)* Retrieve only notifications from that type. More information on Notifications section below.
- page : *(optional)* For paginated results. API will return 100 entries per page in ascending order by date



## Webhooks 
You can setup a public reachable URL on the options for the Webhook functionality. This URL must be secured by a valid SSL. After activating an URL, we'll send every future notification to that URL.

We expect a header response status 200 to consider the notification as sent. In case of failure, we'll try to re-submit it 2 other times, after a few seconds. Most of the notifications are time-sensitive, for this reason we don't intend to create large intervals between notifications. If your application relies on a notification to work, you must ensure to have a fallback which will query the notifications endpoint to check for any non-submitted notification.

### Authenticating the Notification

Every notification submission will have a header 'Digital-Signature', this digital signature can be verified using our Public Key found at the /public-key endpoint. To do so:
- Read the Digital-Signature header
- Decode the Digital-Singature at base64
- Confirm the signature with OpenSSL using the decoded signature with our public key.

### Notifications

The notification will be sent as body POST to your URL. It will always be a JSON Encoded string with a few elements. Each notification have a specific structure as showed below, but all of them share at least 2 variables:
- type : Usually the notification category based in the object referenced
- action : The action which happened with the object

#### A ticket draws are settled
This notification will be sent whenever a ticket have it's draws settled

**Provided Variables**
- type : "ticket"
- action : "finished-draws"
- status : "unawarded" for not winning tickets, "prized" for winning tickets or "manual-award" for winning tickets with tax to be collected
- award : Gross amount of the award
- ticket : The ticket serial number

**Sample Notification**
```json
{
    "type": "ticket",
    "action": "finished-draws",
    "status": "prized",
    "award": 4,
    "ticket": "000820193600007864320585689757"
}
```

#### A order is paid
This notification will be sent whenever a ticket order (created using the endpoint /ticket/order) is paid by the customer.

**Provided Variables**
- type : "order"
- action : "generated"
- order_id : The Order ID which got paid
- ticket : The ticket object generated after the payment

**Sample Notification**
```json
{
    "type": "order",
    "action": "generated",
    "order_id": 1,
    "ticket": {
        "serial": "000820193600007864320585689757",
        "registeredDate": "20/08/2022 12:39:52",
        "draws": {
            "amount": 1,
            "firstDraw": 1025242,
            "firstDrawTime": "20/08/2022 12:40:00",
            "lastDraw": 1025242,
            "lastDrawTime": "20/08/2022 12:40:00"
        },
        "bet": {
            "numbers": [
                2,
                10,
                34,
                56
            ],
            "multiply": 3,
            "golden": true
        },
        "price": 12,
        "award": {
            "settled": false,
            "prize": null,
            "tax": null,
            "prizeFinal": null,
            "paid": null
        }
    }
}
```

#### A ticket order draws are settled
This notification will be sent whenever all the draws which a ticket from an order participates is settled. At this moment we also know if the ticket won a prize or not, and how much.

**Provided Variables**
- type : "order"
- action : "drawn"
- status : "unawarded" for not winning tickets, "prized" for winning tickets or "manual-award" for winning tickets with tax to be collected
- order_id : The Order ID
- ticket : The ticket object generated after the payment

**Sample Notification**
```json
{
    "type": "order",
    "action": "drawn",
    "status": "unawarded",
    "order_id": 1,
    "ticket": {
        "serial": "000820193600007864320585689757",
        "registeredDate": "20/08/2022 12:39:52",
        "draws": {
            "amount": 1,
            "firstDraw": 1025242,
            "firstDrawTime": "20/08/2022 12:40:00",
            "lastDraw": 1025242,
            "lastDrawTime": "20/08/2022 12:40:00"
        },
        "bet": {
            "numbers": [
                2,
                10,
                34,
                56
            ],
            "multiply": 3,
            "golden": true
        },
        "price": 12,
        "award": {
            "settled": true,
            "prize": 0,
            "tax": 0,
            "prizeFinal": 0,
            "paid": null
        }
    }
}
```

#### The prize payment of a ticket order is paid/reviewed
This notification will be sent after the payment of a prize is made or after a prize payment is withhold for further review before release.

**Provided Variables**
- type : "order"
- action : "awarded"
- status : "awarded" for prizes already paid or "manual-award" for prizes which will need a manual review.
- order_id : The Order ID
- ticket : The ticket object generated after the payment
- payment : The proof of the prize payment

**Sample Notification**
```json
{
    "type": "order",
    "action": "updated",
    "status": "awarded",
    "order_id": 1,
    "ticket": {
        "serial": "000820193600007864320585689757",
        "registeredDate": "20/08/2022 12:39:52",
        "draws": {
            "amount": 1,
            "firstDraw": 1025242,
            "firstDrawTime": "20/08/2022 12:40:00",
            "lastDraw": 1025242,
            "lastDrawTime": "20/08/2022 12:40:00"
        },
        "bet": {
            "numbers": [
                2,
                10,
                34,
                56
            ],
            "multiply": 3,
            "golden": true
        },
        "price": 12,
        "award": {
            "settled": true,
            "prize": 10,
            "tax": 0,
            "prizeFinal": 10,
            "paid": true
        }
    },
    "payment": {}
}
```


## Understanding Order Flow

![order flow](https://revenda.kenominas.com.br/beta/assets/img/order-flow.jpeg)


## Error Handling
While reading the response you should always check the Header Status Code. A code different from 200 means that you have an error on your request and it wasn't processed.
You may get the body response text to get more information about  the error, it's always returned as a JSON object with a single variable. Example:
```json
{
    "message": "Not enough credits"
}
```
