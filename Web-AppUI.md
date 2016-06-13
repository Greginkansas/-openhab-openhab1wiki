Technical description for the Classic UI

## Introduction

The standard user interface for openHAB is based on the [WebApp.Net](http://webapp-net.com/) framework and can be accessed through any (webkit-based) web browser. 

You can directly [try it out on our demo server](http://demo.openhab.org:8080/classicui/app)!

## Details on WebApp.Net

WebApp.Net consists mainly of Javascript and CSS files. Thus it runs on top of the embedded Jetty HTTP server of the openHAB runtime. Simple HTTP access to the openHAB server is therefore enough to use this user interface (e.g. also remotely through a dynamic DNS service).

Although WebApp.Net is a pure HTML/JS solution, it mimicks an iPhone app and is optimized for touch operation. It not only works on iPhone/iPod touch, but also perfectly on Android. Even Symbian and Blackberrys are supported, and of course the !WebKit-based web browsers. So where ever you are and whatever device you have available, you should be able to access the UI to operate your home. 

See here some examples of its look:

![classic ui](http://www.openhab.org/assets/images/ui/classicui.png) 

## Technical Implementation

The UI can be found in the bundle org.openhab.ui.webapp. It registers a servlet with Jetty (usually openhab.app) and processes incoming requests. The UI makes use of the Sitemap definition file in order to render the pages.

Each widget of the Sitemap definition is rendered as a line in a list. Hence there are currently not many possibilities to construct custom HTML blocks, but it also means that the user does not have to configure anything special as a Sitemap file suffices for  
the definition of the UI.

Labels and icons for the widgets are retrieved through the ItemUIProviders and thus can be shared among different UIs.