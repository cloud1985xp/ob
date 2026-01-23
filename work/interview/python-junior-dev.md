---
tags:
  - interview
  - hr
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Python Junior Dev

Created: 2023年9月27日 下午3:43

List vs Tuple

(Immutable)

Lambda

Decorator

### **7. How do you copy an object in Python?**

In Python, the assignment statement (`=` operator) does not copy objects. Instead, it creates a binding between the existing object and the target variable name. To create copies of an object in Python, we need to use the **copy** module. Moreover, there are two ways of creating copies for the given object using the **copy** module -

**Shallow Copy** is a bit-wise copy of an object. The copied object created has an exact copy of the values in the original object. If either of the values is a reference to other objects, just the reference addresses for the same are copied.

**Deep Copy** copies all values recursively from source to target object, i.e. it even duplicates the objects referenced by the source object.

### **How are arguments passed by value or by reference in python?**

- **Pass by value**: Copy of the actual object is passed. Changing the value of the copy of the object will not change the value of the original object.
- **Pass by reference**: Reference to the actual object is passed. Changing the value of the new object will change the value of the original object.

In Python, arguments are passed by reference, i.e., reference to the actual object is passed.