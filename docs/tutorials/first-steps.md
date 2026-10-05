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

# Create an Item with processing metadata

[Install the package](../how-to/install.md), then run this complete example:

```python
from datetime import datetime, timezone

import pystac
from pystac.extensions.processing import ProcessingExpression, ProcessingExtension

acquired_at = datetime(2024, 1, 1, tzinfo=timezone.utc)
processed_at = datetime(2024, 1, 2, 12, tzinfo=timezone.utc)
item = pystac.Item(
    id="example-product",
    geometry=None,
    bbox=None,
    datetime=acquired_at,
    properties={},
)
processing = ProcessingExtension.ext(item, add_if_missing=True)
processing.apply(
    expression=ProcessingExpression.create(format="gdal-calc", expression="A * 2"),
    lineage="Scaled the source raster by two.",
    level="L2",
    facility="Example processing centre",
    software={"example-processor": "1.0.0"},
    version="1.0.0",
    processing_datetime=processed_at,
)

serialized = item.to_dict()
assert ProcessingExtension.get_schema_uri() in serialized["stac_extensions"]
assert serialized["properties"]["processing:level"] == "L2"
assert serialized["properties"]["processing:datetime"] == "2024-01-02T12:00:00Z"

restored = ProcessingExtension.ext(pystac.Item.from_dict(serialized))
assert restored.processing_datetime == processed_at
assert restored.expression is not None
assert restored.expression.format == "gdal-calc"
assert restored.expression.expression == "A * 2"
```

The wrapper records the expression; it does not run a processor. The Item's acquisition timestamp and its processing timestamp are independent. Supply timezone-aware UTC datetimes for processing metadata.

Serialization does not validate JSON Schema. See [validation boundaries](../explanation/architecture.md#validation-boundaries).
