# Porting the OpenSpending Components using VueJS

## Scenario

The components in the OpenSpending Next is available in the babbage.ui repo
where it is done as Angular components.  VueJS is an up-and-coming framework
for the View part of MVCs

## Objective

Have an independent components usuable by VueJS developers.  Have an example
application that will demonstrate the Joined-Up Data principle but the OSNext
Application existed by itself.  

## Methods

As the OpenSpending repo code was pre-Alpha, there was a need to track the
libarries as their being developed

The components will react to a 'babbage-update' event signal.

The whole process involved two phases:
    a) Phase 1: Port over individual Components
    b) Phase 2: Port over OSNext Application can Component

### Phase 1: Port over individual Components

Firstly, the top 3 components: TreeMap, BubbleTree and Pie components are ported
over individually.  This ease testing as data structures can be passed in via
the props

Each individual components ported over (TreeMap, BubbleTree, Pie) 
VueJS makes it much cleaner to separate out the internal state of the 
components; for example TreeMap has an additional TreeMapTable.

### Phase 2: Port over OSNext Application can Component

Actually, as the concept of BabbagePackage which can embed the components
 added in via Phase 1
 
When there is action "babbage-click", it triggered a drill-down
 
The BabbagePackage component will have a concept of the tracked "packageid"
which is externally passed in from the larger application (see the Joined Up
data article for more details)
 
### Integrating External using git-subrepo

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

In VueJS; the internal state/structure

To communicate up and down the application; the internal messaging events
is used; it is much cleaner to see vs the Angular

## Conclusion + Future Work

Need to port in the latest components; port it to support VueJS 2.0
functionalyi as the message passing 

See the next article on how to use these components in a joined-ip manner

