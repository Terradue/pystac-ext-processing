<!--
Copyright 2026 Terradue

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Scope, architecture, and validation

## Package structure

The distribution is `pystac-ext-processing`; its import path is `pystac.extensions.processing`. The wheel excludes the shared namespace initializer owned by PySTAC.

`ProcessingExtension.ext()` selects a wrapper for an Item, Asset, or ItemAssetDefinition. Wrappers modify the object's dictionaries directly. PySTAC's normal serialization includes those fields. The package stores metadata and does not execute processing expressions.

Asset wrappers capture an Item owner's property dictionary as a fallback when constructed. Provider wrappers are independent of extension membership. Collection summaries use a separate helper with list and schema-dictionary accessors.

## Specification scope

The [Processing specification](https://github.com/stac-extensions/processing) places Collection processing metadata on producer or processor Providers, summaries, Assets, or item asset definitions, rather than directly on the Collection root. Item fields belong in properties and may also appear on Assets.

`processing:version` identifies the main processing chain; `processing:software` records individual tools. These differ from metadata versioning. Processing time also differs from an Item's metadata creation time. See the specification for the detailed conventions.

## Validation boundaries

The implementation declares the [v1.2.0 schema](https://stac-extensions.github.io/processing/v1.2.0/schema.json). That schema requires at least one processing field in Item properties; Asset-only fields do not satisfy its Item requirement. A Collection needs a processing field in an allowed location. Provider-based satisfaction requires a producer or processor role. Summary values are not validated by the extension schema.

The wrappers do not perform these schema checks. Setters store values without enforcing all annotated types or expression formats. Datetime accessors use PySTAC's conversion utilities; summary datetime accessors leave list values unchanged. `ProcessingExpression` checks required members when read, not when constructed.

`apply()` updates fields sequentially and clears omitted values. It is not transactional: if an assignment fails, earlier changes remain. `to_dict()` serializes without schema validation.

For full validation, install `pystac[validation]` and call `item.validate()` or `collection.validate()`. Remote schema access or an application-configured validator is needed. Validate summary contents separately where your application requires it.

## Migration hooks

`PROCESSING_EXTENSION_HOOKS` exposes the schema URI, the legacy identifier `processing`, and the supported Item/Collection object types. This module does not automatically register its hook object with PySTAC. Importing the wrapper alone is not a migration workflow.
