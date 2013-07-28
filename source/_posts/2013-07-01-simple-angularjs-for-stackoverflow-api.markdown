---
layout: post
title: "Simple AngularJS for Stackoverflow API"
date: 2013-07-01 00:30
comments: true
categories: [javascript, AngularJS, Stackoverflow API]
keywords: javascript, angularjs, stackoverflow api, html, bootstrap
description: How to build a simple Angularjs application for Satckoverflow api
---

##Angular JS

[AngularJS][1] is a javascript and client side Framework. It expose a toolset based on extending the HTML vocabulary for your application.

There's many other Frameworks like [Backbone.js][2], [Ember.js][3], [Knockout][4] ...

##Main concepts

- Model: Application data
- View: What users sees
- Controller: Application behavior
- Scope: Glue between application data and behavior
- ```$```: Angular namespace
- Module: configure the injector
- Injector: assembles the application

<!-- more -->

AngularJS offers two-way data binding without funcing up the model. You don't need to write a model like other javascript librairies.

{% img center http://docs.angularjs.org/img/One_Way_Data_Binding.png %} 
{% img center http://i.stack.imgur.com/KzDS3.png %} 

Data-binding is an automatic way of updating the view whenever the model changes, as well as updating the model whenever the view changes. This is awesome because it eliminates DOM manipulation from the list of things you have to worry about. Source [AngularJS doc][5]

Controllers are the behavior behind the DOM elements. AngularJS lets you express the behavior in a clean readable form without the usual boilerplate of updating the DOM, registering callbacks or watching model changes. 

A model for AngularJS is just a simple Javascript Object (JSON).
This means, if we load a user from an API call, we can use it in html like he's structured from server. Magic!

##How it works

- Browser loads the HTML and then the AngularJS script.
- AngularJS wait for ```DOMContentLoaded``` event and looks for the ```ng-app``` directive.
- The module specified in ```ng-app``` is used to configure the injector.
- The injector is used to create the ```$compile``` service and ```$rootScope```.
- ```$compile``` compiles the DOM and links it with ```$rootScope```.

{% img center http://docs.angularjs.org/img/guide/concepts-startup.png %} 


##Our first application with AngularJS

The goal of this section's to use AngularJS in a simple app, we are going to use [Stack Exchange API V2.0 ][6] just for demonstration purpose. I choose [Stackoverflow][7] simply because I like it and it's helpful for all of us.

##Module

{% codeblock controller.js lang:javascript %}
	var soApp = angular.module('soApp', []);
{% endcodeblock %}

To create an AngularJS module, we just provide our module a name, in our case it's ```soApp```.
We use here an empty array, this array provides modules dependencies, all modules uses by our modules. We don't need this time.

##Controller
{% codeblock controller.js lang:javascript %}
soApp.controller('MainController',
    function MainController($scope){
    	//Controller behavior here
	});
{% endcodeblock %}

To be continued

[1]: http://www.angularjs.org
[2]: http://backbonejs.org
[3]: http://emberjs.com
[4]: http://knockoutjs.com
[5]: http://docs.angularjs.org/api
[6]: http://api.stackexchange.com
[7]: http://www.stackoverflow.com

