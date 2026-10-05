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

# Processing PySTAC extension

`pystac-ext-processing` provides accessors for processing metadata on PySTAC objects. Import its API from `pystac.extensions.processing`.

The code targets the [Processing extension specification](https://github.com/stac-extensions/processing) and declares its [v1.2.0 schema](https://stac-extensions.github.io/processing/v1.2.0/schema.json). The Python distribution version is independent of the specification version.

```bash
python -m pip install pystac-ext-processing
```

- [Create an Item with processing metadata](tutorials/first-steps.md) and round-trip it through STAC JSON.
- [Use the wrappers](how-to/use-extension.md) for Assets, Collection summaries, item asset definitions, and Providers.
- [Look up fields and storage behavior](reference/fields.md).
- [Browse the Python API](reference/api.md).
- [Understand validation boundaries](explanation/architecture.md).
