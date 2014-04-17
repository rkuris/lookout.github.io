---
layout: post
title: Private Parts - A completely serious not at all joking project to change the privacy policy
author: jgortarez
tags:
- privacy
- grunt
- node
- less
- jade
---

At Lookout, the idea that security and privacy are important is shared not just by us engineers, but permeates our entire company culture. This is one of the main reasons why our recently open-sourced project, [Private Parts](https://github.com/lookout/private-parts), enjoyed the medley of team members it did. Our legal team pushed the project forward and worked closely with designers, developers, marketers and project managers to get us to launch. At any one time there were many helping hands all over Private Parts. ....(heh)....

### Our position is simple: 
#### Privacy is important and it shouldn't be confusing

If you've ever took a look at a privacy policy, possibly out of some morbid curiosity or maybe because you lost a bet with someone in legal, you will have likely encountered a dense and intentionally perplexing document which has been designed to make you give up on your silly quest to comprehend what's going on with your data by the second sentence.

We don't get down like that.

We set out in our cross-functional team, not completely unlike another certain fellowship, to conquer the privacy policy and fling the jargon-filled behemoth into the fiery depths of Mordor (and not lose anyone to the Nazgul along the way).

### Proto-prototypes
From a development standpoint, our first attempt was a simple prototype of a new visual design based on the categories and concepts developed by the [NTIA](http://www.ntia.doc.gov/). Our design team performed user testing and came back to the team with results and learnings. 

<center><img
src="/images/post-images/private-parts/pp-screen.png"
width="90%"/></center>
<center><small>Screenshot of final design and layout</small></center>

Some of the things we learned resulted in a few design tweaks, but nothing yet on the development side. At this point, we hadn't yet begun thinking about the process others might take to adopt our proposed solution. During Hacksgiving, our annual hackathon just before Thanksgiving, our team completed an advanced prototype which had the beginnings of a build process. 

Our goal was to create a tool based on all of our research that other developers and legal teams could take advantage of to improve their users experience.

This version was a big improvement over our first prototype and received good praise internally, but it had some limitations. It was built on Backbone.js which, while great for lots of things, didn't serve us as well in this instance. The output was a lot more bloated than we wanted the final product to be. Our goal was to build something as lightweight as we could get away with and that as many people as possible could use.

In the spirit if dogfooding I did my best to act as a front-end developer and use this prototype to build our Privacy Policy from scratch. It was not as easy as we ultimately wanted it to be. I found that we hadn't done a great job of making customization options accessible and easy to change. In fact, it was a bit of a nightmare. All of the icons were image based and having everything written in vanilla css caused a lot of problems when attempting to do even trivial things like change a typeface or background color for certain elements. 

> "Oh, I'm sorry, did you want to add your brand color and your special brand font? That'll be 45 minutes of hunting through the css and making sure you're replacing the correct things."

This is not ideal. 

The other issue was [responsiveness](http://alistapart.com/article/responsive-web-design/). We're a mobile-focused company and it would be very silly of us to build something that only looks good on a desktop. Our hacked version was a decent looker on mobile, but didn't have the full responsive and reflexive behavior we required.

### Build Process
#### Moving configuration from the client to static site generator  
#### or
#### "Grunt all the things"

While our backbone version did have a build process, it was much more complicated and involved than we wanted. A few weeks later the team got back together and started from scratch with a build process utilizing [node](http://nodejs.org/) and [grunt](http://gruntjs.com/) for the heavy lifting. The goal of this iteration was to simplify the process and output, making it easier to get started and get done.

In dogfooding our previous version, I found that updating design and layout was a challenge, but integration proved to be an even more unwieldy process. I had to focus on how css was being applied and ensure proper namespacing of all css classes and id's. 

Our new build process moved complexity away from the client. There would be no more dynamic rendering of templates from a datasource, instead we would opt for a static output and as few files as possible.

We went with [Jade](http://jade-lang.com) as our templating language. I had never used Jade before, but found it great to work with and have since used it in other projects. Jade also allowed us to stay within our JavaScript-based universe of node/npm and grunt. 
 
Jade enables mixins and includes which was great in helping us maintain a modular structure.

Below is the jade mixin for each of the categories:

<pre>
<code>
mixin module(category, type)
  - if (category.options.length > 0) {
      div(class=['module', 'sharedCategory', category.class])
        if(type == 'share')
          span.dingbats-misc-corner-flag.mobile-share-icon
          span.dingbats-misc-rounded-check.mobile-share-icon
        div.module-icon
          span(class=['dingbats-'+category.class, 'pp-icon'])
        - if (type == 'share') {
          h3.module-title.dingbats-misc-rounded-check.pp-desktop-show= category.name
            span.dingbats-misc-play.pp-triangle
          h3.module-title.mobile-title.pp-mobile-show= category.name
          div.module-definition.pp-hide-this
            p= category.description
        - }
        - else {
          h3.module-title.pp-desktop-show= category.name
          h3.module-title.mobile-title.pp-mobile-show= category.name
        - }
        div.module-options-list
          ul
            - if (category.options.length > 0){
                each item in category.options
                  li!= item
            - }
            - else{
                p!= category.emptyText
            - }
  - }
  - else {
      div(class=['module', 'nonShared', category.class])
        if(type == 'share')
          span.dingbats-misc-corner-flag.mobile-share-icon.not-sharing
          span.dingbats-misc-rounded-x.mobile-share-icon
        div.module-icon
          span(class=['dingbats-'+category.class, 'pp-icon'])
        - if (type == 'share') {
        h3.module-title.dingbats-misc-rounded-x.pp-desktop-show= category.name
          span.dingbats-misc-play.pp-triangle
        h3.module-title.mobile-title.pp-mobile-show= category.name
        div.module-definition.pp-hide-this
          p= category.description
        - }
        div.module-options-list
          ul
            - if (category.options.length > 0){
                each item in category.options
                  li!= item
            - }
            - else{
                p!= category.emptyText
            - }
  	- }
</code>
</pre>

As you can see, there is quite a bit of logic embedded in just this mixin (we have a couple others which manage different views). 

Jade's functionality was essential in allowing us to easily cycle through our JSON config file and put the data where it needs to be and keep our layout nicely structured no matter how many or how few categories were being utilized. 

So that mixin, paired with the following bit of code, produces the complete “What do we collect?” section:
  	

<pre>
<code>
section#collected
 div.clearfix
  each option in pageOptions
   h2!= option.collectedSectionHeader
    div.module-row.clearfix
     - var count = 0;
      each category in collected
        - count++
        +module(category, 'collect')
        if count % 3 == 0
          div.module-row.clearfix
</code>
</pre>

The Jade template engine supports full JavaScript code by prefixing each JavaScript line with a hyphen " - ". Other built-in functions like “each...in” can be added without the prefix. In the above, both styles can be seen. The following: 

<pre>
<code>
each option in pageOptions
 h2!= option.collectedSectionHeader
</code>
</pre>

simply loops through the pageOptions hash in our config file and renders the contents in an h2 tag. The addition of "!=" just after the h2 tells Jade to NOT convert the contents of this value into plain text. This allows HTML formatting to be added directly into the data source. The benefit of which is clear when you want to easily add a link or a line-break to the rendered content without messing with the template. 

<pre>
<code>
- var count = 0;
 each category in collected
  - count++
   +module(category, 'collect')
   if count % 3 == 0
    div.module-row.clearfix
</code>
</pre>


This code again loops through our config file, specifically, collected and creates a new module with the category object with the type of 'collect' as arguments. " - count++ " allows us to increment the variable 'count' which we initialized two lines above. This enabled us to "dynamically" adjust and maintain the layout by adding a dividing module-row after every third module created, as a result keeping a three-module max per line layout regardless of how many items were added in the customization process.

# Exposing our private par...errr...customization options 

To deal with the issue of un-fun CSS, we again stayed within the JS family and went with [Less](http://lesscss.org/) as our CSS preprocessor.

With Less we were able to utilize variables which helped to overcome the issue I had ran into while trying to use our previous version.

In our assets directory we included custom.js, variables.less and custom.less. variables.less exposed the many standard configuration options we enabled for css including things like fonts, colors and a few element sizing options, among others.	

Our custom.less file:

<pre>
<code>
// Typography
@font-family: 'Source Sans Pro', helvetica, sans-serif;
@base-font-size: 100%;

// Colors
@background-color: #ffffff;
@dark-background-color: #f1f1f1;
@branding-color: #3db249;

@text-color: darken(@dark-background-color, 50%);
@link-color: @branding-color;
@dark-background-headline-color: @branding-color;
@dark-background-text-color: @background-color;

// General display
@button-corner-radius: 5px;
@module-height: 5.3em;
</code>
</pre>

This ease of configurability takes the required skill level of the prospective developer-user way down. And for the higher skilled developers, we've saved you some time! You're welcome!

Now, as a developer you can simply change the Less variable for @branding-color and having something that immediately feels more at home in your current website design. 

Less color functions like darken(color, value) and lighten(color, value) enabled this high level of customization where we could simply set box-shadows, icons and hover events as percentages of the base colors. 

Our design team produced icon fonts to replace all of the images used in the previous version. This greatly simplified the customization process by requiring a simple change to a css hex value instead of forcing people to crack open photoshop to make their own versions. In addition, by Base64 encoding our icon fonts into our CSS, we significantly reduce the number of files needed.

The output of this build process is two files, but you only need one. The difference being one that is a mostly standalone html page and the other one that is more easily dropped into an existing page or template.

# In summation...

We're very excited to have our private parts out in the open. 

Okay, okay… that's enough. 

Really, we're excited to be a part of the push for a better user experience and a more well-informed public. I imagine that if you're reading this, you probably care about these things too. We're looking forward to seeing what others do with this and would love to get feedback about your experiences with it. Email us at [privacy@lookout.com](mailto:privacy@lookout.com).

That being said, we're not done. We're beginning work on the next iteration which will make it even easier for anyone, developer or not, to create a privacy policy that will benefit their users.

Check out [Private Parts on github](https://github.com/lookout/private-parts)

– [Jesse Gortarez](mailto:jesse@lookout.com)



