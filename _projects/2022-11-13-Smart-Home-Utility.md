---
title: Smart Home Utility
subtitle: Proof-of-concept solution to manage household utilities leveraging smart home technology.
date: 2022-11-13 00:00:00
description: 
featured_image: SHU-title.png
accent_color: '#4C60E6'
gallery_images: 
  - demo.jpg
  - demo.jpg
  - demo.jpg
---
**Technology used:** C#, Unity3D, AWS Serverless Architecture, Android Studio

Smart Home Utility is a proof-of-concept that let the user manage their household utilities (gas, hydro, and electricity) usage through a mobile phone

![](/images/SHU-overview.png)

SHU comprises of 4 main components:
1. Smart meters
2. Central IoT Hub
3. AWS Database
4. Mobile application

The meters will be installed on the corresponding pipes within the household. These meters will constantly records the utilities' usages which are stored in the database and displayed on the mobile application.
The user can monitor the usage, set usage threshold, and recieve notifications through the app

---
#### My work
To easily demonstrate the capabilities of SHU, I programmed a simulation using Unity3D which showcases how the solution works in practice.

![](/images/SHU-3d.png)

The cylinder model represents the user in their household along with appliances that simulate the consumption of each utility. For example, when the lights are turned on, the readings for electricity consumption increases:

{% include post-components/gallery.html
columns = 2
full_width = true
images = "/images/SHU-lights-off.png,/images/SHU-lights-on.png"
%}

This is achieved by writing a C# script to set an initial value, increase the value every frame that the lights are on, calculate the differences, 

{% raw %}
```liquid
 IEnumerator measuringElecUsage(int updateTimer)
    {
        // Send requeset using API after delaying for "timer" seconds.
        while (true)
        {
            yield return new WaitForSeconds(updateTimer);
            if ((light_1.enabled && !light_2.enabled) || (!light_1.enabled && light_2.enabled))
                currentElecUsage += elecDefaultValue;
            // Light 1 = 1 && Light2 = 1
            else if (light_1.enabled && light_2.enabled)
                currentElecUsage += elecDefaultValue * 2;
            else
                currentElecUsage = 0.0m;
            totalElecUsage += currentElecUsage;

            // Debug.Log("Elec Usage : " + currentElecUsage);
        }
    }
    
```
{% endraw %}

then finally send the data as a JSON to the AWS endpoint.

{% raw %}
```liquid
public async Task SendRequest()
    {
        var req = new Request
        {
            id = "iNAanuUM", // id
            type = "centralHub", // type
            devices = new List<IDevice>{ // devices
        ...
        new Device{
            id = "FKUsIR3l",
             type = "ELECTRICITY_METER",
              consumption = new Consumption
              {
                value = currentElecUsage,
                 unit = "wH",
                 }},
        }};
    
        var s = JsonConvert.SerializeObject(req, new JsonSerializerSettings
        {
            Formatting = Formatting.Indented
        });

        var httpClient = new HttpClient();
        httpClient.DefaultRequestHeaders.Add("token", "*******");
        var response = httpClient.PostAsync("***.amazonaws.com/smart-utility-iot/devices", new StringContent(s)).GetAwaiter().GetResult();
        ...
    }  

```
{% endraw %}

---
#### Acknowledgement

Smart Home Utility is a Master academic group project for the course Advanced Software Engineering at University of Windsor.
This is an ambitious group project that tested the limit of the team's technical knowledge, resources management, and collaborative skills.
I am grateful for the contributions of my teammates who participated. 

Feel free to check out more details regarding this project [here](https://docs.google.com/document/d/1FvRi-afj2WumNYQj4AEg4FvXbbNEvSQv/edit)
