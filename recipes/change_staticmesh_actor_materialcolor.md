# Unreal MCP Verified Recipe: Change Static Mesh Actor Material Color

A verified Unreal MCP workflow for changing the visible Material color of one or more `StaticMeshActor` instances without unintentionally modifying shared Static Mesh assets.

This Recipe is intended to guide a local LLM toward a previously successful execution path while still allowing adaptation to the current live MCP schema.

---

Recipe ID: UERCP-MAT-001

Version: 1.0

Target: Unreal Engine 5.8

Status: Verified

Purpose: Change the visible material color of one or more StaticMeshActors while avoiding unintended changes to other Actors that share the same Static Mesh asset.

___

**1. User Intent**

Typical requests include:

- Make this Actor red.
- Change MCP_Test_Cube to blue.
- Change only this Cube\'s color.
- Make MCP_Test_Cube green without affecting other Actors.
- Create a new Material for this Actor and make it red.
- Use an existing specified Material.
- Give Cube_1 through Cube_12 individual new Materials and assign
  different colors.
- Randomly color several existing Cube Actors.

The user should not need to provide:

- MCP tool names
- toolset names
- Actor refPaths
- Component refPaths
- Material object refPaths
- Material Expression refPaths
- Material graph implementation details

Resolve these internally when necessary.

\

**2. Core Principle**

When only a specific Actor should change appearance:

- do not modify the shared Static Mesh Asset\'s default Material,
- do not modify a shared Engine Material,
- prefer an Actor-specific Material assigned through the Actor\'s
  StaticMeshComponent Material override.

Conceptually:

StaticMeshActor

    ↓

StaticMeshComponent

    ├─ Actor-specific Material Override

    │

    └─ StaticMesh

          ↓

       Shared Static Mesh Asset

          ↓

       Default Material

A change intended for one Actor should normally be performed at the
Actor/Component-specific level.

Live MCP schema is authoritative for exact current tools and argument
structures.

\

**3. Material Reuse Policy**

Material reuse should be predictable and should not require an expensive
project-wide dependency search.

**Case A: User explicitly requests a new Material**

Create a new Material.

Examples:

Create a new Material for MCP_Test_Cube and make it red.

Do not reuse the current Material.

Do not reuse an existing Material in this case.

\

**Case B: User explicitly specifies an existing Material**

Resolve and use that Material.

Example:

Use M_MyMaterial on MCP_Test_Cube.

Resolve the user-visible name to the object reference required by MCP.

\

**Case C: User requests only a color change**

Example:

Make MCP_Test_Cube blue.

Inspect the Actor\'s existing Material override.

If a suitable Actor-specific editable Material already exists, prefer
reusing it.

If no suitable Actor-specific Material exists, create one.

Do not perform a project-wide search solely to prove that the Material
is unused by all other Actors.

\

**Main Reason for Reuse**

Material reuse primarily prevents unnecessary Asset proliferation.

Do not assume that Material reuse will always make execution faster.

Tool discovery, schema correction, verification, and MCP calls may cost
more time than Material creation itself.

\

**4. Actor-Specific Material Naming**

When automatically creating a Material for one Actor, prefer an
Actor-oriented generic name.

Recommended:

M\_\<ActorName\>\_Color

Example:

Actor:

MCP_Test_Cube

\

Material:

M_MCP_Test_Cube_Color

Full object refPath example:

/Game/Materials/M_MCP_Test_Cube_Color.M_MCP_Test_Cube_Color

For multiple Actors:

Cube_1  → M_Cube_1_Color

Cube_2  → M_Cube_2_Color

Cube_3  → M_Cube_3_Color

\...

Avoid automatically creating color-specific names such as:

M_RedColor_Mat

M_BlueColor_Mat

M_GreenColor_Mat

for repeated color changes to the same Actor.

The Material name should preferably describe its ownership or role
rather than its current color.

\

**5. Color Values**

Common normalized RGB values:

Red      = R 1.0, G 0.0, B 0.0

Green    = R 0.0, G 1.0, B 0.0

Blue     = R 0.0, G 0.0, B 1.0

Yellow   = R 1.0, G 1.0, B 0.0

Cyan     = R 0.0, G 1.0, B 1.0

Magenta  = R 1.0, G 0.0, B 1.0

White    = R 1.0, G 1.0, B 1.0

