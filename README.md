# A core language with row-based effects for optimised compilation

## Honoursproject
I am participating in the research track of the [Honoursprogramme of the Faculty of Engineering Science of KULeuven (dutch link)](https://eng.kuleuven.be/studenten/engineering-essentials-ingenieurscompetenies/honoursprogramma/).

For my honoursproject I am participating in the "Algebraic Effect Handlers: Harnessing the Fundamental Power of Effects" project from the DTAI group of the department of Computer Science.

We use the information of the type and effect system and the syntactic structure
of goals to perform a number of optimisations. We mainly target optimisations
that helps in removing handlers.

In this work a new type-&-effect system was designed. A type inference algorithm was constructed and has been implemented in Eff.

## Abstract
Algebraic effects and handlers are a very active area of research. An important aspect is the development of an optimising compiler. Eff is an ML-style language with support for effects and forms the testbed for the optimising compiler. However, Eff does not offer explicit typing, which makes it easy for type checking bugs to be introduced during the construction of optimised compilation. This work presents a new core language with row-based effects. The core language is explicitly typed in order to reduce bugs in the optimised compilation.

## Installing Eff
See the [Eff repository](https://github.com/matijapretnar/eff) for instructions.
