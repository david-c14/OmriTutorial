# OmriTutorial
Some notes about programming the Modular Fungi plugin

I'll start by assuming that you know nothing. Apologies if you know more than that.

### Some general ideas in software development

A computer program is generally written in a language that humans can understand, and then translated using other programs into a sequence of bytes
that can be directly understood by the microprocessor inside a computer. For what we are doing, the human readable program is written into 
text files called source code. Then a program called a compiler turns each source code file into a lump of binary data. A finished program might 
involve many source code files, and so a further program called a linker takes each of the lumps of binary data and combines them to produce
the finished executable or dll (library). This whole process of compiling everything and linking is generally called building, and the files that 
result are often called a build.

For any moderate sized project another program called make will likely be used. Make reads a text file called a makefile which contains a recipe
for the project. It says which source code files should be compiled and linked, and in what order. Make has two big advantages for us.

- It means that the programmer can put the recipe alongside the rest of the source code, so that somebody else trying to build the project
does not need to know how the bits fit together. They can simply run the make program, it will read the recipe and do everything in the correct
order.

- Make also understands the dependencies of the project; which parts need to be built before others; which parts are affected by changes. 
So if a single source code file is altered, only some parts of the recipe will need to be followed in order to build the project completely.
This can save a great deal of time. A full build from scratch might take many minutes or hours, but once it is built, small changes to the source code
will usually only require a few seconds to be rebuilt, because most of the compiled sections are unaffected.

### VCVRack

VCVRack source code is mostly written in a language called C++ pronounced see plus plus. There are two main types of source code files. 
The separate binary objects usually each come from a .cpp file; but there are also shared sections of code in files called header files which
have a .hpp file extension. As the compiler reads a .cpp file, it may encounter an instruction to read a .hpp file; it will stop where it is 
in the first file and start reading the named .hpp file; when it reaches the bottom of that, it will carry on where it left off in the .cpp file.
Header files can contain references to other header files, so this reading process may end up including a great many files before the compiler
has read everything it needs. The main purpose of header files is to write information once, that will be used by many different source code
files.

### Modular Fungi

The Modular Fungi source code can be found [here](https://github.com/david-c14/ModularFungi). Lets  take a quick look at just the top of the Makefile:
```
# Must follow the format in the Naming section of https://vcvrack.com/manual/PluginDevelopmentTutorial.html
SLUG = ModularFungi

# Must follow the format in the Versioning section of https://vcvrack.com/manual/PluginDevelopmentTutorial.html
VERSION = 0.6.0
```

These are declarations of two important pieces of information, every VCVRack plugin will define this information in the Makefile, and it 
will be used in the recipe to ensure that the plugin includes this information.

The SLUG is the internal name for your plugin, this is the name that is used inside patch files so that VCV knows which modules need to 
be loaded

The VERSION is how the plugin manager knows when to update. The plugin manager in your copy of VCVRack knows the version of each plugin
(because that VERSION information was baked into it during the build). The plugin manager on the server knows the current version of each plugin
that it hosts. The two of the compare notes, and the server can then send your computer any plugins where the two version numbers are
not the same.
