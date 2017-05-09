# Laravel App Lifecycle from Dev to Prod using Laradock + Nomad Box

#### _`Michael Leow: Lead, Freedom of Information Initiatives, Sinar Project`_
#### _`Laravens: May 27, 2017`_

## Objective

- Show how a typical Laravel App (Lets-Build-a-Forum-in-Laravel) built with Laradock (http://laradock.io/) with its use of Workspace to simplify development to target Docker container
- Show how to "compile" the Laravel App so that it is ready for CI testing and production deployment
- Show problem in the "Lets-Build-a-Forum-in-Laravel" in the DB migration and how to handle in order to have 2 versions of app deployed
- Production deployment onto Nomad Box service fabric using Blue/Green deployment pattern.  Demo the use of consul-template to change production credentials/URLs in real-time

## Code Used

- Forum in Laravel - https://github.com/laracasts/Lets-Build-a-Forum-in-Laravel
- Laradock - https://github.com/laradock/laradock/
- Nomad Box - https://github.com/leowmjw/nomad-box


## Pre-req Reading

- Introduction to Nomad Box on Azure, DevCon Jan2017 - https://goo.gl/ReU12f
