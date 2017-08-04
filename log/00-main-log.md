# Log for the honoursprogramme

This is my main log file for the honoursprogramme. Weekly progress will be recorded in the log directory.

| Config        |          |
| ------------- |:--------:|
| time feedback ICFP paper | ~20 hours |
| time early (april-may) literature study | ~40 hours |
| time late (june-july) literature study | ~40 hours |
| time creating type system | ~20 hours |
| total time | 20 + 40 + 40 + 20 = 120 hours |
| Approximate time required | ~270 |

## Meetings

| Honoursprogramme meetings        |          |
| ------------- |:--------:|
| 29 March 2017   | Initial meeting |
| 23 May 2017   | Discussion of type systems |

| Project meetings        |          |
| ------------- |:--------:|
| 18 April 2017   | Received feedback from ICFP paper |
| 19 April 2017   | Discussed feedback from ICFP paper |
| 4 May 2017   |  |
| 6 June 2017   | Literature study discussion |
| 27 June 2017   | Type-&-effect system discussion |
| 6 July 2017   | Discussion of revision of type-&-effect system |
| 10 July 2017   | Discussion of ICFP SRC paper |
| 14 July 2017   | Meeting to discuss compiler (to start on implementation) |

## ICFP paper (April - May 2017)
During my first honoursprogramme, I co-authored a paper which was sent to ICFP. The honoursproject ended before we received feedback from ICFP. When feedback was received, changes had to be made. These changes for me include:
* Adding more queens benchmarks from other systems
* Adding more loops benchmarks from other systems
* Adding interpreter benchmark
* Adding interpreter benchmark from ohter systems
* Adding parser benchmark
During the development of these benchmarks, I could also get more familiar with the internals of the compiler of Eff. Due to bugs in the compiler, the benchmarks had to be made correctly (in order to not trigger bugs).

## Literature study (April - June 2017)
1. Andrej Bauer and Matija Pretnar. 2014. An Effect System for Algebraic Effects and Handlers. Logical Methods in Computer Science 10, 4 (2014). https://doi.org/10.2168/LMCS-10(4:9)2014

2. Andrej Bauer and Matija Pretnar. 2015. Programming with algebraic effects and handlers. Journal of Logical and Algebraic Methods in Programming. 84, 1 (2015), 108–123. https://doi.org/10.1016/j.jlamp.2014.02.001

3. Stephen Dolan and Alan Mycroft . 2017. Polymorphism, Subtyping, and Type Inference in MLsub. In Proceedings of the 44th ACM SIGPLAN Symposium on Principles of Programming Languages (POPL 2017). ACM, New York, NY, USA, 60–72. https://doi.org/10.1145/3009837.3009882

4. Daniel Hillerström and Sam Lindley. 2016. Liberating Effects with Rows and Handlers. In Proceedings of the 1st International Workshop on Type-Driven Development (TyDe 2016). ACM, New York, NY, USA, 15–27. https://doi.org/10.1145/2976022.2976033

5. Daan Leijen. 2014. Koka: Programming with row polymorphic effect types. arXiv preprint arXiv:1406.2061 (2014).

6. Daan Leijen. 2017. Type Directed Compilation of Row-typed Algebraic Effects. In Proceedings of the 44th ACM SIGPLAN Symposium on Principles of Programming Languages (POPL 2017). ACM, New York, NY, USA, 486–499. https://doi.org/10.1145/3009837.3009872

7. Sam Lindley, Conor McBride, and Craig McLaughlin. 2017. Do Be Do Be Do. In Proceedings of the 44th ACM SIGPLAN Symposium on Principles of Programming Languages (POPL 2017). ACM, New York, NY, USA, 500–514. https://doi.org/10.1145/3009837.3009897

8. Gordon D. Plotkin and Matija Pretnar. 2013. Handling Algebraic Effects. Logical Methods in Computer Science 9, 4 (2013). https://doi.org/10.2168/LMCS- 9(4:23)2013

9. Matija Pretnar. 2014. Inferring Algebraic Effects. Logical Methods in Computer Science 10, 3 (2014). https://doi.org/10.2168/LMCS-10(3: 21)2014

10. Matija Pretnar. 2015. An introduction to algebraic effects and handlers. invited tutorial paper. Electronic Notes in Theoretical Computer Science 319 (2015), 19–35.

## ICFP Student Research Competition (July 2017)
Deadline: 10 July

Towards a core language with row-based effects for optimised compilation

Algebraic effects and handlers are a very active area of research. An important aspect is the development of an optimising compiler. Eff is an ML-style language with support for effects and forms the testbed for the optimising compiler. However, Eff does not offer explicit typing, which makes it easy for type checking bugs to be introduced during the construction of optimised compilation. This work presents a new core language with row-based effects. The core language is explicitly typed in order to reduce bugs in the optimised compilation.

## Start implementation (July - September 2017)

## July 16 - July 20
I made a new branch: rowbased. In this branch, I will implement the new type-&-effect system.

The first thing I did was to remove the `check` keyword. This keyword was unused and removing it cleans up the code.

### July 20 - August 2
I made some diagrams on paper to get a good view of the compilation pipeline. This quick diagram evolved into a thorough study of the compiler in order to deepen my insight.

I went through the code in order to understand the current type inference algorithm. I did this by tracing the execution of a simple program, as well as "rebuilding" the current inference engine. This was much more time consuming than I expected it to be.

* Coercions => no more scheme
* Polytype describe possible types

## Achievements
- Completed literature study
- Wrote extended abstract for ICFP SRC
