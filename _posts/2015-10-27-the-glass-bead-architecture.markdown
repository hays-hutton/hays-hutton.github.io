---
layout:     post
title:      "The Glass Bead Architecture, Part I Overview"
subtitle:   "Distributed Two Tier Application State with TAO (Transactions, Analytics, and Operations)"
date:       2015-10-27 10:00:00
author:     "Hays"
header-img: "img/GlassBead.png"

---

The Glass Bead Architecture
===========================

For over a year, I have been iteratively designing and building
the full stack for a mobile phone app.
I started by building micro services with [Clojure](http://clojure.org/) and [Datomic](http://www.datomic.com) while we tried to hire an Objective-C developer
(FWIW micro services probably aren't the place to start unless you have a few teams that would benefit from the separation like Netflix does).
The Objective-C developer never materialized so I moved to implement a hybrid architecture based on [Meteor](http://www.meteor.com), [Ionic](http://ionicframework.com/), and [Cordova](https://cordova.apache.org/).
Meteor taught me to embrace PubSub and to be ok with the document model.
Ionic and Cordova taught me that for the time being native apps are better. [React Native](https://facebook.github.io/react-native/) launched
at about the same time that I was contemplating Objective-C again. I am thankful for React Native and find that it is the best 
answer for iOS and more. For a while, I continued with [Mongo](https://www.mongodb.org/) as the database and filled in the server tier with [hapi.js](http://hapijs.com/) on [Node](https://nodejs.org/en/) plus some [Oplog tailing](https://www.mongodb.com/blog/post/tailing-mongodb-oplog-sharded-clusters)
to drive events into my analytics tool chain. I prototyped an implementation of [Flux](http://facebook.github.io/flux/), which I really didn't care for, before I found [Redux](http://redux.js.org/).
I love Redux. It is small with ~ 100 lines of "real" code. It is the core of this architecture with it's functional and data first model.
Concurrent with Redux, I pulled in [Firebase](https://www.firebase.com/) to handle client side data interaction (which reminded me of Meteor's pubsub [DDP](https://www.meteor.com/ddp) which I had liked).
On September 1st I watched Joe Witt's [presentation](https://nifi.apache.org/videos.html) on [Apache nifi](https://nifi.apache.org/index.html) and everything really started to come together to what this
post is about: the thing I call The Glass Bead Architecture. (It's an homage to Hermann Hesse's [The Glass Bead Game](http://www.amazon.com/Glass-Bead-Game-Magister-Novel/dp/0312278497/) and
a play on [Java bean](https://en.wikipedia.org/wiki/JavaBeans) -> bead). Nothing I have done here is anymore than putting together some of the best tools in a novel way but I 
do think it points to the future of app development with distributed data, deep analytics, and shared logic in a two tier model.

![overview](/img/GlassBeads.png)

Definitions
-----------
**Glass Bead**: A redux action. Immutable. Persistable. Shareable when
it makes sense. Analagous to a Java bean. Nothing more than a map
(or JS object if you will). A JSON blob representation of an action to be
applied via a function to the current state which client side is in the Store
and pipeline side in an entity key value. It is like either the Command or Event
in the CQRS or Event Driven models. In a sense, it is less defined than either of
those on purpose. These are what gets stored and distributed and "replayed"
or reduced. Each bead has at least the following: 

   *  *bead key*: a time based unique key. Able to be created independently without
      fear of collisions. Similar to a UUID but with time embedded and exposed for
      indexing. Firebase's version of this is a lexicographical key which is still
      ordered based on time too. (See also: [The 2^120 Ways to Ensure Unique Identifiers](https://www.firebase.com/blog/2015-02-11-firebase-unique-identifiers.html))
      Timestamp isn't sufficient in a distributed system where multiple clients
      could create timestamps concurrently. This is implementation dependent but
      should usually be chronologically sortable (timestamp ordering enabled at
      a "best" effort should usually be good enough) See also: [squuids](http://docs.datomic.com/clojure/#datomic.api/squuid)
   *  *at*: the timestamp
   *  *entity*: the entity to which the bead refers (e.g. a User or a domain entity like a Vehicle in my case)
   *  *key*: the entity's key. This must be unique system wide. Sometimes it is
      a domain key.
   *  *type*: the type of bead (e.g. USER_CREATE or VEHICLE_CREATED msg) 

![example bead](/img/beadExample.png)

**Entity**: A collection of glass beads. A stream of actions. In the distributed store a container for
pointers to the glass beads themselves which are normalized. In the client, this is the referenced observer.
In a logical model this is what you think it is: an entity. 
A typical example of a root Entity is a User while the next level down is a Vehicle. Both provide authorization contexts and
pointers to beads which are normalized. Once all of an entity's beads are reduced you have the "current state" of the Entity.

![example beads and users](/img/beadsUsersExample.png )

![example beads and vehicles](/img/beadsVehiclesExample.png )

**Store**: A redux store. The keeper of the derived state. Each client application has one of these. No two are necessarily the same nor really even likely
the same. Although many can have large overlaps. In a "pure" implementation, these will often start off as an empty [] or {} which get built
up by reducing beads.

**SuperStore**: The store at the TAO. This is used to reduce beads at the start of the transactions, analytics, and operations pipeline. This reduction is per Entity which is
different from the client store which produces state for all of the entities which each client application contains. In essence, the superstore is 
a just a super client that has registered to listen to all entities.

**Reducer**: A composable function which takes state and bead -> returns stateNext. fn(state, bead) -> state. This is really the
crux of everything. An old functional programming construct. 

>  A reducing function is simply a binary function, akin to the one you might pass to reduce. While the two arguments might be treated symmetrically by the function, there is an implied semantic that distinguishes the arguments:
>  the first argument is a result or accumulator that is being built up by the reduction, while the second is some new input value from the source being reduced.
>  While reduce works from the 'left', that is neither a property nor promise of the reducing function, but one of reduce itself.
>  So we'll say simply that a reducing fn has the shape:
> 
>    (f result input) -> new-result
>      
>  --Rich Hickey
>    [Anatomy of a Reducer](http://clojure.com/blog/2012/05/15/anatomy-of-reducer.html)

**Marver**: An npm library (mine is currently private) which presents the consistent API of the application to each of the clients (iOS, Android, the super store, an SPA, or even a server tier if you really wanted).
The creator of the glass beads and the entities. It persists most beads but can flow beads through without persistence if necessary. In some ways the Marver is kind of the whole point.
When React Native promises "learn once, write anywhere", the Marver is the place for the shared code. This is the place to put the things that are the same for
the app no matter the deployment target. Move your REST api, server tier logic, plus your app tier logic into this one library. This removes
the need for the middle tier and encapsulates all of the business logic in one place which can be used in multiple targets.
The Marver contains the Store, the Reducers, and the application logic to create, or marver, beads and entities too.
All exposed as an api. This could be the MC in MVC, but for the client and the server tier. View code is specific to each target. Use React!

![marver](/img/marver.png)

Benefits
--------

 + Remove a set of servers.
 + Immutable data. Don't update in place. This is good for analytics and good for sharing state across multiple
   clients.
 + Embraces websockets. HTTP and REST certainly have their place. Websockets push you to build your
   apps in a different way than RESTful CRUD. 
 + Streams of data are the future. This is a way to easily manage and think about them.
 + While we certainly aren't there yet, eventually consistent will become more and more important as we tackle problems and architectures defined by being on the 
   edge versus strongly in the center. The Glass Bead Architecture will gracefully work with and will really even shine with [CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) or some kind of [eventual consistency](http://christophermeiklejohn.com/lasp/erlang/2015/10/27/tendency.html)  


Up Next
--------

While this was a brief overview, soon I will delve into more details on parts of this. Specifically, the Marver and the TAO.
