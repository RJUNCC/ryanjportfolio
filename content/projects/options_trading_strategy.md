---
title: "Options Trading Strategy"
description: "0DTE Iron Condor/Butterfly strategy using Schwab API and Python"
dateString: 2024-06-19
draft: false
tags:
  [
    "Python",
    "Sentiment",
    "News",
    "Trading",
    "Options",
    "Greeks",
    "Schwab",
    "Decision Tree",
  ]
showToc: false
weight: 400
height: 500
cover:
  image: "/projects/options/options.png"
---

## Description

Using Charles Schwab API to automate 0 Days-to-expiration iron condor and iron butterfly strategy based on max loss, premium-credit ratio, implied volatility, delta, gamma, vega, theta, news articles (using my News App), and historical candlestick patterns (using deep learning).

## Learning OAuth and Using AWS Secrets Manager

For this project, I had to learn bearer token and OAuth 2 framework to connect to my account. Bearer token is used as OAuth authorization code. I used AWS Secrets Manager to store my keys and secrets. Then, used boto3 python library to get the keys and applies them using an f-string to the authorization URL that Schwab provided in their documentation. Once I have my authorization URL, I have to open it in a web browser and sign in with my real Schwab account, not the developer account. After that, I copy the returned URL and construct the headers and payload, where headers have authorization and content type scopes, and payload has grant type, response code, and redirect URI.

The authorization header uses the basic autorization scheme. I also had to make a credentials and base64_credentials variable, where credentials is `credentials = application_key:application_secret`. Then, the base64 credentials, I had to use base64 python library and do `base64_credentials = base64.b64encode(credentials.encode("utf-8")).decode("utf-8")`. Finally, to complete the authorization header with the basic authorization scheme, the code looks like this `"Authorization" : f"Basic {base64_credentials}"` and the content type header looks like this `"Content_Type" : "application/x-www-form-urlencoded"`. This content type gives us the ability to request an access token, which for Schwab, we lose access to the token every 30 minutes, which is why a refresh token is useful, so we don't have to keep signing in through a webbrowser each time we run a python script. Now for the payload, for an access token request, we need to set our grant type to `authorization_code`. Our response code scope should be set to the code that is 5 indices after "code=" in our return URL that we copied. And the "redirect_uri" scope, I set to localhost, but you can also use ngrok if you want to be a little more fancy.

Next, I have to retrieve the tokens using a POST method where the url parameter is `"https://api.schwabapi.com/v1/oauth/token"`, headers parameter is the headers from that we set in dictionary form, and the data parameter is the payload that we set also in dictionary form. Once we get our response, we can convert it to json/dictionary format and actually look at our tokens.

Now, we can make a function to refresh the tokens. With AWS Secrets, I can set the refresh token value and also get it. So, when I get the token, I set it to a variable basically construct the headers and payloads again. The difference this time is that the payload has two scopes this time, grant type and refresh token, where grant type is set to `"refresh_token"`, and refresh token scope is set to the refresh token value that we got from AWS Secrets Manager. The headers are the same as before using basic authorization scheme. Then, we do a POST request method with the same url, headers, and the new payload. Then, once we get our reponse 200 code status, we can change the response to dictionary format, which will return a dictionary, and we can get our refresh token, access token, and id token from there.

## Data Cleaning and Subsetting

For the data cleaning, the data comes from Schwab API in a nested dictionary. First, with datetime library I initiate my current date and expiration date (which should be the same data as current date). Then, I specify the parameters taht I want with the documentations help. I also initiate the headers

```
headers = {
  'accept': 'application/json',
  'Authorization': f'Bearer {access_token}',
}
```

Then I get the data by inputting `"https://api.schwabapi.com/marketdata/v1/chains"`, the paramaters, and the headers, and convert it to json/dictionary. After that, I split the calls and puts, and initiate the list of features that I want that you would see in an options chain map on Thinkorswim or Schwab. I also added suffixes to the columns because I'm going to merge the calls and puts dataframes. Next, I set a constant value for the short call and put deltas, which would both be the inner legs of this iron condor strategy, and I would get the the index with the closest delta. Since we are trading the $SPX, the strike prices are in increments of 5. So if we want a 10 point width, then we can go up 2 indices for outer long call leg, and down 2 indices for outer long put leg.

Schwab doesn't really have a paper trading account, but they said later this year that they will include sandbox. There is also a preview feature but the documentation using POST request is really lacking.

If you want to test your strategy manually, you can use Thinkorswim since Schwab acquired TD Ameritrade. The API documentation for Thinkorswim was also removed and there is no news about it. If they are going to recreate the Thinkorswim API, then I hope the data is easier to subset.

## Slack Confirmation Notification

I wanted to send a message via a Slack bot to a certain channel on my Slack server to confirm if I want the order to get through. I also want it to check the order and check the important features that I have in the picture above every 30 minutes or every hour, I still haven't decided yet. I'm still testing on Thinkorswim on what to watch out for, like 52 week high, certain implied volatility, maybe the RSI indicator, and most definitely trend the past 7 days more or less.

The whole point of the iron condor is for the trend to be sideways, and the way we make max profit is if the price at the end of the day ends in between the two inner short legs.

## Calculating Maximum Loss

The maximum allowed loss that I wanted is <10% of portfolio. So, that would be total portfolio value times 0.1. Then, I calculated the net premium received which would be short call premium + short put premium - long call premium - long put premium. And I would multiply that by 100 because I'm doing options, not stock trading. Then, the max loss on the call spread would be the long call strike - short call strike. Then, again we multiply that by 100 and subtract it by the net premium received. Then, the max loss for the put spread is short put strike - long put strike. Then, multiply that by 100 and subtract by net premium received. Finally, the max loss would be the great loss between the max loss of call spread and max loss of put spread.

Next, we do a trade condition and if the max loss is greater than or equal 10% of our portfolio, and also if our premium-credit ratio (net premium received / max loss) is between or equal to 0.15 and 0.25, then we return True, and we can send a slack notification to confirm the trade through user interactivity while using ngrok as a redirect URI. If those conditions do not meet, than we just cancel the trade and wait 30 minutes for the next slack notification about the information of the trade. And we should add another condition in there, if hour initial trade didn't go through and its after 11:00 am, it might be too late to make the trade.

I've noticed when making an iron condor trade, that making a trade when the market opens is not really a good idea. Waiting an hour has been more reasonable when paper trading.

[To be continued]
