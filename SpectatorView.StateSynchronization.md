# State synchronization
1) On the user device, a [StateSynchronizationBroadcaster](Scripts/StateSynchronization/StateSynchronizationBroadcaster.cs) is enabled, while on the spectator device a 
[StateSynchronizationObserver](Scripts/StateSynchronization/StateSynchronizationObserver.cs) is enabled.
    * These classes are responsible for delegating both network messages and network changes to the [StateSynchronizationSceneManager](Scripts/StateSynchronization/StateSynchronizationSceneManager.cs), which drives scene state synchronization.
    * These classes are used to relay camera location, application time, and performance data from the user to spectator device.
    * These classes allow other components to register for custom network events and send network messages through the [CommandService](Scripts/StateSynchronization/CommandService.cs) (Note: this allows Spatial Alignment components to use the same network connection).


2) In both the user and spectator application, [ComponentBroadcasterServices](Scripts/StateSynchronization/ComponentBroadcasterService.cs) register with the [StateSynchronizationSceneManager](Scripts/StateSynchronization/StateSynchronizationSceneManager.cs).
      * [ComponentBroadcasterServices](Scripts/StateSynchronization/ComponentBroadcasterService.cs) specify [ComponentBroadcaster](Scripts/StateSynchronization/ComponentBroadcaster.cs) types for in scene class types. This allows broadcasters to be created as new components are added to the user application scene.
      * [ComponentBroadcasterServices](Scripts/StateSynchronization/ComponentBroadcasterService.cs) also register for a specific id so that they can receive network messages and create ComponentObservers in the spectator scene.


3) When the [StateSynchronizationBroadcaster](Scripts/StateSynchronization/StateSynchronizationBroadcaster.cs) observes that a [StateSynchronizationObserver](Scripts/StateSynchronization/StateSynchronizationObserver.cs)
 has connected, the user's scene is configured to be broadcasted. Configuring the user scene for broadcasting requires adding TransformBroadcasters to root game objects of content that is intended to be synchronized. This can be achieved through different manners:
      * [GameObjectHierarchyBroadcaster](Scripts/StateSynchronization/GameObjectHierarchyBroadcaster.cs) items in the Unity scene will add a [TransformBroadcaster](Scripts/StateSynchronization/TransformBroadcaster.cs) to their associated game object.
      * If [BroadcasterSettings.AutomaticallyBroadcastAllGameObjects](Scripts/StateSynchronization/BroadcasterSettings.cs) is set to true, a [TransformBroadcaster](Scripts/StateSynchronization/TransformBroadcaster.cs) will be added to the root game object of every scene (This is DISABLED by default in SpectatorView).


4) On awake and for every related hierarchy change, the [TransformBroadcaster](Scripts/StateSynchronization/TransformBroadcaster.cs)
 will ensure that all of its children also have [TransformBroadcasters](Scripts/StateSynchronization/TransformBroadcaster.cs). On creation, [TransformBroadcasters](Scripts/StateSynchronization/TransformBroadcaster.cs) also make sure that their associated game objects have [ComponentBroadcasters](Scripts/StateSynchronization/ComponentBroadcaster.cs) created for all components with registered [ComponentBroadcasterServices](Scripts/StateSynchronization/ComponentBroadcasterService.cs). This effectively sets up the classes needed for components in the user application to broadcast state information to spectator devices.


5) After each frame on the user device, the [StateSynchronizationSceneManager](Scripts/StateSynchronization/StateSynchronizationSceneManager.cs) will monitor network connection changes. It also determine if any [ComponentBroadcasters](Scripts/StateSynchronization/ComponentBroadcaster.cs)
 have been destroyed. It then hands all of the known network connections to each [ComponentBroadcaster](Scripts/StateSynchronization/ComponentBroadcaster.cs)
 so that state information can be sent to the spectator devices.


6) On the spectator device, the [StateSynchronizationSceneManager](Scripts/StateSynchronization/StateSynchronizationSceneManager.cs) will receive network messages to relay to the appropriate [ComponentBroadcasterServices](Scripts/StateSynchronization/ComponentBroadcasterService.cs). These messages signal component creation, updates and destruction on the users device. This component state information also contains unique component ids that allow specific instances of [ComponentBroadcasters](Scripts/StateSynchronization/ComponentBroadcaster.cs)
 on the user device to map 1:1 with specific instances of [ComponentObservers](Scripts/StateSynchronization/ComponentObserver.cs) on the spectator device. Through this state information, the spectator device's scene is updated to reflect content on the user's device.
