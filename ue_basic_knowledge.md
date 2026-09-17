# Unreal Engine Basic Knowledge for MCP

General Unreal Engine knowledge intended to be supplied to a local LLM through RAG when using Unreal MCP.

This document contains stable concepts such as coordinate conventions, Actor/Component/Asset relationships, Unreal reference types, shared Assets, and Material relationships.

It is not a Tool catalogue and does not replace live Unreal MCP Tool Search or verified Recipes.

___
Version: 1.0
Purpose: Provide stable Unreal Engine concepts that help a local LLM interpret Unreal MCP tasks correctly without replacing live MCP schema discovery or verified Recipes.

This knowledge describes general Unreal Engine concepts and reference
relationships.

It is not an exhaustive list of Unreal MCP capabilities.

Live Unreal MCP registry and schemas remain authoritative for current
tools and argument structures.
___

**1. Unreal Engine Coordinate System**

Unreal Engine uses a left-handed coordinate system.

Typical world axes:

+X = Forward

+Y = Right

+Z = Up

Horizontal level placement normally occurs on the XY plane.

Vertical movement normally changes Z.

Examples:

X +100

→ move 100 Unreal units forward


Y +100

→ move 100 Unreal units to the right


Z +100

→ move 100 Unreal units upward

Unreal Engine commonly uses centimeters as its distance unit.

100 Unreal units ≈ 100 cm = 1 meter

Rotation values are generally expressed in degrees.


**2. Actor and Component Are Different Objects**

An Actor is an object placed or existing in a Level.

Examples include:

StaticMeshActor

CameraActor

Light

An Actor may contain one or more Components.

Components provide specific functionality or represent parts of an
Actor.

Examples:

SceneComponent

StaticMeshComponent

CameraComponent

LightComponent

A Component is not interchangeable with its owning Actor.

An operation that modifies the Actor itself may require an Actor
reference.

An operation that modifies mesh-specific properties may require a
StaticMeshComponent reference.

Do not assume that an Actor refPath can be used wherever a Component
refPath is required.


**3. StaticMeshActor, StaticMeshComponent, and Static Mesh Asset Are
Different**

These represent different layers.

A typical relationship is:

StaticMeshActor

    ↓

StaticMeshComponent

    ↓

Static Mesh Asset

For example:

Level Actor

    MCP_Test_Cube


        ↓ contains


StaticMeshComponent


        ↓ references


/Engine/BasicShapes/Cube.Cube

The Actor is a Level instance.

The StaticMeshComponent belongs to that Actor.

The Static Mesh Asset is reusable asset data.

Changing the Actor or its Component is different from modifying the
shared Static Mesh Asset.

If several Actors use the same Static Mesh Asset, modifying that shared
Asset may affect multiple Actors.


**4. Unreal References Have Different Meanings**

Do not treat Unreal names and paths as interchangeable.

Common reference types include:

- Actor label
- Actor instance refPath
- Component refPath
- Asset path
- Object refPath
- Class path

They identify different things.

**Actor label**

A user-visible Level name such as:

MCP_Test_Cube

This is normally not a complete object reference.

**Actor instance refPath**

Example:

/Game/Main.Main:PersistentLevel.StaticMeshActor_2

This identifies a specific Actor instance inside a Level.

**Component refPath**

Example:

/Game/Main.Main:PersistentLevel.StaticMeshActor_2.StaticMeshComponent0

This identifies a Component belonging to an Actor instance.

**Asset path**

Example:

/Game/Materials/M_MCP_Test_Cube_Color

This refers to the Asset location.

**Full object refPath**

Example:

/Game/Materials/M_MCP_Test_Cube_Color.M_MCP_Test_Cube_Color

Some Unreal operations require this complete object reference.

**Class path**

Example:

/Script/Engine.StaticMeshComponent

This identifies a class, not an Actor instance or Asset.

Do not substitute one type of reference for another unless the receiving
operation explicitly accepts it.


**5. Do Not Guess Component References**

Component refPaths should be resolved from Unreal Engine information
rather than constructed by guessing.

For example, given:

/Game/Main.Main:PersistentLevel.StaticMeshActor_2

do not assume that a valid Component reference can be created by
manually appending a guessed name such as:

.staticMeshComponent

Actual Components may have names such as:

StaticMeshComponent0

and their names and structure should be obtained from Unreal Engine.

This applies to Components in general, not only StaticMeshComponent.


**6. Content Browser Names and Unreal Object Paths Are Different**

Users normally refer to Assets by the names visible in the Content
Browser.

Example:

M_MCP_Test_Cube_Color

The same Asset may internally be represented as:

Asset path:

/Game/Materials/M_MCP_Test_Cube_Color

and:

Full object refPath:

