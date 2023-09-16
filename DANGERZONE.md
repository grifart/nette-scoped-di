# Danger zone

In some cases, you need to... But please only do this once you are sure of the consequences!

## Access the outer container from the inner one

Please note that this can create a dependency of the inner code on a user's code, leading to breaking the reusability principle.

Please stay away from this!

```yml
# model-container.neon
service:
    - MyBrokenService(@outerContainer::getByType(WhateverServiceFromTheOuterContainer))  # Do NOT do this unless the interface provided by the inner code from the inner container
    - AServiceRequiringTheWholeOuterContainer(@outerContainer)
```


## Accessing internals of the inner container from the outside

Accessing the internals from outside can be only helpful if you need to get some data for tests or manipulate some data. Otherwise, you can quickly break the app using an internal undocumented API (of this library and your inner container).

Please stay away from this!

```yml
service:
    - ServiceRequiringTheServiceFromModelContainer(@model.container.getByName("the.service.name")) # Hey! never do this!
    - ServiceRequiringTheServiceFromModelContainer(@model.container).getByType(WhatEverService) # Hey! never do this!
    - ServiceRequiringTheModelContainer(@model.container) # Hey! This is not the way!
```