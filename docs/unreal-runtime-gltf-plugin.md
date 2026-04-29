# Unreal Runtime glTF Loader Plugin (Starter Spec + Scaffold)

This document defines an implementation-ready blueprint for an open-source Unreal Engine plugin that loads assets at runtime from glTF 2.0 sources.

## Goal

Build a modular Unreal plugin that can import and construct, at runtime and in-editor:

- Static Meshes
- Skeletal Meshes
- Skeletal Animations
- Curve Animations
- Materials
- Textures

Sources:

- Local filesystem
- HTTP/HTTPS URLs
- Command output pipelines
- Clipboard text/binary
- Raw JSON strings

Formats:

- glTF 2.0 (.gltf + external resources)
- Binary glTF (.glb)
- Embedded resources (base64)

## Proposed Plugin Name

`RuntimeAssetBridge`

## High-Level Architecture

### Core Modules

1. `RuntimeAssetBridge` (Runtime)
   - Public C++ APIs and Blueprint libraries
   - Asset construction and object lifecycle
   - Async task orchestration

2. `RuntimeAssetBridgeGLTF` (Runtime)
   - glTF parser adapter
   - Buffer/image/material/mesh translators
   - Skeletal/animation conversion

3. `RuntimeAssetBridgeEditor` (Editor, optional)
   - Testing utilities
   - Validation and inspection tools

### Internal Layers

- **Source Layer**: `IAssetSourceProvider`
  - `FileSourceProvider`
  - `HttpSourceProvider`
  - `CommandSourceProvider`
  - `ClipboardSourceProvider`
  - `MemorySourceProvider`

- **Decode Layer**: `IFormatDecoder`
  - `GLTF2Decoder`

- **Build Layer**: Unreal object builders
  - `UStaticMeshBuilder`
  - `USkeletalMeshBuilder`
  - `UMaterialBuilder`
  - `UTextureBuilder`
  - `UAnimationBuilder`

- **Post-Process Layer**
  - skeleton hierarchy edits
  - LOD generation hooks
  - collision generation hooks

## Minimum Public API (C++)

```cpp
struct FRABLoadRequest
{
    FString SourceUri;
    bool bBuildStaticMeshes = true;
    bool bBuildSkeletalMeshes = true;
    bool bBuildMaterials = true;
    bool bBuildTextures = true;
    bool bBuildAnimations = true;
    bool bAsync = true;
};

struct FRABLoadResult
{
    TArray<TObjectPtr<UObject>> CreatedAssets;
    TArray<FString> Warnings;
    TArray<FString> Errors;
    bool bSuccess = false;
};

DECLARE_DELEGATE_OneParam(FRABOnLoadCompleted, const FRABLoadResult&);

class IRABRuntimeLoader
{
public:
    virtual ~IRABRuntimeLoader() = default;
    virtual void Load(const FRABLoadRequest& Request, FRABOnLoadCompleted Callback) = 0;
};
```

## Minimum Blueprint API

`URABBlueprintLibrary`:

- `LoadFromFile`
- `LoadFromUrl`
- `LoadFromJsonString`
- `LoadFromClipboard`
- `LoadFromCommand`
- `GetCreatedStaticMeshes`
- `GetCreatedSkeletalMeshes`
- `GetCreatedAnimations`
- `ModifySkeletonHierarchy`

All async loads should expose latent/action or async node variants.

## glTF Support Scope

### Required

- Accessors, bufferViews, buffers
- POSITION/NORMAL/TANGENT/TEXCOORD/COLOR/JOINTS/WEIGHTS
- Indices: 16/32-bit
- PBR Metallic-Roughness base workflow
- BaseColor/Normal/ORM/Emissive textures
- Node transforms and parent-child hierarchy
- Skin + inverse bind matrices
- Animation channels: translation/rotation/scale + morph weights (optional v1)

### Optional (Phase 2)

- KHR_materials_unlit
- KHR_texture_transform
- KHR_materials_clearcoat
- Draco / Meshopt extensions

## Runtime Safety / Performance Requirements

- No blocking game thread during source IO and parse.
- Build heavy buffers off-thread; marshal UObject creation safely to game thread.
- Cache decoded textures and deduplicate materials.
- Provide cancellation tokens for active jobs.
- Emit detailed logs with source offsets and glTF path traces.

## Packaging + Distribution

- Open-source under MIT in GitHub repo.
- Marketplace package with prebuilt binaries and sample content.
- Keep extension loaders in separate optional modules:
  - `RuntimeAssetBridgeOBJ`
  - `RuntimeAssetBridgeSTL`
  - `RuntimeAssetBridgeFBX`
  - `RuntimeAssetBridgeVoxel`

## Testing Matrix

- UE versions: 5.2, 5.3, 5.4+
- Platforms: Win64, Linux, Android (optional), consoles as allowed
- Cases:
  - static-only glb
  - skeletal + animations gltf
  - embedded textures
  - malformed JSON
  - network timeout/retry

## Suggested Initial Milestones

1. Scaffold plugin + module boundaries.
2. Implement file + memory source providers.
3. Parse glTF static meshes + textures + basic materials.
4. Add skeletal meshes + animation import.
5. Expose Blueprint async nodes.
6. Add HTTP/clipboard/command providers.
7. Add editor test harness + sample map.

## Starter File Layout

```text
Plugins/RuntimeAssetBridge/
  RuntimeAssetBridge.uplugin
  Source/
    RuntimeAssetBridge/
    RuntimeAssetBridgeGLTF/
    RuntimeAssetBridgeEditor/
```

---

If you want, the next step is generating the full plugin scaffold files (`.uplugin`, `Build.cs`, module classes, Blueprint nodes, async loader skeletons) so you can drop directly into an Unreal project.
