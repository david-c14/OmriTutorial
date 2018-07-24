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

Further down the Makefile we find this:
```
# Include the VCV Rack plugin Makefile framework
include $(RACK_DIR)/plugin.mk
```

This is another example of including shared information. The Makefile inside each plugin project is generally tiny. All the hardwork is
done in other makefiles that Andrew has written and which contain the recipes and rules to make plugins in general. Here the instruction
is to incorporate the contents of the plugin.mk file from the main Rack directory (this file is probably not in the downloaded version of Rack).

### Resources

There is a directory in the project called res, this includes the resources that the plugin will require. In our project it simply
contains 6 svg files and 6 png files. The res directory will be included in the finished zip file so that the graphics become part of 
what the plugin manager publishes.

### Source files

The src directory contains our source code files, this directory is not included in the zip file, because it is not necessary to have
the source code for someone to run the plugin, generally all they need is the plugin.dll and the resources.

### ModularFungi.cpp

Take a look at the ModularFungi.cpp file in the src directory, we'll break it down bit by bit:
```
#include "ModularFungi.hpp"
```
This is the instruction to include a header file. We'll look at what's in there later. The compiler will stop reading the .cpp file at this point, and read all of the .hpp file, including any further files that the .hpp file asks to include; only then will it move on to the next line in the .cpp file.
```
Plugin *plugin;
```
This is where we get into some programming, don't worry too much about this yet. This line is saything that we need a reference to a Plugin object (capital P) and we will call that reference plugin (small p). 
```
void init(rack::Plugin *p) {
	plugin = p;
	p->slug = TOSTRING(SLUG);
	p->version = TOSTRING(VERSION);

	// Add all Models defined throughout the plugin
	p->addModel(modelBlank_3HP);
	p->addModel(modelBlank_6HP);
	p->addModel(modelBlank_10HP);
	p->addModel(modelBlank_16HP);
	p->addModel(modelBlank_20HP);
	p->addModel(modelBlank_32HP);

	// Any other plugin initialization may go here.
	// As an alternative, consider lazy-loading assets and lookup tables when your module is created to reduce startup times of Rack.
}
```
Here we are declaring a function, a piece of code that carries out a particular action or operation. The function is called 'init' and it relies on a reference to a Plugin (capital P) and that reference is called p. If someone tries to use this function, they will have
to give it a reference to a Plugin.

From top to bottom the function then does the following:
- Set our plugin reference (small p plugin from earlier) to refer to the same Plugin as p
- set the slug of the plugin to the SLUG value in the Makefile
- set the version of the plugin to the VERSION value in the Makefile
- add 6 models to the list of models that the plugin has.

This is how VCVRack manages all the plugins and modules. When VCVRack starts, it searches for subfolders in the plugins folder. In each one it expects to find a plugin.dll. It will load this and each plugin.dll should have an init function. VCVRack constructs a Plugin object and puts it in a list. It calls the init function, passing it a reference to the Plugin object, and the init function sets the
internal name (SLUG) and version, and gives the Plugin a list of models.

After all this VCVRack has a list of Plugins, each of which has a list of models, and this is everything VCVRack needs to be able to 
offer the module browser, and to load existing patch files.

### ModularFungi.hpp
```#include "rack.hpp"

using namespace rack;

// Forward-declare the Plugin, defined in Template.cpp
extern Plugin *plugin;

// Forward-declare each Model, defined in each module source file
extern Model *modelBlank_3HP;
extern Model *modelBlank_6HP;
extern Model *modelBlank_10HP;
extern Model *modelBlank_16HP;
extern Model *modelBlank_20HP;
extern Model *modelBlank_32HP;
```
At the top we include rack.hpp, this header file is provided by Andrew, and through this file the compiler has access to definitions of all the things in VCV. 

Next is `using namespace rack;` which is there because many of the definitions of VCV are segregated into a collection of names called Rack. By declaring that we are using that namespace, we don't need to explicitly say which namespace we are referring to. instead of having to say we want a `rack::Plugin`, we can simply say `Plugin`. 

Now there is a declaration of a reference to a Plugin, called plugin.  We've seen this before in the ModularFungi.cpp file, but this one is slightly different. It says that reference is external, using the `extern` keyword. There are lots of pieces of code which need to know about the reference, but once everything is linked together, there must be only one actual reference. The actual reference will come from the ModularFungi.cpp file, and every other declaration of it should be external, to indicate that there will be a reference 
like this, but this is not it.

Finally we declare references to our 6 models. Again these are external declarations, because we are going to declare the one true reference somewhere else.

### Blanks.cpp

I'm not going to copy the entirety of this file into here. You can look at it yourself.

At the top we include the ModularFungi.hpp, this in turn includes rack.hpp and everything else that comes from that.

Then we declare a new type of structure which we have decided to call `Bitmap` and it's a structure based on a structure that Andrew provided called `TransparentWidget`. In this case Transparent means that it doesn't listen to mouse events, they just pass straight through it so that the underlying ModuleWidget can be dragged around. A knob would be an example of an OpaqueWidget because it does listen to the mouse events, allowing the knob to respond to being dragged, and at the same time not passing those messages through so that the module doesn't move at the same time.

We declare some information that this structure will hold:
- The path to the png file
- A flag to say that we have tried to load the png file
- A number which is used internally to refer to the png file after it's been loaded.

Then we provide a function which can draw the png file when we ask it to. This function is passed a context from the graphics library, and everything it does happens in that context. Basically the function will try to load the png file once, and if it succeeds, it will
centre this widget onto the module based on the size of the png image. It will store the internal reference number for the loaded png so that it can be used later. After that, if the function is called it will use the png image to create a fill texture, and it will draw 
a rectangle on the screen filled with that texture. This is how we get the image displayed.

A second function in the Bitmap structure is the `draw` function. This is a standard function in the TransparentWidget, and we are override it's behaviour. VCVrack will call this function when it thinks the widget needs to be drawn, and so in here we call our big `DrawImage` function, and then we let TransparentWidget do whatever it would normally do with a draw.

Now we declare another structure, this is our ModuleWidget for the 3HP blank. Every module in rack has a Module and a ModuleWidget. The Module does the audio work, and the ModuleWidget does the visuals. Because our plugin does no audio work, we can get away with using a plain Module and not have to declare our own variation. But we do want to control the visuals, so we declare our own variety of ModuleWidget which we've called Blank_3HP. When one of these is created it sets the panel background to the svg file, and it creates a new
Bitmap (as we declared above) and tells it which filename to load. Then it makes the Bitmap widget a child of itself, so that it will get drawn in the correct order. 

We declare all 6 of our different ModuleWidgets in this way.

Finally at the bottom of the file we declare our 6 models. These are the actual declarations of the models which will satisfy the external declarations that we saw in the header file, and these are also the actual models which we added to the list of models in the plugin in the ModularFungi.cpp file.

For each of the models we create, we specify which Module and ModuleWidget it uses, What names to use in the module browser, and what tags to use to categorise the module.

The end.


