# unreal-mcp-local-llm
Resources for the “Local LLMs Meet Unreal MCP” article series, including prompts, UE knowledge, and verified recipes.

# Unreal MCP + Local LLM Resources

Resources for using Unreal Engine 5.8 Unreal MCP with local LLMs.

This repository contains System Prompts, Unreal Engine basic knowledge, and verified workflow recipes used in the **Local LLMs Meet Unreal MCP** article series.

The goal is to reduce unnecessary Tool Search, repeated trial and error, and unstable execution when controlling Unreal Engine through Unreal MCP with smaller local LLMs.

## Contents

### `system_prompt.md`

A System Prompt for guiding a local LLM when working with Unreal MCP.

It defines general behavior such as:

* how to use Tool Search,
* how to handle Tool errors,
* how to reuse resolved references,
* how to avoid unnecessary exploration,
* how to verify the actual Unreal Engine state before reporting success.

This file is intended to define **how the AI should work**, rather than how to perform one specific Unreal Engine task.

---

### `ue_basic_knowledge.md`

Basic Unreal Engine knowledge intended to be supplied through RAG.

It contains stable information such as:

* Unreal Engine coordinate conventions,
* units and rotation values,
* differences between Actors, Components, and Assets,
* Actor labels and object references,
* Static Mesh Asset paths,
* Material and Component relationships.

This file is intended to provide **general Unreal Engine knowledge** without turning the System Prompt into a large reference document.

---

### `recipes/`

Verified workflows for specific Unreal MCP operations.

Recipes are based on procedures that succeeded during actual testing.

Unlike the System Prompt and UE Basic Knowledge, Recipes describe **how a particular type of task can be completed successfully**.

Current Recipe:

* `change_staticmesh_actor_materialcolor.md`

  * Change the visible Material color of one or more StaticMeshActors.
  * Create or reuse Actor-specific Materials.
  * Avoid changing shared Static Mesh default Materials.
  * Reuse successful Tool paths for repeated operations.
  * Includes single-Actor and multiple-Actor workflows.

## How to Use

These files were tested with a setup using:

* Unreal Engine 5.8
* Unreal MCP
* LM Studio
* Localito Buddy
* Local LLMs such as Qwen3.5-9B

A typical setup is:

1. Add the contents of `system_prompt.md` to the AI client's System Prompt or Meta Prompt.
2. Load `ue_basic_knowledge.md` through RAG.
3. Load a matching Recipe when performing a more specific Unreal Engine operation.
4. Connect the AI client to Unreal MCP and run the task.

When using Localito Buddy, both UE Basic Knowledge and Recipe files can be loaded as chat attachments and used as RAG context.

## Why Recipes?

Unreal MCP uses Tool Search to discover and execute Unreal Engine operations.

This helps reduce the amount of Tool schema initially loaded into the model context, but smaller local LLMs may spend significant time exploring Toolsets, correcting arguments, or retrying failed Tool calls.

Recipes provide previously verified execution paths so the model does not have to rediscover the same successful workflow every time.

Recipes do not replace Unreal MCP Tool Search.

The live MCP registry and Tool schemas remain authoritative. Recipes are intended to guide the model toward a known successful path while still allowing adaptation when Unreal MCP changes.

## Tested Results

During testing, adding verified Recipes reduced the amount of trial and error required for Material operations.

For example:

* changing the color of a single Cube became substantially faster and more consistent,
* multiple Cube Actors could be processed using the same successful workflow,
* after the first Actor established the Tool path, the model was able to reuse that pattern for subsequent Actors.

Results may vary depending on the model, hardware, prompt, Unreal Engine version, and Unreal MCP implementation.

## Article Series

These files accompany the **Local LLMs Meet Unreal MCP** article series.

The series covers:

* connecting a local LLM to Unreal Engine 5.8 through Unreal MCP,
* reducing Tool Search confusion with System Prompts and RAG,
* reusing successful workflows through Recipes,
* sharing one LM Studio server between multiple Unreal Engine machines.

Article links will be added here as they are published.

Medium (English)
Local LLMs Meet Unreal MCP Part 1
Can a Local LLM Control Unreal Engine 5.8 Through MCP?
https://medium.com/@murata_90507/local-llms-meet-unreal-mcp-part-1-9b652fed0f75

## Status

This repository is based on experimental testing with Unreal Engine 5.8 Unreal MCP.

Unreal MCP is still evolving, so Tool names, schemas, or behavior may change in future Unreal Engine versions.

When a Recipe differs from the live MCP schema, follow the live MCP schema and adapt the Recipe while keeping its workflow intent.

## License

The resources in this repository are released under the MIT License.
