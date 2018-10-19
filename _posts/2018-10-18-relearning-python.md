---
title: "Relearning Python: Introduction"
categories: 
  - Python
tags:
  - best practice
  - structure
---

As of today, it has been around 6 years since I abandoned a Python game framework that I was experimenting with. Looking back at the project now, I'm amazed at how inconsistent and unstructured my code was.
There are a lot of conventions that I picked up on without fully understanding and there wasn't a clear direction for how to maintain and distribute the project. However, I will say that the framework
had some neat features and a solid architecture in terms of its ideas -- based on the entity-system-component architecture.

I'm going to be making a series of posts covering the adventures that I went through while bringing my old project up to present day Python standards. First, the dependencies and the language version
will need to be updated. After that an automated build/deploy system will need to be put in place -- python-sfml and SFML will be dependencies. Then the old code will need to be refactored.

The dependencies updating task will be particularly difficult. The game framework was written against a Python binding for a C++ library and that Python binding has since been abandoned. 
In the end, we need a build of sfml and python-sfml as a dependency of my game framework. In order to solve this issue, I decided to try contacting the python-sfml author -- I was successful.
After discussing with the author, I have been given permission to suggest an automated build/deploy strategy. This crutial step should be documented so that python-sfml's future is bright,
so this will be covered in the series as well.
