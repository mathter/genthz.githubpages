---
layout: page_doc
title: Simple Types
menu_type: topic
page_type: page_doc
---
# Documentation v3
## Simple types
If constructor calling is enough to instance creating and filling fields of the class than this one is simple type.
All primitives are simple types also.

There is default realization for generation of simple types (see [DashaDsl.def()](../apidocs/org/genthz/dasha/dsl/DashaDsl.html){:target="_blank"}.
This class [DashaDsl.def()](http://localhost:4000/v3/apidocs/org/genthz/dasha/dsl/DashaDsl.html){:target="_blank"} uses
[DashaDefaults](../apidocs/org/genthz/dasha/DashaDefaults.html){:target="_blank"} to create default instance builders for filler.
Thus you can override any of methods [DashaDefaults](../apidocs/org/genthz/dasha/DashaDefaults.html){:target="_blank"} or implements
[Defaults](../apidocs/org/genthz/Defaults.html){:target="_blank"} directly.

