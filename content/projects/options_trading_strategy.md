---
title: "Options Trading Strategy"
description: "0DTE Iron Condor/Butterfly strategy using Schwab API and Python"
dateString: June 19 2024
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

[Will Continue]
