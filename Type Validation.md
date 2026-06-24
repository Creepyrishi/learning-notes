
### Annotated
* Syntax:

  ```python
  from typing import Annotated
  Annotated[type, metadata]
  ```

* What it does:
  Adds **extra information (metadata)** to a type **without changing the type itself**.
  Python ignores the metadata, but frameworks (like FastAPI, Pydantic) use it for:
  * validation
  * documentation
  * constraints

* Example:

  ```python
  age: Annotated[int, "must be > 0"]
  ```

  → still an `int`, but with extra info attached

* When to use:
  When libraries need more info about your types.

