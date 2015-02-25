# VOLABIT API DOCUMENTATION

The following document is divided into three parts. Part I describes how to gain access to the swagger documentation, Part II describes how to generate your first token and how to refresh your token and Part III covers the API calls that can be made as well as some implementation tips. 

## PART I. ACCESS [SWAGGER](https://github.com/swagger-api/swagger-ui) DOCUMENTATION.
 The swagger documentation provides an overview of all the methods with their parameters and expected responses.

  1. Go to https://www.volabit.com/api-docs/
  2. Enter the following APP ID `a47f325d0718ccd3e8aa1ec29c47d6d3a2cf18bca639b3fb1014b0362d163d06` and click save.
  3. Sign into your account or create one and click on Authorize button.

  To get full access to the API we will require you send us some KYC/AML documents. Please contact support@volabit.com for further instructions. 

## PART II. CREATING AN APP AND MANAGING TOKENS
  Once the paperwork has been cleared it is time to generate an Application ID with its secret key and use these to generate a token and all subsequent token refreshes.

  Go to https://www.volabit.com/oauth/applications and create an application with the default Redirect URL `urn:ietf:wg:oauth:2.0:oob`.
  In the following view, click Authorize, follow instructions until you are provided with an authorization code. 

  if you like to do this programmatically you can redirect your users to `https://www.volabit.com/oauth/authorize?client_id=APP_ID&redirect_uri=REDIRECT_URL&response_type=code`.

  This authorization code (`AUTH_CODE`) will be used to generate a `access_token` and a `refresh_token` with the following call:

  ```bash
    curl -F grant_type=authorization_code \
    -F client_id=APP_ID \
    -F client_secret=APP_SECRET \
    -F code=AUTH_CODE \
    -F redirect_uri=urn:ietf:wg:oauth:2.0:oob \
    -X POST https://www.volabit.com/oauth/token
  ```

  Tokens can be used to make calls until they expire **every 8 hours**, afterwards you will use the `refresh_token` to generate a new token.

  ```bash
    curl -F grant_type=refresh_token \
    -F client_id=APP_ID \
    -F client_secret=APP_SECRET \
    -F refresh_token=REFRESH_TOKEN \
    -X POST https://www.volabit.com/oauth/token
  ```
  
  You can check your token info in
  
  ```bash
    curl -H "Authorization: Bearer YOUR_TOKEN" \
      https://www.volabit.com/oauth/token/info
  ```

  **Tip: Check for an expired token in every call you make**

  Now that you can create and refresh tokens, lets move on to the interesting part.

## PART III: API IMPLEMENTATION

  **For partner services**

  - Send money to a user in Volabit, the user might or might not be registered in volabit.com
  - Request money from a Volabit user.

  Required EndPoints:
  1. `api/v1/users [POST]`
  2. `api/v1/users/send [POST]`
  3. `api/v1/users/requests [POST]`
  4. `api/v1/users/requests/methods [POST]`

### Create User `api/v1/users [POST]`

  For a seamless money sending and receiving experience Volabit requires you to register your customer as a Volabit user.

  After you have created a Volabit user you can proceed to **push** and **request** payments from him/her.

### Send Money to a User `api/v1/users/send [POST]`

  In order to minimize exposure to the BTC value changes, Volabit allows some services to create instant/green addresses. Any money sent to one of these addresses will be immediately converted to fiat currency.

### Request Payment from a User `api/v1/users/requests [POST]`

  You will create a payment slip for the user and will provide a payment Bitcoin address. Whenever the user pays the slip (convenience store, bank deposit, etc) the fiat will be automatically converted to Bitcoins and immediately sent to the address that was provided when making the API call.

### Send Money to User Implementation

  1. If you haven’t already, create a Volabit user for your user, ideally you will use the email address he used for signing up with your service. This action is done with a `POST api/v1/users` with the following parameters:
    - [email] _required_: email of your user
    - [password] _optional_: if left blank Volabit will generate a random password for the user.
    An email notification will be sent to the user.

  #### Bash Example

  ```bash
    curl -H "Authorization: Bearer YOUR_TOKEN" \
      -F user[email]=a_user@volabit.com \
      -X POST https://www.volabit.com/api/v1/users
  ```

  2. Get a quote in BTC for the amount of pesos to be sent to the user. `GET api/v1/spot-prices` allows for BTC/MXN and MXN/BTC conversions that include Volabit’s fees, hence the spot suffix.
    - [currency_from] _required_: Base currency you are converting from
    - [currency_to] _required_: Counter currency you are converting to.
    - [amount] _required_: Amount of the Base currency you want to convert to Counter currency.
  For sending 10,000 pesos to a Volabit user the params are: currency_from = MXN, currency_to = BTC, amount = 1000000
  
  #### Bash Example

  ```bash
    curl -F currency_from=BTC \
      -F currency_to=MXN \
      -F amount=100000000 \
      https://www.volabit.com/api/v1/spot-prices
  ```

  3. Generate a BTC address for the user. `POST api/v1/users/send`. Any bitcoins sent to this address will be immediately converted to fiat with 0 confirmations. Note that this method allows for selecting who will pay the fee. If Sender is selected, the Volabit user will receive the full amount entered. If Receiver is selected, the fees will be deducted from the amount sent. Params:
    - [email_to] _required_: Email of the Volabit user that will be credited the money.
    - [currency_from] _required_: Only BTC for the moment
    - [currency_to] _required_: Only MXN for the moment
    - [amount] _required_: Amount in fiat (MXN) in cents the Volabit user will receive.
    - [fee_payer] required: Will the fee be deducted from the Volabit user? or will the BTC amount returned include the fee?

  #### Bash Example

  ```bash
    curl -H "Authorization: Bearer YOUR_TOKEN" \
      -F email_to=a_user@volabit.com \
      -F currency_from=BTC \
      -F currency_to=MXN \
      -F amount=500000 \
      -F fee_payer=SENDER \
      https://www.volabit.com/api/v1/users/send
  ```

### Request Payment from User Implementation

  1. If you haven’t already, create a Volabit user for your user, ideally you will use the email address he used for signing up with your service. This action is done with a `POST api/v1/users` with the following parameters:
    - [email] _required_: email of your user
    - [password] _optional_: if left blank Volabit will generate a random password for the user.
    An email notification will be sent to the user.

  #### Bash Example

  ```bash
    curl -H "Authorization: Bearer YOUR_TOKEN" \
      -F user[email]=a_user@volabit.com \
      -X POST https://www.volabit.com/api/v1/users
  ```


  2. Create a payment slip for your user so you can collect a payment from him/her. `POST api/v1/users/requests`
    - [email] _required_: Email of the Volabit user a payment is being requested from
    - [currency] _required_: The payment slip’s currency (only MXN for now)
    - [amount] _required_: The amount for which the payment slip will be created (in cents).
    - [type] _required_: For what convenience store will the slip be created for, you can get all accepted method through `GET api/v1/users/requests/methods`.
    - [address] _required_: Bitcoin address where you want Volabit to send you the payment. Bitcoins will be sent the moment the payment is confirmed in Volabit’s platform.

  ```bash
    curl -H "Authorization: Bearer YOUR_TOKEN" \
      -F email=a_user@volabit.com \
      -F currency=MXN \
      -F amount=500000 \
      -F type=oxxo_mx \
      -F address=q15YTN2uxdJ7GMkvQ2prVJVfHcFw3t5YEwjq \
      https://www.volabit.com/api/v1/users/requests
  ```

If you have any questions please contact genaro@volabit.com
