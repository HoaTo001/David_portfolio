---
title: Moodle Webhook Plugin
subtitle: Plugin for Moodle LMS that allow courses to send notification webhooks to custom endpoints
date: 2024-03-12 00:00:00
description: 
featured_image: MWP-title.png
accent_color: '#4C60E6'
gallery_images: 
  - MWP-title.png
  - MWP-discussioncreated_2.png
  - MWP-discussioncreated_3.png
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

Moodle Webhook plugin works as follows: an instructor "Course A" creates an announcement for the final exam which takes place next week. The data of this announcement will then be sent to the endpoint URL assigned to "Course A". Similarly, each announcement of each course will be sent to that course's endpoint.

{% include post-components/gallery.html
    columns = 1
    full_width = true
    images = "/images/MWP-discussioncreated_1.png,/images/MWP-discussioncreated_2.png,/images/MWP-discussioncreated_3.png
"
%}

This is achieved using Moodle's [Events API](https://docs.moodle.org/dev/Events_API) and [Custom fields](https://moodledev.io/docs/4.4/apis/plugintypes/customfield).

Firstly, I created a course custom field to store the endpoint to which we want the webhooks to be sent.
Doing this allows us to sent notifications to different endpoints correlated to each course.

![](/images/MWP-customfield_1.png)

Furthermore, data in course custom field can be modified anytime via course settings.

![](/images/MWP-customfield_2.png)

Secondly, I needed to create an event observer and handler for each type of event that we want to be notified of. For example,
if we want to be notified when a new discussion is created we need to:
* Add the event we want to catch (mod_forum\event\discussion_created)
   
**events.php**

{% raw %}
```liquid
$observers = array(
    array(
        'callback'    => 'local_webhookdata_observer::discussion_created',
        'eventname'   => 'mod_forum\event\discussion_created',
        'includefile' => null,
        'internal'    => true,
        'priority'    => 200,
    ),
```
{% endraw %}

* Create the handler for the event. Noted that the logic for the handler has to be queued using adhoc task so that it can be run in the background without impacting the performance of the site.

**observer.php**

{% raw %}
```liquid
class local_webhookdata_observer {
    // Listener for \mod_forum\event\discussion_created event
    
    public static function discussion_created(\mod_forum\event\discussion_created $event) {
        // Get event data from Moodle's Events API.
        $eventdata = $event->get_data();

        // Create an ad hoc task and run it in the background
        $task = \local_webhookdata\task\discussion_create_noti_task::instance($eventdata);
        \core\task\manager::queue_adhoc_task($task, true);
    }
```
{% endraw %}

* Create the logic for the handler (extract event data->get course's endpoint->add relevant info->send data)

**discussion_create_noti_task**

{% raw %}
```liquid
class discussion_create_noti_task extends \core\task\adhoc_task {
    ...
    
        $eventdata = $this->get_custom_data();

        // Get related course, instructor data, callback url.
        $user = $DB->get_record('user', ['id' => $eventdata->userid]);
        $course = $DB->get_record('course', ['id' => $eventdata->courseid], 'fullname');
        $url = local_webhookdata_get_course_callback($eventdata->courseid);
        $post = $DB->get_record('forum_posts', ['id' => $eventdata->objectid]);

        // Append related data to event data.
        $eventdata->url = $url;
        $eventdata->instructor = $user->firstname . ' ' . $user->lastname;
        $eventdata->course = $course->fullname;
        $eventdata->subject = $post->subject;
        $eventdata->message = strip_tags($post->message);
        $eventdata->event = 'MOODLE_DISCUSSION_CREATE';

        // Send a POST request to the callback URL.
        $curl = curl_init();
        curl_setopt($curl, CURLOPT_URL, $url);
        curl_setopt($curl, CURLOPT_POST, true);
        curl_setopt($curl, CURLOPT_POSTFIELDS, json_encode($eventdata));
        curl_setopt($curl, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);
        $response = curl_exec($curl);
        // Check for errors
        if ($response === false) {
            $error = curl_error($curl);
            // Handle the error
        }
        curl_close($curl);
    }
```
{% endraw %}

---
#### Motivations
[Moodle's notifications](https://docs.moodle.org/403/en/Notifications) alert students, teachers, and other users
through
