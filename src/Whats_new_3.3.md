# What's New in Hazelcast 3.3 EA



## Release Notes

### New Features
This section provides the new features introduced with Hazelcast 3.3 release. 

-	Heartbeat for Java client: Before this release, a Java client could not detect a node as dead, if the client is not trying to connect to it. With this hearbeat feature, each node will be pinged periodically. If no response is returned from a node, it will be deemed as dead. Main goal of this feature is to decrease the time for detection of dead (disconnected) nodes by Java clients, so that the user operations will be sent directly to a responsive one.

In this way, user operations will be channelled to other nodes.

-	Tomcat 6 and 7 web Sessions Clustering: Please see [Session Replication](#session-replication).

-	Replicated Map implemented: Please see [Replicated Map](#replicated-map-beta).










