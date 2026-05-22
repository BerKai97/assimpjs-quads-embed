# assimpjs-quads-embed

Custom build of [assimpjs](https://github.com/repalash/assimpjs) (Emscripten port of [Assimp](https://github.com/assimp/assimp)) with a one-token wrapper patch.

## The patch

In `assimpjs/src/assimpjs.cpp`, the importer post-process flag-OR contains
`aiProcess_Triangulate | …`. This build swaps that token for
`aiProcess_EmbedTextures | …`. Two effects in one change:

1. **Quads survive.** Removing `aiProcess_Triangulate` lets faces with >3 vertices
   round-trip through `aiScene` to the FBX exporter as real polygons (FBX
   `PolygonVertexIndex` uses one's-complement-of-last-index to encode polygon
   boundaries, so quads are first-class).
2. **Textures embed.** Adding `aiProcess_EmbedTextures` invokes Assimp's
   `EmbedTexturesProcess` at import time. It reads texture files through the
   wrapper's `FileListIOSystemReadAdapter` (so any PNG/JPG you pass in the
   `FileList` is eligible), populates `aiScene->mTextures` with their bytes, and
   rewrites material texture paths to `*N` form. The `FBXExporter` then sees
   embedded textures and emits `Content:` base64 blobs inside Video nodes, so
   the exported FBX is fully self-contained.

## Files

| File | Size | Notes |
|---|---|---|
| `assimpjs.wasm` | 4,374,756 bytes | Built with `emscripten/emsdk:3.1.56` against the repalash/assimpjs `main` branch source, Assimp submodule pinned at `cf7d363`. |

The matching JS glue (`assimpjs.js`) is unchanged from upstream — load it from
`https://cdn.jsdelivr.net/gh/repalash/assimpjs@main/dist/assimpjs.js` and pass
the `wasmBinary` option when instantiating.

## CDN

Once this repo is public, jsdelivr serves the file directly:

```
https://cdn.jsdelivr.net/gh/BerKai97/assimpjs-quads-embed@main/assimpjs.wasm
```

## License & attribution

The WASM binary contains code from Assimp (BSD-3-Clause, © 2006–2024 Assimp
contributors) and assimpjs (MIT, © repalash). Their licenses apply to the
respective portions. The one-token patch is in the public domain.