Black    = R 0.0, G 0.0, B 0.0

Do not confuse:

Blue = 0,0,1

with:

Cyan = 0,1,1

For random colors, choose valid normalized RGB values that provide
visually distinguishable results unless the user requests another
scheme.

Alpha should normally remain:

A = 1.0

for an opaque color unless the task requires transparency.

\

**6. Single-Actor Workflow**

**Step 1: Resolve the Actor**

Resolve the Actor from its user-visible Level name or label.

Verified tool direction:

Toolset:

editor_toolset.toolsets.scene.SceneTools

Tool:

find_actors

Example:

MCP_Test_Cube

→ /Game/Main.Main:PersistentLevel.StaticMeshActor_2

Do not manually construct an Actor refPath.

Avoid providing a root unless a valid Actor root is specifically needed.

Do not treat PersistentLevel itself as an Actor root.

Preserve the resolved Actor refPath.

\

**Step 2: Resolve the StaticMeshComponent**

Verified tool direction:

Toolset:

editor_toolset.toolsets.actor.ActorTools

Tool:

get_components

When appropriate, filter by:

/Script/Engine.StaticMeshComponent

Example:

/Game/Main.Main:PersistentLevel.StaticMeshActor_2.StaticMeshComponent0

Do not construct Component refPaths by appending guessed names to the
Actor refPath.

Preserve the returned Component refPath.

\

**Step 3: Inspect Existing Material Override**

Verified tool direction:

Toolset:

editor_toolset.toolsets.object.ObjectTools

Tool:

get_properties

Inspect:

OverrideMaterials

Example without an override:

{

  \"OverrideMaterials\": \[\"None\"\]

}

Example with an existing Actor-specific Material:

{

  \"OverrideMaterials\": \[

    {

      \"refPath\":
\"/Game/Materials/M_MCP_Test_Cube_Color.M_MCP_Test_Cube_Color\"

    }

  \]

}

Use this information to decide whether to reuse or create a Material.

\

**7. Create a Material When Needed**

Verified tool direction:

Toolset:

editor_toolset.toolsets.material.MaterialTools

Tool:

create_material

Recommended destination:

/Game/Materials

Recommended name:

M\_\<ActorName\>\_Color

Example result:

/Game/Materials/M_MCP_Test_Cube_Color.M_MCP_Test_Cube_Color

Preserve the complete returned Material refPath.

\

**8. Find or Create the Color Expression**

For an existing Actor-specific Material, inspect its current Expressions
before creating duplicates.

Verified tool:

get_expressions

Look for an existing:

/Script/Engine.MaterialExpressionConstant3Vector

If a suitable Constant3Vector already exists, reuse it.

Otherwise add one.

Verified tool:

add_expression

Expression class:

/Script/Engine.MaterialExpressionConstant3Vector

Example Expression refPath:

/Game/Materials/M_MCP_Test_Cube_Color.M_MCP_Test_Cube_Color:MaterialExpressionConstant3Vector_0

Preserve the Expression refPath.

\

**9. Set the Color**

Verified tool direction:

Toolset:

editor_toolset.toolsets.object.ObjectTools

Tool:

set_properties

The tested Constant3Vector.constant property is represented as an
FLinearColor-style structure.

Example red:

{

  \"constant\": {

    \"r\": 1.0,

    \"g\": 0.0,

    \"b\": 0.0,

    \"a\": 1.0

  }

}

Example green:

{

  \"constant\": {

    \"r\": 0.0,

    \"g\": 1.0,

    \"b\": 0.0,

    \"a\": 1.0

  }

}

Example blue:

{

  \"constant\": {

    \"r\": 0.0,

    \"g\": 0.0,

    \"b\": 1.0,

    \"a\": 1.0

  }

}

When required by the live schema, set the complete nested structure.

Do not reinterpret the color as X/Y/Z simply because both use numeric
components.

\

**10. Connect the Color to Base Color**

For a newly created simple color Material, this is required.

Verified tool direction:

Toolset:

editor_toolset.toolsets.material.MaterialTools

Tool:

connect_to_output

Use the Constant3Vector Expression as the source.

Typical values:

expression        = Constant3Vector refPath

output_name       = \"\"

material_property = MP_BaseColor

Do not attempt to set MP_BaseColor directly on the Material using
ObjectTools.set_properties.

