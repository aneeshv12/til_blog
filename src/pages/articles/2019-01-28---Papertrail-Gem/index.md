---
title: The Papertrail Gem
date: "2019-01-28T00:00:00.000Z"
layout: post
draft: false
path: "/posts/papertrail-gem/"
category: "Rails"
tags:
  - "Rails"
  - "Web Development"
description: "What's the 📄 Papertrail Gem 💎 and how can it help me?"
---

Working in Support for a software company means receiving requests to uncover "who changed \<insert piece of data here\>?"

When I worked in Application Support for a Healthcare EMR company from 2012-2014, it was crucial for my clients to figure out who modified confidential patient information, especially in large healthcare entities where hundreds of users had access to patient records.  It was frustrating for someone in billing to update patient information and then the next day, they see that someone else had changed it back.  It could mean missing out on substantial amounts of insurance claim remittance money for the organization. 💸

When I would troubleshoot these types of issues, I was limited in the data I had access to.  Oftentimes our SQL tables had few fields that pointed me to the right answer _if I was lucky_.  I was able to see:
- `created_by` (a user_id)
- `created_at` (a timestamp)
- `updated_by` (a user_id)
- `updated_at` (a timestamp)

Everytime a user would make an update to the information in the database, it would update the `updated_by` field with their user ID and the `updated_at` field with the time when it occurred.

If multiple people had touched that piece of data, I would not have much history of all of those changes.  What I could see was the ID of the last user, but it was usually the ID of the user who corrected the bad data.  It was certainly not a helpful data model!

On my first day at Pingboard in Sept. 2018, I looked through the internal Wiki pages and wrote down `Look into Papertrail Gem` as a to-do item.

Our app is a Ruby on Rails app, which means we can use gems (a type of plugin/package for Rails apps).  This specific gem enables us to track the history of chosen data models within our app.

Like most apps (including the healthcare software I previously worked with), we have a `User` model.  The Ruby model file, including the line `has_paper_trail`, indicates that the app should track the different versions of that model in our database.  Papertrail uses a table called `Versions` to store all of this data.  The relationship between the model and the versions table is a one-to-many relationship since `One` User row has `Many` previous version rows containing snapshot information of the model.

Having a versions table means having the ability to view WHO made all the changes to a piece of data and WHEN those changes were made.  This is especially valuable in apps where data is owned by multiple people, or there are multiple admins who have access to modify data.  The table sure makes my life a lot easier as a Support Engineer, because answering "who changed \<insert piece of data here\>?" is as simple as finding the ID of the model and then running `user.versions` in a Rails console.

Papertrail is able to identify who changed the data by attaching itself to the `current_user` method often defined in a Rails application controller.  As long as data is changed through the app, there is a record of "Whodunnit" (this is the term Papertrail uses in its data model!).  However, if data is manipulated through a Rails console or external API calls, there may not be a current_user set.  It is important to keep this in mind when allowing data manipulation through new channels so that the user/system changing data is captured.

Keeping track of different versions allows a person to revert back to a previous version if necessary. This idea is very similar to the one used in the version control system many developers are familiar with, Git.  Papertrail has a method called `reify` that can be called upon an instance of a version to revive that model back to the state in the specified version. This is super handy when a lot of unwanted changes happen in an app!

It is important to realize that in an app where data is changed often, this versions table could become extremely bloated and expensive.  Papertrail gives you the ability to configure a limit of versions saved.  At Pingboard, we keep a record of 40, [which is documented in our security information](https://pingboard.com/security).

I highly recommend adding the Papertrail Gem to any Rails app where data is owned by multiple users.  It may be overkill and wasteful in an app where data is always owned by one user, and unable to be changed by anyone else.

Read more about Papertrail [here](https://github.com/paper-trail-gem/paper_trail)!
