# Overview

How The Fabric works

## Parts of the Fabric

The Fabric consists of two programs, the **client** and the
**server**, and a set of **resources** used by both.

The **resources** used by both the client and the server are files
containing the data that the game needs to simulate the game world and
present it in real time in 3D. Fabric resources include images of
planets and asteroids, fonts used to present text in the interface,
authentication data used to grant access to registered players, data
describing the current state of the game world, and so on.

The **server** runs on a remote host provided by the Fabric's
developers. It stores and updates the virtual world simulated by the
game, as well as the user-account information needed for players to
interact with the world and other players.

The server is a "headless" program (that is, one without a graphical
user interface) that runs as a Java process on a network-accessible
host managed by the Fabric team.

The **client** runs on each player's computer and presents an
immersive 3D view of the virtual world of the Fabric. It enables users
to create and play characters, to explore the Fabric's world, and to
share experiences online with other Fabric players.

The client is a desktop windowing application that uses Java and
OpenGL to present a 3D view of the Fabric universe.

## How the client starts up

The Fabric is a Java application. When you launch it, Java starts a
process and loads the Fabric's jarfile. That jarfile creates a window
and an OpenGL context that are then used to render views of the Fabric
game world.

The Fabric is built in Scheme using Kawa, a Scheme compiler that turns
Scheme code into Java bytecode. It builds on JMonkeyEngine 3.0, a 3D
scene-graph engine based on the design described in David Eberly's book
*3D Game Engine Design: A Practical Approach to Real-Time Computer
Graphics*.

At startup, the Fabric instantiates the class FabricClient, a subclass
of com.jme3.app.SimpleApplication, the basic representation of a
JMonkeyEngine class. The process initializes the new instance of
FabricClient, then calls its `start` method to start the game. That
method sets up the initial scene graph and OpenGL context and starts
event handling.

All of the Fabric-specific functionality of the application is
implemented in Scheme in the source files found in the_fabric/src. The
only code not written in Scheme in the Fabric is code found in the
library jars in the_fabric_libraries.

After the Fabric app starts up, it chooses an initial **AppState** and
initializes it. An **AppState** is a JME3 data structure that
represents a state that an application can be in. As an example, the
Fabric presents an initial set of user-interface elements that a
player can use to connect to the Fabric server or to create a new
account there. After it logs a player in, the client changes its
display to show the characters the player can choose to play, and the
option of creating new characters. The login screen and the
character-selection screen are represented by two different AppStates.

Similarly, after a player has chosen a character and entered the game
world, he or she has opportunities to visit any of a variety of
locations in the game. Each location is represented by an AppState
that knows how to display the celestial objects in the player's
vicinity, and how to render other players and characters in
surrounding space.

## Network connections

In order to communicate with the server and with other players'
clients, a client must make network connections to the Fabric
server. Each AppState manages its own set of network connections:
connections for authentication and authorization in the login
AppState; connections for browsing, selecting, and creating characters
in the character-picker, and connections for moving through and
interacting with the world in the gameplay appstates.

Certain connections may be shared among several AppStates. For
example, players can use in-game chat channels to talk with one
another while playing the game. When a player moves a character from
one location to another through the in-game transit system, the game
hands off most of the chat channels from the starting AppState to the
destination AppState. Channels specific to a particular location are
not handed off in this way when a player travels.

## Server startup

The Fabric server doesn't start up as often as the Client; instead,
it's designed to run for long periods without restarting. Conceptually
(and actually, at the moment), the server is a single process on the
single machine that represents the Fabric world and serves
communication features that enable disparate clients to interact with
one another as if sharing the game world. The server maintains
authoritative information about the position, orientation, velocity,
and other attributes of player and non-player characters, landscape
features, buildings, and other objects in the game. It distributes
these data and changes to them to all connected clients in order to
enable them to render the same shared game world on many client
machines at (close to) the same time.

The server provides no rendering services. It is completely headless,
relying entirely on the clients to render views of the game world.

On the other hand, all authoritative data about positions, velocities,
transient states, effects of abilities used, and other such
gameplay-related data, are maintained by the server and only the
server. The client has no authority over game-state data so that game
clients cannot enable players to cheat.

Like the client, the server is a Java/JME3 application written in
Scheme using Kawa. Lacking any GUI, it's a command-line
application. When it's started, Java runs and loads the server
jarfile, instantiates and initializes FabricServer, another subclass
of com.jme3.app.SimpleApplication, and then creates and initializes
network-listener objects that manage client connections.

The fabric server reads configuration data from resources installed
alongside it. These data enable the server to run locally for
development purposes, or remotely for production and testing
purposes. Starting the Fabric server means running a Java command to
launch the server's jar file, with appropriate configuration data in a
config file where the server can find it.

## Coordination between client and server

There are two modes of operation for the client:

1. **local,** in which the server runs on the same host as the client,
normally used only in development and testing

2. **remote,** in which the server and client run on different hosts,
used in production

The only practical difference between local and remote modes of
operation is the client's configuration, which is determined by the
file init-config.scm in the case of remote operation, or
init-config-local.scm in the case of local operation. Changing a
single line of code in the client build controls whether it operates
in local or remote mode.

The primary advantge of running in local mode is that a developer can
run the client and server in the same Emacs session, enabling easy
debugging and the ability to inspect the internal states of both
client and server at the same time.

## Interactive development of client and server

The normal mode of development is to work in an Emacs session with a
running Kawa repl with the Fabric library jars loaded. In this mode of
work, a developer can easily start and stop both client and server
from the Emacs repl, make changes to code in Emacs buffers, and reload
the buffers to effect changes to the working programs.

Please note that because of limitations on the JVM, Kawa does not sort
the same degree of dynamism that traditional Lisp environments do. In
particular, it's possible to reload a changed source file in a running
Fabric session, but there's no guarantee that the new definitions will
work. In fact it's common for the JVM on which Kawa is running to
become confused and crash, especially when Java classes are
dynamically redefined. As a result, the recommended development cycle
for working with Kawa and JME3 is to run, test, identify needed
changes, kill the running Kawa session and restart it, then reload the
changed code. This cycle is not as smooth or quick as a typical Lisp
development cycle, but is still much quicker and smoother than a
typical C or Java cycle.

