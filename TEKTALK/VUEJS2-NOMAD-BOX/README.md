# VueJS2 App Lifecycle from Startup to Scale using Laradock + Nomad Box

#### _`Michael Leow: Lead, Freedom of Information Initiatives, Sinar Project`_
#### _`KLJS: July 19, 2017`_

## Slides

- VueJS2 App Lifecycle from Startup to Scale using Laradock + Nomad Box - \<TODO\>

## Objective

- Narrative of a startup as it grows from "ideation" to MVP, pivoting and scaling a successful paid "Personal Music Streaming-as-a-Service"
- Show how this VueJS2 App (Koel) is built with Laradock (http://laradock.io/) with its use of Workspace to simplify development to target Docker container
- Show production deployment onto Nomad Box service fabric using Blue/Green deployment pattern.  Demo the use of consul-template to change production credentials/URLs in real-time
- Demonstrate operational simplicity of scaling a successfull App and show how multiple versions of the application can exist simultaneously, catering to both "free" + "paid" category of users.

## Code Used

- Vue2JS App: Music Streaming Server - https://github.com/phanan/koel
- Laradock - https://github.com/laradock/laradock/
- Nomad Box - https://github.com/leowmjw/nomad-box


## Pre-req Reading

- Introduction to Nomad Box on Azure, DevCon Jan2017 - https://goo.gl/ReU12f
