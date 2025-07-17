# Physics Fountain: Multisynq, Rapier physics, and Three.js rendering

A single page [Multisynq](https://multisynq.io) application that demonstrates:
- synchronized physics
- shooting (throwing physical objects into the scene)
- simple camera motion

All this in about 600 lines of code which includes the HTML and styles.

## Multisynq Multiplayer

As usual, the logic is split into the synchronized Model part and the local View part. In this case, the Model uses Rapier for the deterministic simulation, and the View uses Three.js for high-performance 3D rendering.

## Rapier Physics

We use [Rapier](https://rapier.rs) for physics, which is a Rust package compiled to web assembly.
The `@dimforge/rapier3d-deterministic-compat` variant provides deterministic simulation independent of the processor architecture,
while "compat" means it can be used via a CDN (without a bundler). We import it in an `importmap` script.

To define the serialization for the physics objects we extend `RAPIER.World` into our own `MultisynqRapierWorld` subclass.
When a rigid body or collider is created, it adds a `_world` reference to it, which is needed by the `write`/`read` methods in the types declaration for Multisynq.
While Multisynq Models are serialized automatically, [types](https://multisynq.io/docs/client/Model.html#.types) declarations are necessary to define the serialization of non-model classes.

Otherwise no special handling of the physics is needed, it just works. Multisynq takes care of executing it identically for all participants in the session.

## Three.js rendering

The [Three.js](https://threejs.org) renderer is straight-forward. It uses instanced rendering for the physics objects. Its main loop is driven by Multisynq's `update()` method (which uses `requestAnimationFrame()` internally).

For smooth 60 fps rendering of the 30fps simulation there is a smoothing factor applied to interpolate `currentPos` towards `targetPos` for each instance. To avoid the initial swoosh from the creation position (out of sight at -100 depth) to the first real position, `currentPos` is snapped to `targetPos` if the distance between the two is too large.

## UI

* Click or tap to shoot
* Drag to spin
* Wheel to move closer/further away
* press R to reset the camera for everyone
