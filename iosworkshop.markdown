---
layout: page
title: "locoquizz"
date: 2014-05-10 14:23
comments: true
sharing: true
footer: true
---
{% img right http://i.imgur.com/46fEYE1.png 150 250 locoquizz_minisplash %}

### Welcome

In this tutorial, we will write an iOS app that playfully tests a user's basic
knowledge of nearby landmarks. The code for the completed tutorial is on
Github:

* [Click here to download the full Xcode project of the completed app](https://github.com/arielelkin/LocoQuizz/archive/master.zip), or
* [Read the code on Github](https://github.com/arielelkin/LocoQuizz)
* [Download the session's slides here.](http://www.arivibes.com/downloads/Intro_to_iOS.pdf)


### Initial Setup

  1. Open Xcode, create a new project using the **Single View Application** template. Click **Next**. 
  1. Pick a name for the app ("LocoQuizz" is good). Leave all the other fields' defaults. Click **Next**. 
  1. Select "iPhone" as the Devices.
  1. Save the project somewhere dear to you. 
  1. We'll give our app an icon and a launch screen. 
    * [Help yourself to some assets here!](http://arivibes.com/downloads/Loco_Quizz_Assets.zip)
  1. By default, the Simulator doesn't support WiFi Positioning, so we have to enable Location Simulation. Click on **Product** --> **Scheme** --> **Edit Scheme**. Open the **Options** tab. Make sure that "Allow Location Simulation" is ticked and that you have selected a Default Location. 
{% img http://i.imgur.com/sfmNsqY.png 400 %}

### The Welcome View

  1. Open **Main.storyboard**. This view controller will display our welcome view. Add text welcoming the user, and brief instructions. Add a button to start the game.
{% img http://i.imgur.com/eLvhRv5.png 400 %}
  1. This view controller will also fetch the user's location so that we have it from the outset. 
  1. However, the game may start only once our location has been found! So the button must be disabled at the outset. 
  1. Add code to **ViewController.m** to
    * Conform to the CLLocationManagerDelegate protocol.
    * Initialise our instance of **CLLocationManager**
    * Disable the Start Game button

### The Game View

  1. Go back to the Storyboard and add a View Controller to the right of the screen. Add a label for our quesion, three round rect buttons to display the answers, and a second label at the bottom to give the result to the user. Set number of lines in the labels to **0** to avoid truncation. ![5](http://arivibes.com/wp-content/uploads/2013/01/5.png)
  1. Cmd+N to create a new file, select "Objective-C class" as the template. Name it **GameViewController**, a subclass of UIViewController, and save it. 
  1. In the Storyboard, specify the View Controller's class to **GameViewController**! 
  1. In the Storyboard, create a modal segue from the welcome view (ViewController) to the game view (GameViewController). 
  1. Pass the **CLLocationCoordinate2D** we have fetched in **ViewController** from **ViewController** to **GameViewController** using **prepareForSegue**. Print them in **viewDidLoad** to verify they've been correctly fetched and transmitted from one view controller to the other. 

### Fetching online data

Here's what the syntax of our query URL will look like: 

 > <pre>https://api.foursquare.com/v2/venues/search?ll=myCoordinates&client_id=MYCLIENTID&client_secret=MYCLIENTSECRET&v=DATE</pre>

 * [FOURSQUARE API RESPONSE (JSON).](https://api.foursquare.com/v2/venues/search?ll=51.509980,-0.133700&client_id=CPG1OA2FD0OE43PLES4MFOK133GSJADXF3DUMLO4CG0TKOEV&client_secret=30UCWAOGHIVF1C3MSUQ1RNPEUBKO20S01DQCWUQ0LEKCNE4D&v=20130126)
  * JSON not displaying nicely in your browser? Here's a JSON Viewer for [Chrome](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc) and one for [Safari](https://github.com/rfletcher/safari-json-formatter/downloads).

### Making the query and processing the response

  1. Scroll up **GameViewController** . Add the following instance variables: 
```objective-c
NSMutableDictionary *venuesDict; 
NSString *currentVenue; 
int correctAnswersCount; 
```
  
  1. Create a `fetchGameContent` method in **GameViewController** under **viewDidLoad**. This will build the query URL string and retrieve the JSON from FourSquare via a HTTP request. 
    * [Use the code from **fetchGameContent** here.](https://gist.github.com/arielelkin/5718605)
  1. Turn the JSON data into objects, and place them into **venuesDict**. 
    * [Use the code from **createVenuesDict** here.](https://gist.github.com/arielelkin/5718605).

### Displaying the question

  1. Scroll up **GameViewController's** . Specify **IBOutlets** for our three **UIButtons**, the question's **UILabel**, the result's **UILabel**. 
```objective-c
	IBOutlet UILabel *questionLabel;
	IBOutlet UIButton *buttonOne;
	IBOutlet UIButton *buttonTwo;
	IBOutlet UIButton *buttonThree;
	IBOutlet UILabel *resultLabel;
```
  1. Create a `nextQuestion` method which fills these up! It should get all the venues' names, pick one to quiz the user about, figure outs its category (the right answer), and get two other random categories to be displayed as the wrong answers. 
  1. Open the Storyboard and ctrl+drag from File Owner to the buttons and the label. 
  1. Test the app! 

### Checking the user's input

  1. Create a `userTappedAnswer:` method which checks the user's answer against our **venuesDict**. We should also update the text in our **resultLabel** to keep the user in the loop! 
  1. Open the storyboard and in **GameViewController's** ctrl+drag from the buttons to the File Owner. Let go and select `userTappedAnswer:` 
{% img http://i.imgur.com/iRsiu2l.png %}
  1. Now test the app! 

### Improving the app

  1. There's several cool and important things we now could add to our app (if time allows!) Below are some suggestions, let's take a vote... 
    * [Fetch and display relevant Tweets to provide user with hints.](http://search.twitter.com/search.json?q=piccadilly%20circus&amp;geocode=51.522199,-0.109762,25mi&amp;show_user=true&amp;lang=en)
    * Add animations.
    * Randomise the order in which the buttons are displayed.
    * Write a **GameContentFetcher** class to handle the content fetching.
    * [Fetch and display relevant images (Reddit + Imgur) to provide user with hints.](http://www.reddit.com/search.json?q=site:imgur.com+piccadilly%20circus)
    * When app is resumed after going into the background, check if location is different to current.
    * Play sounds.
    * Add a **MKMapView** to the initial View Controller.
    * Display **UIAlertView** if Core Location fails.


