# Porting the OpenSpending Application into VueJS Components

## Audience

Application Developers looking for details on how an Angular v1 
application can be decomposed to its constituent and be ported out 
as a separate and reusable VueJS Component
 
## Objective

- Learn how to safely
- Have an independent components usuable by VueJS developers.  Have an example
application that will demonstrate the Joined-Up Data principle but the OSNext
Application existed by itself.  

## Scenario

For the high-level details and example code; see the companion article at 
[here](./Visualize-JoinedUp-Budget-OpenSPending.md)

This is meant for Developers 

The extracted low-level components can be found at: 
[Babbage.UI Github Repo](https://www.github.com/openspending/babbage.ui)

The components in the OpenSpending Next is available in the babbage.ui repo
where it is done as Angular components.  VueJS is an up-and-coming framework
for the View part of MVCs


## Methods

As the OpenSpending repo code was pre-Alpha, there was a need to track the
libarries as their being developed

The components will react to a 'babbage-update' event signal.

The whole process involved two phases:
    a) Phase 0: Pre-requisite setup
    b) Phase 1: Port over individual Components
    c) Phase 2: Port over OSNext Application using Component

### Phase 0: Pre-requisite setup

- The demo app runs the latest VueJS 1.x (VueJS 2.x have been released in 
the meanwhile) to handle the View portion of the standard MVC pattern.
- Next, we need the API data endpoints (both which were still being developed
along th way )
- Deployment; since this is a Demo and PoC, custom subdomain is not 
as important.  In light of that, Netlify was seleced to macth criteria
of low/no cost with easy deployment/test steps.
 
### Phase 1: Port over individual Components

Firstly, the top 3 components: TreeMap, BubbleTree and Pie components are ported
over individually.  This ease testing as data structures can be passed in via
the props

Each individual components ported over (TreeMap, BubbleTree, Pie) 
VueJS makes it much cleaner to separate out the internal state of the 
components; for example TreeMap has an additional TreeMapTable.

The babbage.ui comes with its own API implementation; which has some 
being repeate in the Application.

#### Phase 1a: Porting the API

Speciail concept like drilldown, asking for the right filter needed to b
be understood "on-the-wire"

#### Phase 1b: Standardize the messaging

Conceptually, the actions available is to drill down and the 

#### Phase 1c: Unmodified

are able to run/generate

Benefit is that with the test of each component wrapped by VueJS
standalone, it can be easily tested by hard coding values to understand
its behavior and can be tested in a headless manner; which makes it faster.


Any bugs found (like TreeMap not able to handle) must be pushed back 
upstream.  Of particular challenge is the BubbleTree component as 
it i using a very old from the okfn repo; and has dependencies on 
jQuery being available.

So althrough try to remove jQuery as adependency; it was not very
successful.

Since the other restriction would be to not modify upstream (other than
proven bugs); or reloading the whole component and cannot be efficiently
cache between calls.

### Phase 2: Port over OSNext Application can Component

Actually, as the concept of BabbagePackage which can embed the components
 added in via Phase 1
 
When there is action "babbage-click", it triggered a drill-down
 
The BabbagePackage component will have a concept of the tracked "packageid"
which is externally passed in from the larger application (see the Joined Up
data article for more details)
 
 Compare this Angular code with its Vue equivalent; which since it is
 well tested, it is reusable.
 
 When budgets are selected; it is destroyed and
 
Since OSNext is loaded with many abilities; but I was only interested in
showig data un the 3 most fancy visualization (SanKey, Radar comes later)

Pie was the easiest to port, TreeMap next and finally BubbleTree (which
is very complex)

### Glbal challenges

many npm packages; and it is a challenge to try to minimize it

### Deployment
only read-only using 3rd party API (POpPit by Sinar project and 
OpenSpending by OKFN, it could simply run ; it is actually quite easy to deploy it at no / low cost
by for demo is cool :) 

It is also advantageous that Netlify's CDN allow the single JS generated 
by Webpack (not optimized!)

### Integrating External Repository using git-subrepo

The repo is at URL: https://github.com/ingydotnet/git-subrepo.git

Clone it and have a startup script somethign like below

To use it, source the following like below:

```
leow$ cat source-git-subrepo 
GITSUBREPO=`pwd`

echo "Turn on Git Completion!"
source ${GITSUBREPO}/git-subrepo/share/git-completion.bash

echo "Turn on Git SubRepo Completion!!"
source ${GITSUBREPO}/git-subrepo/share/completion.bash

echo "Setup Git SubRepo!!"
source ${GITSUBREPO}/git-subrepo/.rc
```

As a bonus, you get auto-completion for git commands

### Porting AngularJS to VueJS

It is quite simple; a lot can be simplified

This is encapsulated in a higher order component called the BabbagePackage

At an even hgher level; there is the BabbageUI Component which acts
like a strip down os-viewer.


In VueJS; the internal state/structure

To communicate up and down the application; the internal messaging events
is used; it is much cleaner to see vs the Angular

## How to help?

The code for the components have been nicely pulled in to 
## Conclusion + Future Work

Breadcrumb component is needed; so that there is ability to navigate
the hierarchy.

Need to port in the latest components; port it to support VueJS 2.0
functionalyi as the message passing 

See the next article on how to use these components in a joined-ip manner

