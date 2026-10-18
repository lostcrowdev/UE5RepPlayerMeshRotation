# Custom Mesh Rotation Component

A replicated Actor Component plugin for Unreal Engine 5 that rotates a Character's mesh independently of the capsule.

## Pre-requisites

### Engine versions

- Unreal Engine 5.x (update with the version you built it in).

- No additional plugins required.

### Requirements

- The owning Actor must be a Character (the component casts its owner to Character on BeginPlay).

- The Character and the component must both be set to replicate for multiplayer use.

## What it does

The mesh is rotated in world space every tick, separate from the Character's capsule and control rotation. This allows:

- The mesh to turn toward the direction of movement.

- The mesh to face the camera, or look at a specific Actor.

- Adjustable body pitch (lean) based on vertical velocity, example when swimming, flying, etc...

- Smooth, rate-limited turning for both yaw and pitch.

## Setup

1. Add BP_PlayerMeshRotation to your Character Blueprint.

2. Disable Use Controller Desired Rotation and Orient Rotation to Movement.

3. Enable / Confirm **Start with Tick Enabled** (Component Tick).

4. Enable / Confirm **Component Replicates** on the component (make sure the Character replicates).

5. Set **Base Mesh Yaw Offset** so the mesh faces forward (example: -90 for the UE mannequin).

6. Compile and play.

## How it works

On BeginPlay the component gets its owner, casts it to Character, and stores it as OwnerCharacter. Tick only runs its logic if this reference is valid.

### Yaw Target

Runs on the locally controlled client and on the server. Picks the target yaw using the first condition that applies:

- **Face Camera Forward** is enabled: use the Character's control rotation yaw.

- **Look at Valid** is true: use the yaw of the look rotation toward the look-at target.

- **Movement input** (current acceleration relative to max acceleration) is above **Min Input for Yaw Update**: use the yaw of the acceleration direction.

- Otherwise: hold the current yaw.

- Can be used for a targetting system as well, since you can just call SERVER_SetLookAtTarget on an actor, and the player yaw will rotate towards the actor. Use SERVER_ClearLookAtTarget to clear targets.

### Pitch Target

Takes the Character's vertical velocity and can be adjusted with **Max Body Pitch**.

### Interpolation

Current yaw and pitch move toward their targets at a constant rate (RInterp), using **Yaw Turn Rate** and **Pitch Turn Rate** in degrees per second.

### Applying the rotation

The final rotation is the base rotation (Base Mesh Roll, Base Mesh Yaw Offset) composed with the interpolated rotation (Current Mesh Pitch, Current Mesh Yaw). It is applied to the Character's Mesh with Set World Rotation.

## Replication

- The server stores its Target Mesh Yaw in **Replicated Target Yaw** (RepNotify), so other clients can follow it.

- **SERVER_SetLookAtTarget** and **SERVER_ClearLookAtTarget** are server RPCs that set or clear the look-at Actor.

### Debug

- **Show Looking Debug** - Draws look-at debug info.

- **Debug Target Index** - Selects which target to debug.

## Usage from other Blueprints

Get a reference to the Character, then use **Get Component by Class** with **Component Class** set to BP_PlayerMeshRotation. Promote the result to a variable and reuse it instead of fetching it every tick.
