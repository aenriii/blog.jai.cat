---
Title: "making a VM from scratch, part 1"
Date: 2023-06-19
---

## or: what in the cosmic fuck do we even start with

ok so lets get one thing straight; _i have no idea what im doing_-
i have mild experience working with the jvm's internals (via helping
with [something that could run hello world](https://github.com/kwzuu/jvm-rs))
but that's more than about it.

### what specs do we even want to build it to?

this ive done some planning for; so here's what im thinking

#### 1. Register Based

stack-based vms are The Thing for many people, and not only are there a lot
more optimization and jit compiling possibilities with register based vms,
it also gives my vm a little bit more standing-out-from-the-croud points

#### 2. Entirely in Rust, as few dependencies as possible

other than memory safety and speed promises made by the rust language,
this is just a personal preference.

#### 3. First Priority: Speed (and Accuracy)

other than security, speed and accuracy are the core of any good VM. Running as fast
if not faster than the JVM on a similar computational application using pre-runtime
optimizations (including JIT soon:tm:) is the goal here, without sacrificing accuracy.

#### 4. Embeddable, Anywhere rust can go

the intent for this vm is to be embeddable in any application, and to be able to
run any rootIR (or bytecode) that is compiled for it. this means that if someone
compiles java to rootIR and then makes a vm instance in a Go application, it should
run just fine.

#### 5. Extensible, promoting usage of third-party bidirectional transpilers

The hope is to have other people (and me, of course) make transpilers from common
languages or other IRs. [eden (WIP)](https://github.com/jdadonut/eden) is an example
of something that converts JVM bytecode (such as what's distributed in JAR and
CLASS files) to rootIR, and therefore can be run on this vm. (it's very much in
the early stages of this, and relies on another project before i can get it
done, but it's a good example of what i mean)

#### 6. Interoperability as a first-class feature

Considering the fact that this vm is meant to be embeddable and many languages
are supposed to be able to be transpiled to it, interoperability is a must.
This means that the vm should be able to not only interact with the environment
of the host application, but also with transpiled code from other languages.
For example, a vm instance in a Rust application should be able to call a function
transpiled from Python that uses a library transpiled from x86 assembly.

#### 7. Secure by design

The VM should be secure by design, and should be as fast as possible without
sacrificing the security of the end user.

### now, what are we gonna need to make to make this process easier?

personally, i have a few ideas as to libraries/subprojects we can make to make this process
easier. here's a list of them:

#### 1. rootir

rootIR is a rust library that will be used to represent the ir of the vm in code.
It will be able to write and read both .rir files and .rira files, the archive format
associated with rootIR.

#### 2. vmalloc

vmalloc is a rust library that will be used to handle memory allocations for the vm.
it will need to be able to allocate stack and heap memory, and will need to be able
to manage blocks of memory that have arbitrary sizing.

#### 3. vstarr

vstarr is a rust library that will (possibly) function as a bidirectional abstract 
syntax converter between stack-based operations and registry-based operations. this
might be one of the hardest parts of the project, but it will be necessary to make
languages like java and python work on the vm.

#### 4. rootpack

rootpack is a rust application that can be used to pack a root-ir based application
as well as all of its dependencies and the rootvm into a single executable. i have
no idea how this woulf fucking work lol