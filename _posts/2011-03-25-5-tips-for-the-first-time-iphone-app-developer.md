---
id: 118
title: 5 Tips for the first-time iPhone App developer
date: 2011-03-25T09:59:26+00:00
author: mharsch
layout: post
guid: http://blog.harschsystems.com/?p=118
permalink: /2011/03/25/5-tips-for-the-first-time-iphone-app-developer/
categories:
  - Uncategorized
tags:
  - iOS
---
Having just completed the process of developing [my first iPhone App](http://itunes.apple.com/us/app/shot-o-matic/id426601901), I thought I&#8217;d share some tips that I picked up along the way while they&#8217;re still fresh in my mind (i.e. still licking my wounds).

**1.) Learn Objective-C and the iOS SDK in a structured way**
  
Stanford has done a great job packaging [their iOS programming course](http://itunes.apple.com/itunes-u/developing-apps-for-ios-hd/id395605774) for mass consumption via iTunesU. The material is presented in a logical progression that does not assume any previous experience with Objective-C or any Apple development environment (though you do need a strong handle on object-oriented design concepts). Lectures are interleaved with demos that introduce new concepts by-example. I&#8217;ve found this lecture/demo format very effective for quickly ramping-up on the platform. I did buy a book to supplement the iTunesU course, but I never used it. Apple&#8217;s online documentation will become your new best friend, and between the class and [developer.apple.com](http://developer.apple.com), no additional documentation is needed (ok, maybe [stackoverflow.com](http://stackoverflow.com)).

**2.) Design your model, prototype, then iterate on details**
  
Everything is keyed off your model, so design it first before diving in to View Controllers. You can iterate the model design of course, but big decisions should be made up-front (like whether or not to use Core Data). I&#8217;d suggest saving Core Data for your second project, as it adds a fair amount of complexity. 

I found that it was easy to bog down on trying to get each little piece to display or function as you want it to. In order to keep your momentum, I&#8217;d suggest prototyping all your features first, without worrying about look-and-feel. The SDK provides tons of functionality with very little code when you opt for their defaults. As soon as you start saying things like: &#8220;I wish that button were a different color&#8221; or &#8220;could it slide in from the other side?&#8221; &#8212; you will be forced to start descending levels of the stack and write custom code that overrides the SDK convenience features. This may be necessary to achieve your objective, but this kind of customization should be saved for the last stage (iteration).

**3.) Keep a constant eye on performance**
  
I took a lot for granted while developing my first App. One mistake that came back to bite me was relying too heavily on the iOS Simulator (part of the development environment) to gauge the app&#8217;s performance. I reached the end of my development effort and was surprised to discover that the app had horrible performance on real-world hardware (specifically iPhone 3). My first app used a large custom scroll view as the central feature (allowing the user to horizontally scroll through a list of pages, like the &#8216;Weather&#8217; app that comes with the phone). Scrolling through the pages is an animation effect that the phone must generate when you put your finger down and start dragging it. The amount of work that the graphics handling layers must do is a function of several factors including how many subviews must be rendered to produce the next frame in the animation. My initial design had 6-8 subviews (a combination of labels, images, buttons, background, etc.). This proved to be way too many for the phone to handle, and thus the framerate during animation was very poor &#8211; producing a choppy effect. I was able to work out the biggest factor (too many subviews) and fix it by merging the subviews that didn&#8217;t change into one layer (the background image). Apple&#8217;s debugging tools helped with this diagnosis (specifically the &#8216;Core Animation&#8217; [Instrument](http://developer.apple.com/library/mac/#documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/Built-InInstruments/Built-InInstruments.html)), but I should have caught the problem much earlier in the development cycle. I now keep my old iPhone 3 on hand to use as a &#8216;lowest common denominator&#8217; spot check whenever I change anything that could have a performance impact.

**4.) Learn and master Apple&#8217;s provisioning scheme early on**
  
The hoops that one must jump through to get code running on a device are many and varied. Joining the Apple Developer program ($99/yr.) is necessary to be able to push your app to a device. Once you&#8217;re signed up for the developer program, you have access to Apple&#8217;s Provisioning Portal where you generate certificates, associate phone IDs with certain profiles, etc. It&#8217;s all very complicated and not very well documented. It&#8217;s worth your time to dig into this process and really learn how certificates, profiles, App IDs, and Device IDs combine and interact to allow your app to be deployed to other phones (either through Ad-Hoc distribution, or via the iTunes App Store). A solid understanding of this mechanism will be very nice to have when it comes time to submit your App to the iTunes App Store for review (you don&#8217;t want to be scrambling at that point).

I had a small team (3 other team members) that were providing feedback on builds as the app was taking shape. Since I was the only developer, I couldn&#8217;t rely on others having the XCode development environment. To facilitate distribution of builds, it was necessary to create an &#8216;Ad-Hoc Provisioning Profile&#8217; and associate that profile with builds. Once the profile was setup and working, I could generate .ipa files from XCode and my teammates could just drag them into iTunes and update their phones from there. This delivery system worked very well, and as I mentioned &#8211; it forced me to learn about the provisioning scheme early on which paid dividends later.

**5.) Consider adding Analytics**
  
It&#8217;s astoundingly easy to add analytics functionality by dropping in the [Flurry SDK package](http://www.flurry.com/product/analytics/index.html){.broken_link}. Doing so will allow you to track user sessions (launches of your app) and events that occur during app usage (clicking buttons, searching, whatever). There are obvious tradeoffs to consider here: The Flurry package is not open source, so you can&#8217;t be sure exactly what they&#8217;re doing with your data. Also, linking to their library introduces a risk factor since Apple can change their policies regarding privacy which could force you to update your app with a new Flurry SDK version (this has happened to Flurry users at least once already). With all of that said, the appeal of having analytics (for basically no investment on your part) is undeniable.

If you do decide to include Flurry Analytics in your app, I&#8217;d suggest that you utilize Blocks and [GCD](http://developer.apple.com/library/mac/#documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html%23//apple_ref/doc/uid/TP40008091-CH102-SW1) to dispatch calls to FlurryAPI on low priority threads (separate from the main UI thread). Since users don&#8217;t see any output from these network connections, there&#8217;s no reason to make them suffer their latency.