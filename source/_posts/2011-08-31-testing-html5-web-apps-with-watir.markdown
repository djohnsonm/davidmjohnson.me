---
layout: post
title: "Testing HTML5 Web Apps with WATIR"
date: 2011-08-31 13:59
comments: true
categories: [ruby,html5,WATIR,testing] 
---
{% img https://dl.dropbox.com/u/10021156/blog/InteractiveRuby.png %}

##What is WATIR?
Watir means “Web Application Testing in Ruby.” It’s an easy way to automate the testing of your Web Apps through a Ruby interactive console. Watir will click buttons, fill in forms, navigate to pages, you name it. It’s also compatible with all major browsers.

##How do I get started?
So let’s say hypothetically you have an HTML5 web app using jQuery Mobile running on top of ASP.NET MVC 3. A quick and easy way to test the UI functionality of a new user registration would be to use Watir.

##The Test
{% youtube 640 320 sKsQFLB6ZG8 %}

{% codeblock lang:ruby The Code %}
#INTERACTIVE TESTING WITH RUBY GEMS
#LOAD UP RUBY GEMS, Watir and Watir-webdriver
require 'rubygems'
#this is for IE
require 'watir'
#this is for Chrome
#require 'watir-webdriver'
#load Browser
b = Watir::Browser.new
 
#go to URL
b.goto 'http://watirtesting.apphb.com/'
 
#Select the Register Link
l = b.link(:name,"Register")
l.exists?
l.click
 
##Populate Register Fields
b.text_field(:name, "UserName").set("Fake Name")
b.text_field(:name, "Email").set("SampleUser@SampleUser.org")
b.text_field(:name, "Password").set("Password12345")
b.text_field(:name, "ConfirmPassword").set("Password12345")
 
##Create New User
b.button(:value => 'Register').click

{% endcodeblock %}

##Who uses WATIR?
Facebook
SAP
Oracle
Yahoo
Expedia

##Get WATIR
Watir can be downloaded. You’ll need to install ruby as well. All watir commands can be run from the Ruby Interactive Console.