/Game/Materials/M_MCP_Test_Cube_Color.M_MCP_Test_Cube_Color

Do not assume the user-visible Content Browser name itself is a complete
Unreal object reference.

Resolve the required internal reference when necessary.


**7. Built-in Basic Shape Assets**

Unreal Engine provides built-in basic shape Assets under:

/Engine/BasicShapes

A commonly used built-in Cube Static Mesh is:

/Engine/BasicShapes/Cube.Cube

These Engine Assets can be referenced by Level Actors.

A placed Cube Actor is not the same object as the shared built-in Cube
Static Mesh Asset.

Avoid modifying a shared Engine Asset when the requested change is
intended to affect only one Level Actor.


**8. Actor-Specific Material Override and Shared Mesh Material Are
Different**

A StaticMeshComponent can use a Material that overrides the Material
defined by its referenced Static Mesh Asset.

Conceptually:

StaticMeshActor

    ↓

StaticMeshComponent

    ├─ Actor/component-specific Material Override

    │

    └─ StaticMesh

          ↓

       shared Static Mesh Asset

          ↓

       default Material slots

Changing an Actor-specific Material override can affect only that Actor
or Component.

Changing the default Material on a shared Static Mesh Asset may affect
other Actors using the same Asset.

Therefore, when the user explicitly requests a visual change to only one
Actor, these two levels must not be confused.

The exact MCP operation used to perform the change should be determined
from live MCP tools or a relevant verified Recipe.


**9. Material Assets and Material Expressions Are Different Objects**

A Material is an Asset.

A Material may contain Material Expressions that form its graph.

For example, a color-producing Expression may exist inside a Material
object.

Conceptually:

Material Asset

    ↓ contains

Material Expression

    ↓ connected to

Material Property / Output

The Material refPath and the Expression refPath are different
references.

Do not use a Material refPath where an Expression reference is required,
or vice versa.


**10. Color Values May Be Represented as Linear Color Components**

Some Unreal material-related properties represent colors using an
FLinearColor-style structure:

R

G

B

A

Typical normalized values range from 0.0 to 1.0.

Examples:

Red:

R=1.0 G=0.0 B=0.0 A=1.0


Green:

R=0.0 G=1.0 B=0.0 A=1.0


Blue:

R=0.0 G=0.0 B=1.0 A=1.0

Do not confuse RGB color components with world-coordinate X/Y/Z values
simply because both may be represented numerically.

The exact property structure must still be checked against the live
schema when modifying it.


**11. Shared Assets Can Affect Multiple Actors**

Unreal projects commonly reuse Assets.

Examples include:

- Static Meshes
- Materials
- Textures
- Blueprints

Multiple Level Actors may reference the same Asset.

Therefore:

changing one Actor instance

and:

changing the shared Asset used by that Actor

are not equivalent operations.

When the user requests:

change only this Actor

avoid modifying shared Assets unless the user explicitly intends a
shared change.


**12. World Space and Local Space Are Different**

Transforms may be interpreted relative to different coordinate spaces.

**World Space**

Position, rotation, or scale is interpreted relative to the Level/world
coordinate system.

**Local Space**

Position, rotation, or scale is interpreted relative to an object\'s
parent or local orientation.

Do not assume that a transform request is automatically local when the
user provides world-level positions.

For Level placement tasks, world-space coordinates are often the
intended interpretation unless context indicates otherwise.


**13. Existing Object, Duplicate, and New Object Are Different
Operations**

These intents should remain distinct.

**Modify existing Actor**

Change an Actor that already exists.

**Duplicate existing Actor**

Create another Actor based on an existing Actor.

**Create/add a new Actor from an Asset**

Create a new Actor using an Asset or class.

These operations may produce visually similar results but are not
semantically equivalent.

Preserve the user\'s requested intent.


**14. Material Editor UI May Not Immediately Reflect MCP Changes**

Changes made through Unreal MCP may modify the underlying object
successfully even when an already-open Material Editor window does not
immediately display the updated value or graph state.

Therefore, stale editor UI alone should not be treated as proof that the
underlying modification failed.

When validating MCP-driven changes, prefer current Unreal
object/property state.

If visual confirmation inside the Material Editor is necessary,
reopening the Material Editor may refresh its displayed state.


**15. Basic Knowledge Is Not a Tool Catalogue**

This document provides stable conceptual knowledge.

It does not define all available Unreal MCP tools.

Absence of a tool, operation, property, or workflow from this document
does not mean that Unreal MCP cannot perform it.

Use:

Basic Knowledge

→ understand Unreal concepts


Verified Recipe

→ follow previously successful workflows


Live MCP

→ determine current tools, schemas, arguments, and actual state

Live MCP information has priority when current implementation details
are required.
