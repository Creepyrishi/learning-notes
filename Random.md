`@abstractmethod` make it compulsory for the subclass to implement some specific method

```python
from abc import ABC, abstractmethod

class Vehicle(ABC): # Abstract Base Class
    
    @abstractmethod
    def start_engine(self):
        pass # No implementation here, just a signature

class Car(Vehicle):
    def start_engine(self):
        print("Car engine started! Vroom!")

# Usage
# my_vehicle = Vehicle()  # TypeError: Can't instantiate abstract class
my_car = Car()            # Works perfectly

```

```python
from __future__ import annotations
```
it is a feature that changes **how type annotations are evaluated**. it lets you write type hints without Python immediately trying to evaluate them.

we call ai a dense model when all of the parameters are active and we call space to MoE models

