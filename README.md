twitter_latest_twwet
====================

#Application provides latest tweet of the user by using oauth2 authentication machenism.
#Here is the flow of process of oauth 2 authentication
views.py


from django.shortcuts import render_to_response

from django.contrib.auth.models import User

import sys

from django.template import RequestContext

import twitter

import oauth2 as oauth

import urlparse

import cgi

import tweepy

from django.http import HttpResponseRedirect

def home(request):

    context=RequestContext(request,{})
    
    return render_to_response('index.html',context)

def value(request):

    print 'as'
    request_token_url = 'http://twitter.com/oauth/request_token'
    authorize_url = 'http://twitter.com/oauth/authorize'
    consumer_key = 'enter consumer key here'
    consumer_secret = 'enter consumer secret here'
    consumer = oauth.Consumer(consumer_key, consumer_secret)
    client = oauth.Client(consumer)
    
    resp, content = client.request(request_token_url, "GET")
    if resp['status'] != '200':
        raise Exception("Invalid response from Twitter.")

    # Step 2. Store the request token in a session for later use.
    request.session['request_token'] = dict(cgi.parse_qsl(content))

    # Step 3. Redirect the user to the authentication URL.
    url = "%s?oauth_token=%s" % (authorize_url,
        request.session['request_token']['oauth_token'])

    return HttpResponseRedirect(url)

def twitter_authenticated(request):

    access_token_url = 'http://twitter.com/oauth/access_token'
    
    consumer_key = 'add your consumer key'
    consumer_secret = 'add consumer secret'
    consumer = oauth.Consumer(consumer_key, consumer_secret)
    token = oauth.Token(request.session['request_token']['oauth_token'],
    request.session['request_token']['oauth_token_secret'])
    client = oauth.Client(consumer, token)

    # Step 2. Request the authorized access token from Twitter.
    resp, content = client.request(access_token_url, "GET")
    if resp['status'] != '200':
        print content
        raise Exception("Invalid response from Twitter.")

    """
    This is what you'll get back from Twitter. Note that it includes the
    user's user_id and screen_name.
    {
        'oauth_token_secret': 'IcJXPiJh8be3BjDWW50uCY31chyhsMHEhqJVsphC3M',
        'user_id': '120889797', 
        'oauth_token': '120889797-H5zNnM3qE0iFoTTpNEHIz3noL9FKzXiOxwtnyVOD',
        'screen_name': 'heyismysiteup'
    }
    """
    access_token = dict(cgi.parse_qsl(content))
    auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
    auth.set_access_token(access_token['oauth_token'],access_token['oauth_token_secret'])
    api = tweepy.API(auth)
    tweet=api.user_timeline(count=1)
    for i in tweet:
       print i.text #this is the latest tweet by user
    return render_to_response('result.html',{'latest_tweet':tweet})


urls.py

urlpatterns = patterns('',
    # Examples:
    
    url(r'^$', 'gettweet.views.home', name='home'),
    
    url(r'^final$','gettweet.views.value',name="value"),
    
    url(r'^twitter-authenticated$','gettweet.views.twitter_authenticated',name="auth"),
    
)

index.html

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN"
"http://www.w3.org/TR/html4/strict.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" lang="en">
  <head>
		<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
		<title>index</title>
		<meta name="author" content="shobhit" />
		<link href="/static/cssall/bootstrap.css"  rel="stylesheet" type="text/css"  />
		<link href="/static/cssall/bootstrap.min.css" rel="stylesheet" type="text/css" />
		<!-- Date: 2012-08-30 -->
	</head>
	<body>
        <div class="span12">
		<div class="row">
		<div class="container">
			<button class="btn btn-succes" id="twitter">Login with Twitter</button>
		</div>
	</div>
	</div>
	</body>
	<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js"></script>
    <script src="/static/javascripts/bootstrap.min.js"></script> 
	<script type="text/javascript">
		$(function(){
			$("#twitter").click(function(){
				window.location="/final"
			});
		});
	</script>
</html>

result.html

<!DOCTYPE html>
<html lang="en">
  <head>
		<link href="/static/cssall/bootstrap.css"  rel="stylesheet" type="text/css"  />
		<link href="/static/cssall/bootstrap.min.css" rel="stylesheet" type="text/css" />
		<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js"></script>
	</head>
	<body>
	<div class="span12">
		<div class="container">
			<div class="well" align="center">
			  {%  for c in latest_tweet%}
			  <p> {{c.text}}</p>
			  {% endfor %}
			</div>
		</div>
	</div>
	</body>
</html>

