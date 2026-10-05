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

# Processing fields

These are the seven fields exposed by the current implementation. The [v1.2.0 schema](https://stac-extensions.github.io/processing/v1.2.0/schema.json) defines their JSON representation.

| STAC field | Python property | Value |
| --- | --- | --- |
| `processing:expression` | `expression` | `ProcessingExpression` or a dictionary when assigning; a wrapper when reading. |
| `processing:lineage` | `lineage` | Text describing the processing history. |
| `processing:level` | `level` | A short level name, such as `L2A`. |
| `processing:facility` | `facility` | Facility name. |
| `processing:software` | `software` | Dictionary of software names and version strings. |
| `processing:version` | `version` | Primary processor or chain version. |
| `processing:datetime` | `processing_datetime` | Python `datetime`, serialized as a string. |

These accessors are shared by the Item, Asset, item asset definition, and Provider wrappers. Absent fields read as `None`, subject to the Asset fallback described below. Assigning `None` removes the local field. `apply()` assigns every field, so omitted arguments clear existing local values.

## Expressions

`ProcessingExpression.create(format=..., expression=...)` constructs an expression wrapper. Both members are required by the schema; the payload type depends on the format. The wrapper accepts arbitrary payloads without interpreting or executing them.

`ProcessingExpression.to_dict()` returns its underlying dictionary, without copying it. Assigning a wrapper to the extension stores that dictionary. Missing required members raise `pystac.RequiredPropertyMissing` when accessed; constructing a wrapper does not validate its contents.

## Storage and inheritance

| Wrapper | Storage |
| --- | --- |
| `ItemProcessingExtension` | `item.properties` |
| `AssetProcessingExtension` | `asset.extra_fields` |
| `ItemAssetsProcessingExtension` | `item_asset.properties` |
| `ProviderProcessingExtension` | `provider.extra_fields` |

An Asset wrapper created with an Item owner falls back to that Item's properties when a field is absent locally. Writes always target the Asset. Removing an override exposes the Item value again. Collection-owned Assets have no such fallback. Item asset definitions do not inherit Collection fields.

## Collection summaries

`ProcessingExtension.summaries(collection)` returns `SummariesProcessingExtension`:

| Properties | Representation |
| --- | --- |
| `lineage`, `level`, `facility`, `version`, `processing_datetime` | Lists, returned in full. |
| `expression`, `software` | Dictionaries accessed through PySTAC's schema-summary API. |

Summary datetime values are strings: this helper does not convert them to or from Python datetimes. Dictionary summaries describe possible values using JSON Schema; they are not ordinary Item field values. Setters replace the whole summary, and `None` removes it. There is no summary `apply()` method or automatic aggregation of Items.

See [the guide](../how-to/use-extension.md) for examples and [validation boundaries](../explanation/architecture.md#validation-boundaries) for limitations.