In this workflow, MP_BaseColor is a Material graph connection.

If connect_to_output fails, the task is not complete.

Read the live schema, correct the call, and retry.

\

**11. Recompile the Material**

Verified tool:

recompile

Recompile after modifying the Material graph or its color Expression.

\

**12. Verify Base Color Connection**

Verified tool:

get_property_input

Inspect:

MP_BaseColor

The property should be driven by the intended Expression.

A Material graph connection error must not be ignored.

Do not report success solely because the Material Asset exists.

\

**13. Apply the Material to the Actor**

If the intended Material is not already assigned, set the
StaticMeshComponent Material override.

Verified direction:

Toolset:

editor_toolset.toolsets.object.ObjectTools

Tool:

set_properties

Set:

OverrideMaterials\[0\]

to the complete Material object refPath.

Example:

/Game/Materials/M_MCP_Test_Cube_Color.M_MCP_Test_Cube_Color

Do not modify the shared Static Mesh Asset\'s default Material merely to
affect one Actor.

\

**14. Verify the Actor Material**

Inspect:

OverrideMaterials

Example:

{

  \"OverrideMaterials\": \[

    {

      \"refPath\":
\"/Game/Materials/M_MCP_Test_Cube_Color.M_MCP_Test_Cube_Color\"

    }

  \]

}

When useful, also verify the color Expression value.

A successful property-setting call alone is not sufficient if the
resulting Actor state can be verified directly.

\

**15. Multiple-Actor Workflow**

For multiple Actors, process one Actor completely before moving to the
next.

Preferred pattern:

Cube_1

→ resolve Actor

→ resolve StaticMeshComponent

→ create/reuse Material

→ create/reuse Constant3Vector

→ set color

→ connect BaseColor

→ recompile

→ assign OverrideMaterials

→ verify

→ complete

\

Cube_2

→ same complete workflow

\

Cube_3

→ same complete workflow

\

\...

Do not unnecessarily use this pattern:

create all Materials

→ create all Expressions

→ set all colors

→ connect all Base Colors

→ assign all Materials

when it increases:

- unresolved reference state,
- long-lived intermediate objects,
- nested tool-call complexity,
- malformed JSON risk,
- recovery difficulty.

The verified multi-Actor workflow completed 12 Cube Actors without
Unreal warnings after switching to one-Actor-at-a-time execution.

\

**16. Reuse the Verified Tool Path**

For repetitive Actors:

The first one or two Actors may require schema confirmation or setup.

Once the workflow has succeeded:

- reuse the same verified tool path,
- reuse known schemas,
- change only Actor-specific refs, Material names, Expression refs, and
  color values,
- do not rediscover the same Toolset structure for every Actor.

This can make later repeated operations substantially more direct.

\

**17. Failure Recovery for Multiple Actors**

If one Actor fails:

completed Actors

→ keep them unchanged

\

current Actor

→ diagnose and repair

\

remaining Actors

→ continue afterward

Do not restart all previously completed Actors.

If an intermediate Asset was already successfully created, reuse it
rather than creating a duplicate.

If an Expression already exists, inspect and reuse it when appropriate.

\

**18. Tool-Format / Malformed JSON Recovery**

A verified failure mode during multi-Actor processing was:

tool_format_generation_error

with an error such as:

Unterminated string in JSON

This means the generated Tool Call JSON was malformed before the Unreal
MCP operation could execute correctly.

If this occurs:

1.  preserve completed Actors,
2.  preserve already-created Materials and Expressions,
3.  regenerate only the failed tool call,
4.  use the minimum required arguments,
5.  ensure nested JSON strings are properly closed and escaped,
6.  continue from the current unfinished Actor.

Do not interpret a malformed Tool Call JSON error as proof that Unreal
MCP lacks the requested capability.

\

**19. Avoid Broad Property Enumeration When the Property Is Known**

A tested failure mode occurred when attempting broad property listing on
complex Unreal objects.

This produced large numbers of warnings related to delegate/event
properties.

Therefore, when the required property name is already known, prefer
targeted property access.

Examples include:

OverrideMaterials

StaticMesh

StaticMaterials

constant

Do not enumerate every property merely to confirm a property that is
already known.

\

**20. Known Default Cube Material Chain**

The built-in Cube used during verification:

