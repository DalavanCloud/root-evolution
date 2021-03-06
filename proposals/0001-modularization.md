# ROOT Modularization

* Proposal: [ROOT-0001](0001-modularization.md)
* Authors: [Brian Bockelman](https://github.com/bbockelm),  [Oksana Shadura](https://github.com/oshadura), [Vassil Vassilev](https://github.com/vgvassilev)
* Review Coordinator: 
* Status: Under review
* Decision Notes:
* Bug:

## Introduction

One clear advantage of object-oriented systems is the ability to have abstraction layers provide separation between the user of an object and the implementer.  Ultimately, this helps projects scale to very large code bases.  Another technique for scaling projects is using modularization: grouping together significant functionality into distinct units with clear points of interaction.  While ROOT has a strong history in object-oriented programming, it doesn’t have a strong concept of modules. 

Thus, we propose to add four concepts to the ROOT ecosystem:
* __Module__: A set of interdependent classes implementing coherent functionality and providing well-defined APIs.
* __Library__: a module or set of modules  which solves a global problem, while  being  together and that can be used in a program or another library.
* __Package__: A distinct, self-describing resource (file, URL) that provide one or more modules.
* __Package database__: A record of all packages currently available in a ROOT installation.
* __Package manager__: An actor that can locate and install packages into a ROOT installation from a package reference, along with their transitive dependencies.

There are other legacy large object-oriented software systems which, similar to ROOT, consisting of a large number of interdependent  and loosely-coupled classes, mostly organized as set of libraries and build targets.  Classes are often the lowest level of granularity to serve as a unit of software modularization. In ecosystems such as Java,  Python and C++, a further package structure can allow software developers organize their programs into components. A good organization of classes into identifiable and collaborating packages eases the understanding, maintenance of software. To improve the quality of software modularization, assessing the package organization is required [[3]].

Since we address large software systems, such as ROOT framework, consisting of a very large number of classes and packages, we consider that packages are the units of software modularization. 

Packages and modules are not synonymous concepts.

A ROOT module is a set of functions, types, classes and etc., defined in a common namespace (namespace not in a terns of C++ concept). We can define module also as a set of related functionality that exposes a well-defined API. A library is a module or set of modules  which makes sense to be together and that can be used in a program or another library. A package is a unit of distribution that can contain a library,an executable or both. Considering a library is a set of modules, a package bundles a set of modules with related metadata (such as name, version and dependencies) which can be distributed as a standalone unit.

## Problem Statement

Modularization defines a way of grouping of functionality from a software product. It outlines groups in form of modules which identify a particular piece of functionality to solve a set of problems. In general, modularization helps reducing management, coordination and development costs.

We aim to define a set of mechanisms that enables a modular version of ROOT, centered around C++ modules [[1]] [[2]], working as package-centric ecosystem.

Library dependencies alone result in an imposingly complex relationship diagram. By introducing a module layer, we would provide better boundaries between components, allowing ROOT to scale as a project. For example, the level of expertise for the contributor needs can be more localized. It could mean that ROOT by itself could evolve in a new phase, and can potentially interact with many more packages and turns itself into even more useful toolkit. 

By making the boundaries and relationships more explicit through modules, we can better define and implement a ''minimal ROOT'', increasing the chances its functionality can be embedded in other contexts. This enables ROOT users to interact with the wider data science ecosystem.

Packages and package management provide a mechanism for ROOT users to socialize and and reuse projects built in the context of ROOT, it helps to make ROOT more flexible and open for new customers.  This allows ROOT to continue to serve as a community nexus.

In particular, this provides the ROOT team with an improved mechanism to say ''no'' to new modules within the ROOT source itself as users can simply share their packages among each other or in a common store such as github.

Having a package format can help define community standards and allows the community to go in a similar direction, focusing what effort we have and then in the same time to reduces the desire / ability for every experiment and analysis group to invent their own set of packaging on top.

## Proposed solution

Main idea is to provide modularisation of ROOT and easy separability in multiple discrete and independent modules. Packaging system consisting from ROOT package engine and ROOT package manager, with help to get rid of complex library dependencies that are introduced in current product status.

## Idea

Minimal requirement for ROOT package manager is  to be able to define the dependency and the version, in manifest of package. It could be done either for ROOT packages or for external dependencies (compression algorithms packages, for example). This should be a basic schema for ''in-source;; modularization of ROOT subsystems and for plugin procedure of packages that are stored in Git and defined as ROOT external dependencies.

Maximum requirements for package manager could be a next set of features (the list is inspired by functionality of Swift Package Manager, which is supported by a large Swift community, see section ''Future aspects of design'')[[8]].

## Packaging
To have a successful socialization of ROOT project via modularization, we need to agree on a format of ROOT package and define set of available modules and its packages for existing monolithic ROOT framework.

ROOT package is defined as a grouping of software for data analysis and associated resources, intended for it distribution (extension or upgrade of  ROOT functionality). The definition of package assumes a contract for code organization in order to simplify the build and deploy steps. The contract defines a *manifest* file and particular organization of each module.

Manifest file is a file which describes the content of a package. It has self-describing and easy to process by machines format. The manifest file contains information about how the contents should be built, deployed and versionized. 

For example, a package shall contain:
* Manifest file
* Code
  * Set of entities which are grouped in a module
* Code validation
  * Checks for correctness
  * Checks for performance
* Documentation
  * User documentation
  * Developer documentation
* Resources
  * Binary files (such as images and icons)
  * Scripts

Packages can be source or binary. A source package is a package which needs to be built before it can be used. That can impose extra requirements for the users, that is they will have to provide a compiler and external libraries in order to use the package. A binary package is a pre-built source package. Pre-built packages might have specific requirements to the runtime environment. Package byproducts are a set of modules, which could be packed as a library and/or executables, with documentation and unit tests. 

Package manager is a tool (usually a standalone tool) for managing the distribution of software code. There are two separate entities (third one is OS package manager (OSPM) that we are not interested in the current discussion):

1. Language package manager (LPM): an interactive tool (e.g., `go get`) that can retrieve and build specified packages of source code for a particular language (or in the case of ROOT particular runtime).
2. Project/application dependency manager (PDM): an interactive system for managing the source code dependencies of a single project in a particular language. That means specifying, retrieving, updating, arranging on disk, and removing sets of dependent source code, in such a way that collective coherence is maintained beyond the termination of any single command. Its output , which is precisely reproducible , is a self-contained source tree that acts as the input to a compiler or interpreter. One might think of it as ''compiler, phase zero.''

We think about ROOT package manager in terms of of LPM and PDM.

A ROOT package managing system can manage the package lifetime to ensure sustainability in a transparent to the user way. The role of the ROOT package manager is to reduce coordination costs by automating the process of downloading and building all of the dependencies for a project required for users, experiments or total community.

It is challenging to define package granularity. That is left for the users. Packages should not contain too little and too big modules because this in a way defeats the purpose of modularization. Packages should not contain too many and too small modules because this introduces a lot of package management overhead. 

### Manifest file
A package is described by a *manifest* file. The manifest file has a well-defined, textual format which expresses important package properties. It contains metadata such as:
* Package name
* Package version
* Package source location
* Dependency list
* Dependency version

### An example package
In this section we outline how to encapsulate `math` from ROOT into its own package. The example is illustrational and does not suggest ROOT's math to be a package.

#### Manifest file

In this section we make a straw-man proposal how the package manifest file may look like.
```yaml
package:
  name: "Math"
  version: "6.12"
  packageurl: "https://github.com/root-project/root/math/"
Branch: "6.12"
  targets:
    target
      name: "Math"
      dependencies: ["root-base", "gsl2.0"]
Products:
  Module:
    name: MathCore
    Type: Library
    targets: xxx
    sourceurl: "https://github.com/root-project/root/math/mathcore/"
  Module:
    name: MathMore
    type: Library
    targets: xxx
    sourceurl: "https://github.com/root-project/root/math/mathmore/"
```

#### Core, testing, documentation

By default we assume the directory structure of the `sourceurl` is the following:

```bash
math
   |--- manifest.yaml
   |---inc/
   |---src/
   |---test/
   |---perf/
   |---doc/
   |---res/
```

If the package source tree follows other structure this has to be expressed with more configuration in the manifest file.


A very convenient shipping format is an archive file. For example, zip files are also Github-friendly way of package distribution which can make external contributions easier.

## Dependencies
There is no single centralized index of packages. A package's metadata and dependencies are specified in manifest file, which is stored along with the code in its repository. 

### "Dependency Hell"
When a project's packages have requirements that conflict with one another, it creates a situation defined as "dependency hell". 
Dependency hell is a common problem found in software that are built using an add-on software package or that rely on one for complete functionality. Dependency hell can take many forms and occur for many reasons, such as the need to instal add-on software libraries, the need for long chains of installations, problems with a conflicting program, the creation of circular dependencies and more.

## Versioning
In systems with many dependencies, adding new package versions can quickly become a real nightmare. If the dependency specifications are too tight,  we could appear in danger of version lock (the inability to upgrade a package without having to release new versions of every dependent package).

As a solution to this problem, could be a simple set of rules and requirements that dictate how version numbers are assigned and incremented.

Versioning problem could be by itself an independent root-evolution proposal, after implementing the package manager for ROOT. 

## Future aspects of design
1. Package index
2. Documentation generation
3. Resource management
4. Support for Other Build Systems (brew and etc.)
5. Support for Version Control Systems
6. Library-Based Architecture
7. Standardized Licensing (CMS related problem)
8. Importing Dependencies by Source URL
9. Module Interdependency Determination
10. Dependency Resolution
11. Resource Management
12. And many more.

## Source compatibility
ROOT project source compatibility is one main features of project providing by ''regression'' test suites (tutorials/roottest/rootbench repositories). 

New modules should pass full compatibility test suite including support of OS vs. compiler matrix used in Jenkins CI.
Modularization tool could provide a possibility of testing external projects as a part of new modules to be included in ROOT.

## Alternatives considered
One of ideas is to provide a ''homebrew''-like tool that will be able to provide cross platform installation of modules (libraries), with dependencies based on recipes which could be closer to OS package manager. 

''Homebrew'' packaging system by itself is Git-based system with allow user or developer easily upload, synchronize, and extend existing modules - recipes. Homebrew is supported on Mac OS X, has a fork for Linux - LinuxBrew, and there are some efforts in porting it to Windows (need to be tested). Homebrew by itself already has a recipe of ROOT version 5.34.36 and 6.08.10, so first step could be to update existing recipe for ROOT6 and other tags. The second alternative was to develop "ROOTBrew" system.

## Impact on existing code
No impact on existing source code by itself, but it could change a code distribution schema for ROOT  eco-system (ROOT based projects).

## Resolution
We hope to get a feedback from community to evaluate if it supports an idea of development of package manager for ROOT.

## Acknowledgments
The project is developing by support of DIANA-HEP funding.

## Resources
(1): [Vassilev, V., 2017, October. Optimizing ROOT’s Performance Using C++ Modules. In Journal of Physics: Conference Series (Vol. 898, No. 7, p. 072023). IOP Publishing.][1]

(2): [Clang Modules][2]

(3): [Abdeen, H., Ducasse, S. and Sahraoui, H., 2011, October. Modularization metrics: Assessing package organization in legacy large object-oriented software. In Reverse Engineering (WCRE), 2011 18th Working Conference on (pp. 394-398). IEEE.][3]

(4): [Martin, R.C., 2002. Agile software development: principles, patterns, and practices. Prentice Hall][4]

(5): [Parnas, D.L., 1972. On the criteria to be used in decomposing systems into modules. Communications of the ACM, 15(12), pp.1053-1058.][5]

(6): [Java Modularization Dashboard][6]

(7): [Python Import System][7]

(8): [Swift Package Manager, Community Proposal][8]


[//]: # (Links)
[1]: https://www.researchgate.net/profile/Vassil_Vassilev3/publication/319717664_Optimizing_ROOT%27s_Performance_Using_C_Modules/links/59bad690aca272aff2d01c1c/Optimizing-ROOTs-Performance-Using-C-Modules.pdf "Vassilev, V., 2017, October. Optimizing ROOT’s Performance Using C++ Modules. In Journal of Physics: Conference Series (Vol. 898, No. 7, p. 072023). IOP Publishing."

[2]: https://clang.llvm.org/docs/Modules.html "Clang Modules"

[3]: https://scm.gforge.inria.fr/anonscm/svn/cutter/OtherDocs/2011WCRE-ModularizationMetrics.pdf "Abdeen, H., Ducasse, S. and Sahraoui, H., 2011, October. Modularization metrics: Assessing package organization in legacy large object-oriented software. In Reverse Engineering (WCRE), 2011 18th Working Conference on (pp. 394-398). IEEE."

[4]: http://druss.co/wp-content/uploads/2013/10/Agile-Principles-Patterns-and-Practices-in-C.pdf "Martin, R.C., 2002. Agile software development: principles, patterns, and practices. Prentice Hall"

[5]: https://dl.acm.org/citation.cfm?id=361623 "Parnas, D.L., 1972. On the criteria to be used in decomposing systems into modules. Communications of the ACM, 15(12), pp.1053-1058."

[6]: http://openjdk.java.net/projects/jigsaw/dashboards/modularization "Java Modularization Dashboard"

[7]: https://docs.python.org/3/reference/import.html "Python Import System"

[8]: https://github.com/apple/swift-package-manager/blob/master/Documentation/PackageManagerCommunityProposal.md "Swift Package Manager, Community Proposal"

[9]: https://medium.com/@sdboyer/so-you-want-to-write-a-package-manager-4ae9c17d9527 "So you want to write a package manager, Blog Post"
