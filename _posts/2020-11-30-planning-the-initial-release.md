---
layout: post
title: "Triathlon App: Planning the Initial Release"
comments: true
excerpt_separator: <!--more-->
---

<figure>
  <img src="https://source.unsplash.com/eC0idQ0eek4/1920x1280" alt="Brainstorming">
  <figcaption>
    <span class="photo-credit">Photo by <a href="https://unsplash.com/@magnetme?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Magnet.me</a> on <a href="https://unsplash.com/s/photos/brainstorm?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
  </figcaption>
</figure>

I haven't written an iOS app since my senior year of college. And although I'm typing this wearing the sweatshirt of my alma-mater, the hairstyle of a knockoff Tim Riggins, and a backwards hat with the words "High St. Deli" written on it, it's been half a decade since I graduated. While it's clear not everything has changed, I'd like to think that I've managed to pick up a thing or two about writing software over the years and hopefully put them into practice when writing my [triathlon app]({% post_url 2020-11-20-triathlon-app %}).

<!--more-->

The first app I ever made was called "Cork-It". In hindsight, this seems like a clever way to tell someone to "shut up" whilst sipping on a bottle of Two-Buck-Chuck. Instead, it was an app to save text messages so you could easily read them again later. In essence, it's an app that saves data to your phone, which isn't very complicated, but it ended up taking forever because I found myself wanting to do everything perfect. Instead of typing words into my code editor, I entered them into Google hoping to find a simple answer but only leading myself down another rabbit hole.

Instead of learning my lesson, I fell into the same trap with my second app...and my third. Rather than focusing on the end goal of releasing an app, I wasted time trying to perfect some minute detail: like what is the best way to store persistent data or which shade of white to use in my app. (In the end I chose #DADADA. Looking back, I don't know what to be more worried about, mistaking what is clearly Gainsboro for "egg shell" or that I've discovered what looks to be a cry for help encrypted in hexadecimal.) At the end of the day, writing software is about offering a solution, but it wasn't until I got a bit of real world experience that I learned this.

I've spent the majority of my career working at Apple, building, testing, and mostly breaking iOS' more seasoned yet often unappreciated brother -- macOS. I'm fortunate enough to have been able to get a peek inside and see how the sausage is made. As an AppKit Engineer, I experienced the lifecycle of new features from inception to fruition. As a compatibility engineer I had the unique opportunity to see how each team worked together to ship software to millions of people around the world. And not one year did we, a company with nearly unlimited resources, release software without missing features and known bugs. What I began to understand is that it's not about being perfect, it's about improving the life of your users one release at a time.

Okay so back to business. I'm not a trillion dollar company, but I'm not a green college grad either. Which basically puts me in the ballpark of, I sort of know what to do, and I sort of know what I'm doing and I'm pretty sure I know what not to do. So on that vote of confidence, here's what I've got so far. My goal is to find a version of my app with smallest feature set that is still worth using. Silicon Valley startups call this a "Minimum Viable Product" -- but I'm not trying to start a business in my parent's garage so I don't think I have enough street cred to use that term.

For my triathlon app, I started by brainstorming a list of all the features I would want in an app: a calendar for planning my race season, guided training sessions, exporting data to 3rd party apps, phone and watch widgets...the list goes on. This is clearly too much for me to handle at once, but with all this written on paper, I noticed that many features depended on each other. For example, before the app can guide you through a workout, it needs to know how to gather health metrics like heart rate, pace, or duration. From here, I decided to organize features into phases.

Each phase is ordered such that subsequent phases cannot not be implemented until the previous phase is complete. In doing so, my first release's feature set boils down to a simple workout tracker. I know, there's millions of these apps available, but for my app to be successful I have to start somewhere. This will also allow me to get it into the hands (and wrists?) of people to give it try and tell me what they think -- but let's be honest besides myself, my only other users will be those I'm related to -- thanks mom. But what is family for if not for free beta-testing lying about how much they like your app.

Thanks for reading 🕺🏼

Duff