/Engine/BasicShapes/Cube.Cube

Default Material:

/Engine/EngineMaterials/WorldGridMaterial.WorldGridMaterial

The tested WorldGridMaterial returned no exposed Material Instance
parameters:

{

  \"returnValue\": \[\]

}

Therefore, do not assume that creating a Material Instance directly from
WorldGridMaterial will expose a color parameter suitable for this task.

A verified path is:

custom Material

→ Constant3Vector

→ MP_BaseColor

→ Actor-specific OverrideMaterials

\

**21. Material Editor UI Refresh**

An already-open Material Editor window may display stale state after an
MCP-driven change.

The underlying Material may already have changed correctly.

Prefer live Unreal property/state verification.

If visual graph inspection is required, reopening the Material Editor
may refresh the display.

Do not classify a successful MCP modification as failed solely because
an already-open editor window has not refreshed.

\

**22. Avoid These Failed or Inefficient Paths**

Do not:

- assume that a dedicated set_material tool is required,
- conclude Material assignment is impossible because set_material is
  absent,
- modify shared StaticMesh StaticMaterials when only one Actor should
  change,
- directly set MP_BaseColor on the Material with
  ObjectTools.set_properties,
- report success after connect_to_output fails,
- confuse Blue (0,0,1) with Cyan (0,1,1),
- manually construct Component refPaths,
- repeatedly perform broad list_properties calls when specific
  properties are known,
- assume WorldGridMaterial exposes a usable color parameter,
- create a new color-specific Material for every color change when an
  Actor-specific reusable Material already exists,
- perform expensive project-wide Material usage analysis unless the user
  requests it,
- forget valid Actor, Component, Material, or Expression references
  already obtained during the task,
- treat Content Browser display names as complete object refPaths,
- perform all first stages across many Actors when one-Actor-at-a-time
  execution is more robust,
- restart all completed Actors because one later Actor failed,
- treat malformed tool-call JSON as an Unreal MCP capability failure.

\

**23. Recommended Repeated Color Lifecycle**

For the first color change:

MCP_Test_Cube

→ create M_MCP_Test_Cube_Color

→ add Constant3Vector

→ set requested RGB

→ connect to MP_BaseColor

→ recompile

→ assign OverrideMaterials\[0\]

→ verify

For later color changes:

MCP_Test_Cube

→ reuse M_MCP_Test_Cube_Color

→ reuse existing Constant3Vector

→ change RGB

→ recompile

→ verify BaseColor

→ verify OverrideMaterials

The primary benefit of reuse is avoiding unnecessary Material
proliferation.

\

**24. Completion Criteria**

For a single Actor, report success only when the required relevant
conditions are satisfied:

- target Actor resolved,
- target StaticMeshComponent resolved,
- correct Material selected or created,
- requested color applied,
- required Expression exists,
- Expression is connected to MP_BaseColor,
- Material recompilation completed,
- Actor Material override points to the intended Material,
- shared Static Mesh default Material was not unintentionally changed.

For multiple Actors:

- each requested Actor must independently satisfy the required workflow,
- one completed Actor does not imply the remaining Actors are complete,
- incomplete or failed Actors must be reported accurately.

Do not report success merely because all Assets were created.

The requested visible/resulting state must be established.

\

**25. Authority and Adaptation**

This Recipe records a workflow that succeeded in actual Unreal MCP
testing.

Treat it as a strong execution guide for matching tasks.

However:

- live MCP registry is authoritative for current tool availability,
- live schemas are authoritative for argument structure,
- actual Unreal state is authoritative for verification.

If a future Unreal MCP version changes a tool schema:

keep the workflow intent

→ adapt the specific tool call to the live schema

Do not discard the entire Recipe merely because one implementation
detail changes.

\

**Verified Results**

This workflow has been successfully used for:

- changing one Cube Actor to red,
- changing one Cube Actor to green,
- reusing an Actor-specific Material to change its color,
- correcting an existing Material from an incorrect color to blue,
- assigning Actor-specific Materials without modifying the shared Cube
  Static Mesh,
- creating and applying individual Materials to Cube_1 through Cube_12,
- completing a 12-Actor color assignment workflow without Unreal
  warnings after switching to one-Actor-at-a-time processing.

The Recipe is intended to preserve these successful execution patterns
while allowing live MCP schema correction when necessary.

