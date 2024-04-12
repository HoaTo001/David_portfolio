---
title: Moodle Webhook Plugin
subtitle: Plugin for Moodle LMS that allow courses to send notification webhooks to custom endpoints
date: 2024-03-12 00:00:00
description: 
featured_image: MWP-title.png
accent_color: '#4C60E6'
gallery_images: 
  - MWP-title.png
  - SHU-lights-off.png
  - SHU-lights-on.png
---
#### Technology used:
* Moodle 4.3
* XAMPP
* PHP
* MySQL

Moodle is a Learning Management System (LMS) that is used by academic organizations to provide instructors with the ability to manage students' progresses, organize teaching materials, and customize their own teaching methods.

Moodle's functionalities benefit from its rich plugin (official and custom) library. This project's purpose is to enhance and customize the notifications feature within Moodle.

---
#### My work
I developed a local plugin for Moodle which allows each course to send notifications webhooks to
its custom endpoint. 

This is achieved using Moodle's [Events API](https://docs.moodle.org/dev/Events_API) and [Custom fields](https://moodledev.io/docs/4.4/apis/plugintypes/customfield).

Firstly, I created a course custom field to store the endpoint to which we want the webhooks to be sent.
Doing this allows us to sent notifications to different endpoints correlated to each course.

![](/images/MWP-customfield_1.png)

Furthermore, data in course custom field can be modified anytime via course settings.

![](/images/MWP-customfield_2.png)



---
#### Motivations
[Moodle's notifications](https://docs.moodle.org/403/en/Notifications) alert students, teachers, and other users
through