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

# Work with processing metadata

Run the [tutorial](../tutorials/first-steps.md) first. The examples below continue with its imports, Item, and timestamps, in order.

## Update or remove a field

```python
processing.level = "L3"
processing.lineage = None
assert "processing:lineage" not in item.properties
```

`ProcessingExtension.ext(item)` requires extension membership. Pass `add_if_missing=True` to add it. Use individual setters to preserve other values; `apply()` clears omitted fields.

## Attach Asset fields

```python
asset = pystac.Asset(href="https://example.org/product.tif", media_type="image/tiff")
item.add_asset("data", asset)
asset_processing = ProcessingExtension.ext(asset)
assert asset_processing.level == "L3"
asset_processing.level = "L2"
assert asset.extra_fields["processing:level"] == "L2"
assert processing.level == "L3"
asset_processing.level = None
assert asset_processing.level == "L3"
```

Attach Assets to their owner before wrapping them. Missing Asset fields fall back to an Item owner's properties; writes affect only the Asset. Extension membership is checked or added on the owner.

## Manage Collection summaries

```python
collection = pystac.Collection(
    id="example-collection",
    description="Example processed products",
    extent=pystac.Extent(
        pystac.SpatialExtent([[-180.0, -90.0, 180.0, 90.0]]),
        pystac.TemporalExtent([[acquired_at, None]]),
    ),
    license="proprietary",
)
summaries = ProcessingExtension.summaries(collection, add_if_missing=True)
summaries.level = ["L2", "L3"]
summaries.processing_datetime = ["2024-01-02T12:00:00Z"]
summaries.software = {
    "type": "object",
    "properties": {"example-processor": {"type": "string"}},
}
assert summaries.level == ["L2", "L3"]
assert summaries.processing_datetime == ["2024-01-02T12:00:00Z"]
summaries.software = None
assert summaries.software is None
```

Summary accessors read and replace whole lists or schema dictionaries. They do not aggregate Items or offer `apply()`. Passing a Collection to `ProcessingExtension.ext()` raises `pystac.ExtensionTypeError`.

## Use Collection item asset definitions

```python
collection.item_assets = {
    "data": pystac.ItemAssetDefinition({"type": "image/tiff", "roles": ["data"]}),
}
definition = collection.item_assets["data"]
definition_processing = ProcessingExtension.ext(definition, add_if_missing=True)
definition_processing.level = "L2"
assert definition.properties["processing:level"] == "L2"
```

Retrieve definitions through `collection.item_assets` so PySTAC attaches their Collection owner.

## Describe a processing Provider

```python
provider = pystac.Provider(
    name="Example processing centre",
    roles=[pystac.ProviderRole.PROCESSOR],
)
provider_processing = ProcessingExtension.provider(provider)
provider_processing.apply(level="L2", software={"example-processor": "1.0.0"})
collection.providers = [provider]
ProcessingExtension.add_to(collection)
assert provider.extra_fields["processing:level"] == "L2"
```

The Provider wrapper writes `extra_fields`. It does not check roles or manage extension membership; declare the extension on the Collection separately.

## Link provenance resources

Use ordinary PySTAC links for the [specification's relation types](https://github.com/stac-extensions/processing#relation-types): `derived_from` for inputs, `processing-expression` for a script, `processing-execution` for a run, `processing-software` for dependencies, and `processing-validation` for validation results.

```python
item.add_link(pystac.Link(
    rel="derived_from",
    target="https://example.org/source-item.json",
    media_type="application/geo+json",
))
item.add_link(pystac.Link(
    rel="processing-expression",
    target="https://example.org/process.py",
    media_type="text/plain",
))
```
