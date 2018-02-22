<a name="doc-gnumeric-pure"></a>

Gnumeric/Pure: A Pure Plugin for Gnumeric
=========================================

Version 0.16, March 06, 2017

Albert Gräf &lt;<aggraef@gmail.com>&gt;

Gnumeric/Pure is a [Gnumeric](http://www.gnumeric.org/) extension which lets
you use [Pure](http://purelang.bitbucket.org/) functions in Gnumeric, the
Gnome spreadsheet. It offers better execution speed than the existing Perl and
Python plugins, and provides some powerful features not found in other
Gnumeric scripting plugins, such as asynchronous data sources created from
Pure streams.

Introduction
------------

This package provides a [Gnumeric](http://www.gnumeric.org/) extension which
gives you access to the [Pure](http://purelang.bitbucket.org/) programming
language in Gnumeric. It works pretty much like the Perl and Python plugin
loaders which are distributed with Gnumeric, but Gnumeric/Pure offers some
powerful features which aren't found in other Gnumeric scripting plugins:

-   Pure is a functional programming language which fits the computational
    model of spreadsheet programs very well.
-   Pure is based on term rewriting and thus enables you to do symbolic
    computations in addition to the usual numeric calculations.
-   Pure has a built-in [MATLAB]()/[Octave]()-like matrix data structure which
    makes it easy to deal with cell ranges in a spreadsheet in an efficient
    manner.
-   Pure also provides a bridge to [Octave](http://www.octave.org/) so that
    you can call arbitrary Octave functions using this extension.
-   Pure also has built-in support for lazy data structures and thus allows
    you to handle potentially infinite amounts of data such as the list of all
    prime numbers. Gnumeric/Pure lets you turn such lazy values into
    asynchronous data sources computed in the background, which update the
    spreadsheet automatically as results become available.
-   Last but not least, Pure is compiled to native code on the fly. This means
    that, while startup times are a bit longer due to Pure's JIT compiler
    kicking in (you'll notice this if you open a spreadsheet with Pure
    functions), the resulting compiled code then typically executes *much*
    faster than equivalent interpreted Perl and Python code.

Once the plugin loader is installed and enabled, you can try the Pure
functions in the provided examples and start adding your own plugin scripts.
As of version 0.12, there's a new helper script `pure-gnm` which generates the
required plugin.xml files to make this easy. Various examples can be found in
the examples folder in the distribution, which should help you to get started
with Gnumeric/Pure fairly quickly.

For more advanced uses, Gnumeric/Pure also provides a programming interface
which lets you do various special tasks such as modifying entire ranges of
cells with one Pure call, calling Gnumeric functions from Pure, and setting up
asynchronous data sources. The manual explains all this in detail.

------------------------------------------------------------------------------

> **Note:** This manual assumes that you're already familiar with Gnumeric as
> well as the Pure language and its programming environment. If not then you
> should consult the corresponding documentation to learn more about these.

------------------------------------------------------------------------------

Copying
-------

Copyright (c) 2009-2017 by Albert Graef.

Gnumeric/Pure is free software: you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation, either version 2 of the License, or (at your option) any later
version.

Gnumeric/Pure is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with
this program. If not, see &lt;<http://www.gnu.org/licenses/>&gt;.

Installation
------------

Obviously, you need to have both Pure and Gnumeric installed. Pure 0.36 and
Gnumeric 1.9.13 or later are known to work. The current release of this module
will work with the latest, GTK3-based versions of Gnumeric (Gnumeric 1.11 and
later, various versions starting from 1.12.23 have been tested and are known
to work).

Please note that the Gnumeric plugin interface is a moving target, which means
that from time to time the gnumeric-pure module sources are updated to
accommodate the latest changes in Gnumeric. Thus, depending on which Gnumeric
version you are running, you may have to go with an older version of
gnumeric-pure. In particular, if you are still running Gnumeric 1.10 or
earlier then you should use version 0.12 of the module instead:

> <https://bitbucket.org/purelang/pure-lang/downloads/gnumeric-pure-0.12.tar.gz>

Run `make` to compile the software. You might have to adjust the settings at
the beginning of the Makefile to make this work. Once the compile goes
through, you should now have a `pure_loader.so` file in the `pure-loader`
subdirectory. You can install the plugin and related stuff with
`sudo make install` in the global Gnumeric plugin directory, or if you prefer
to install it into your personal plugin directory then run
`make install-local` instead. (The latter is recommended if you plan to
customize any of the sample plugin scripts included in the distribution for
your purposes.)

Typically, `make install` and `make install-local` will install the plugins
into the following directories by default (here and in the following
`<version>` denotes the version of Gnumeric you have installed):

-   System-wide installations go into
    `/usr/local/lib/gnumeric/<version>/plugins` or similar, depending on
    Gnumeric's installation prefix (usually either `/usr/local` or `/usr`).
-   User-specific installations go into `~/.gnumeric/<version>/plugins`.

The Makefile tries to guess the installation path and version number of
Gnumeric on its own. If it guesses wrong, you can change these using the
Makefile variables `prefix` and `gnmversion`, respectively. For instance:

``` {.sourceCode .sh}
$ make prefix=/usr gnmversion=1.12.4
```

In either case, `make install` also installs the `pure-gnm` helper script
under the Pure installation prefix. (This is a little convenience script to
generate the `plugin.xml` files used by Gnumeric to load a plugin; see
[Defining Your Own Functions](#defining-your-own-functions) for details.)

If `make install` doesn't work for some reason, you can also just copy the
`pure-func` and `pure-loader` directories manually to your Gnumeric plugin
directory. You can still run `make install` in the `pure-gnm` subdirectory to
get the `pure-gnm` script installed in this case.

Setup
-----

Once Gnumeric/Pure has been properly installed, you should see it in
Gnumeric's Tools/Plug-ins dialog. There are actually two main entries, one
labelled "Pure functions" and the other one labelled "Pure plugin loader". You
need to enable both before you can start using Pure functions in your Gnumeric
spreadsheets.

Gnumeric doesn't provide much in the way of GUI customization options right
now, but at least it's possible for plugins to install and configure
additional menu and toolbar options. Gnumeric/Pure adds three additional
options to the Tools menu which allow you to stop asynchronous data sources,
reload Pure scripts and edit them. After installation, the definitions of
these items can be found in the `pure-loader/pure-ui.xml` file in your
Gnumeric plugin directory. Have a look at this file and edit is as desired.
E.g., if you want to put the Pure-related options into a submenu and enable
toolbar buttons for these options, then your `pure-ui.xml` file should look as
follows:

``` {.sourceCode .xml}
<ui>
  <menubar>
    <menu name="Tools" action="MenuTools">
      <separator/>
      <menu name="Pure" action="PureMenu">
        <menuitem action="PureStop"/>
        <menuitem action="PureReload"/>
        <menuitem action="PureEdit"/>
      </menu>
    </menu>
  </menubar>
  <toolbar name="StandardToolbar">
    <separator/>
    <toolitem action="PureStop"/>
    <toolitem action="PureReload"/>
    <toolitem action="PureEdit"/>
  </toolbar>
</ui>
```

Basic Usage
-----------

With Pure/Gnumeric installed and enabled, you should be ready to join the fun
now. Start up Gnumeric, click on a cell and invoke the "f(x)" dialog. The Pure
functions available for use are shown in the "Pure" category. E.g., click on
`pure_hello`. Now the Pure interpreter will be loaded and the function
description displayed. Click "Ok" to select the `pure_hello` function and then
"Ok" again to actually insert the function call (without arguments) into the
current cell. You should now be able to read the friendly greeting returned by
the function.

Of course, you can also enter the function call directly as a formula into a
cell as usual. Click on a cell, then enter the following:

    =pure_hello(getenv("USER"))

The greeting should now be displayed with your login name in it.

Play around a bit with the other Pure functions. These functions are nothing
special; they are just ordinary Pure functions which are defined by the
`pure_func.pure` script in the `pure-func` subdirectory of your Gnumeric
plugin directory. You can have a look at them by invoking the "Edit Pure
Scripts" option which gets added to the Tools/Pure menu once the Pure plugin
loader is enabled. (This will invoke the emacs editor by default, or the
editor named by the `EDITOR` environment variable. You can set this
environment variable in your shell's startup files.) The Tools/Pure menu
contains a second Pure-related option, "Reload Pure Scripts" which can be used
to quickly reload all loaded Pure scripts after edits; more about that later.

Please note that most of the functions in `pure_func.pure` are rather useless,
they are only provided for illustrative purposes. However, there are some
useful examples in there, too, in particular:

-   `pure_eval` lets you evaluate any Pure expression, given as a string in
    its first argument. E.g., try something like
    `=pure_eval("foldl (+) 0 (1..100)")`. Additional parameters are accessible
    as `x!0`, `x!1`, etc. For instance: `=pure_eval("x!0+x!1",A1,B1)`.
-   `pure_echo` just displays its arguments as a string in Pure syntax, as the
    interpreter sees them. This is useful for debugging purposes. E.g.,
    `=pure_echo(A1:B10)` shows the given range as a Pure matrix.
-   `pure_shell` is a variation of `pure_eval` which executes arbitrary Pure
    code and returns the last evaluated expression (if any) as a string. This
    is mainly provided as a convenience to create an "interactive Pure shell"
    which lets you evaluate Pure code inside Gnumeric. To these ends, simply
    prepare a text cell for entering the code to be evaluated, and then apply
    `pure_shell` on this text cell in another cell to display the result.

A spreadsheet showing most of the predefined functions in action can be found
in `pure-examples.gnumeric` example distributed with Gnumeric/Pure.

Interactive Pure Shell
----------------------

The `pure-examples.gnumeric` spreadsheet also includes an instance of
`pure_shell` which lets you evaluate arbitrary Pure code in the same
interpreter instance that executes Gnumeric/Pure functions. This is very
helpful if you're developing new Pure functions to be used in Gnumeric. It
also lets you use Gnumeric as a kind of GUI frontend to the Pure interpreter.
You can try this now. Open the `pure-examples` spreadsheet in Gnumeric and
enter the following into the input cell of the Pure shell:

    > scanl (+) 0 (1..20)
      [0,1,3,6,10,15,21,28,36,45,55,66,78,91,105,120,136,153,171,190,210]

Note that here and in the following the prompt `>` indicates a Pure expression
to be evaluated in *Gnumeric* (rather than the standalone Pure interpreter),
which is followed by another line indicating the result (printed in the output
cell below the input cell of the Pure shell). You can find the Pure shell at
the bottom of the first sheet in `pure-examples`, see the screenshot below.
For your convenience, there's also a second, bigger one on the second sheet.
You might want to copy this over to a separate spreadsheet which you can use
as a scratchpad for experimentation purposes.

![The Pure shell.](shell.png)

Also note that this is in fact *Pure code* (not a Gnumeric formula) being
evaluated there. You can execute any Pure code, including Pure declarations,
so you can type:

    > using system; puts "Hello, world!";
      14

This prints the string `"Hello, world!"` on standard output, visible in the
terminal window where you launched Gnumeric. Here is another example, showing
how you can invoke any function from the C library, by declaring it as a Pure
`extern` function:

    > extern int rand(); [rand | i = 1..5];
      [1810821799,2106746672,1436605662,1363610028,695042099]

All functions in the Pure prelude are readily available in the Gnumeric Pure
shell, as well as the functions defined in `pure_func.pure` and its imports,
including the programming interface described in [Advanced
Features](#advanced-features). For instance, here's how you can retrieve a
cell value from the current sheet:

    > get_cell "A1"
      "Gnumeric/Pure Examples"

Using `call` (see [Calling Gnumeric from Pure](#calling-gnumeric-from-pure)),
you can also invoke any Gnumeric function:

    > call "product" (1..10)
      3628800.0

Defining Your Own Functions
---------------------------

After playing around with `pure_func.pure` and the interactive Pure shell for
a while, of course you will want to write your own functions, that's what this
extension is about after all! This section shows you how to do this.

### Creating a Simple Plugin

Let's consider a simple example: the factorial function. In Pure this function
can be implemented as follows:

    factorial [x] = foldl (*) 1 (1..x);

Note the list bracket around the argument `x`. You wouldn't normally pass a
single numeric argument that way in Pure, but this is needed here since by
default Gnumeric passes arguments as a list to a Pure function. There are
other ways to configure the call interface to Pure functions, but these
require that we tell Gnumeric about the number and types of arguments, see
[Gnumeric/Pure Interface](#gnumericpure-interface) below. For the moment let's
stick to the default scheme, however, in order to keep things simple.

Put the above definition into a script file, say, `myplugin.pure`. Next we
need to create a `plugin.xml` file to tell Gnumeric about our plugin and which
functions it provides. While these files can be written by hand, this is
tedious and error-prone. Fortunately, recent Gnumeric/Pure versions provide
the `pure-gnm` helper script which makes this quite easy. To use `pure-gnm`
with our plugin script, we have to add a special "hashbang" comment block to
our script which supplies the needed information. In our case, this might look
as follows:

    #! N: My Pure functions
    #! C: Pure
    #! D: My Pure functions.
    #! F: factorial

You can add this comment block anywhere in your plugin script, but usually it
is placed near the beginning. The different fields have the following meaning:

-   `N`: the name of the plugin
-   `C`: the function category
-   `D`: a more detailed description of the plugin
-   `F`: a whitespace-delimited list of Pure function names

The contents of the `N` and `D` fields (name and description) are visible in
Gnumeric's "Plugin Manager" dialog. You should specify at least the name field
(otherwise the plugin will be displayed as "Unnamed" in the dialog), while the
description is optional (if you don't specify one, the description of the
plugin will be empty). The `C` field denotes the category under which the
functions listed in the `F` field will be shown in Gnumeric's "f(x)" dialog;
if you don't specify this, the functions will be in the "Unknown" category.
The `F` field is the most crucial part. It must contain all Pure functions
defined in the plugin script or its imports that you want to be visible in
Gnumeric, so you have to keep this in sync with the actual function
definitions in the script; if you don't specify this, the plugin will provide
no functions at all.

The `D` and `F` fields can also be split into multiple lines (each prefixed
with the "hashbang" comment marker and the corresponding field identifier) if
necessary.

So our `myplugin.pure` script now looks like this:

    #! N: My Pure functions
    #! C: Pure
    #! D: My Pure functions.
    #! F: factorial

    factorial [x] = foldl (*) 1 (1..x);

Once you've added the comment block, you can generate the `plugin.xml` file
for the plugin simply as follows:

``` {.sourceCode .sh}
$ pure-gnm myplugin.pure > plugin.xml
```

Note that by default `pure-gnm` writes the `plugin.xml` file to standard
output which is useful if you want to check the generated file first. To
actually create the file, we simply redirect the output to `plugin.xml`.

You'll have to redo this every time your plugin changes (i.e., you've added
new functions or deleted or renamed old ones, or changed the name or
description of the plugin). It's easy to automate this step using `make`.
E.g., the following Makefile will do the trick:

``` {.sourceCode .make}
myplugin = myplugin.pure

all: plugin.xml

plugin.xml: $(myplugin)
        pure-gnm $< > $@

clean:
        rm -f plugin.xml
```

Now you can just run `make` in the plugin directory and it will rebuild the
`plugin.xml` file as needed.

### The plugin.xml File

The `plugin.xml` file resulting from the previous step looks like this:

``` {.sourceCode .xml}
<?xml version="1.0" encoding="UTF-8"?>
<plugin id="Gnumeric_myplugin">
  <information>
    <name>My Pure functions</name>
    <description>My Pure functions.</description>
    <require_explicit_enabling/>
  </information>
  <loader type="Gnumeric_PureLoader:pure">
    <attribute name="module_name" value="myplugin.pure"/>
  </loader>
  <services>
    <service type="function_group" id="myplugin">
      <category>Pure</category>
      <functions>
        <function name="factorial"/>
      </functions>
    </service>
  </services>
</plugin>
```

You can also edit this file by hand if you know what you're doing; in that
case, please check the Gnumeric documentation for details about the format of
these files. The template used by `pure-gnm` to generate these files can be
found in the source distribution (see `plugin.xml` in the `pure-gnm` folder)
or under `/usr/local/lib/pure-gnm` after installation. You can edit this file
(carefully!) in order to implement global changes that you want to be in every
`plugin.xml` file generated by `pure-gnm`.

Two specific items that you might want to edit by hand are the
`<require_explicit_enabling/>` tag and the `id` properties of the `<plugin>`
and `<service>` tags:

-   The `<require_explicit_enabling/>` tag indicates that Gnumeric shouldn't
    enable the new plugin until you explicitly tell it to. You can remove that
    line if you want Gnumeric to automatically enable new plugins as they are
    added to the system.
-   The `pure-gnm` script automatically derives the `id` properties of the
    `<plugin>` and `<service>` tags from the name of the plugin script, which
    is a sensible default in most cases. However, you might have to change
    these identifiers if they happen to collide with other Gnumeric plugins
    and services. This can be done by either editing the generated
    `plugin.xml` file or by renaming the plugin script accordingly.

Note that the only really Pure-specific part in the xml file is the loader
description which also names the Pure script implementing the plugin in the
value of the `module_name` attribute. In this case this is just
`"myplugin.pure"`. This path is taken relative to the directory containing the
`plugin.xml` file, but you can also specify an absolute path there if you want
to keep the plugin script elsewhere. To achieve this with `pure-gnm`, you can
just invoke it with an absolute path name, e.g.:

``` {.sourceCode .sh}
$ pure-gnm $PWD/myplugin.pure > plugin.xml
```

Now you can move the `plugin.xml` file whereever you like and still have
Gnumeric find the script file in its prescribed location. Again, this can be
automatized using `make` fairly easily; we'll return to that in the following
section.

### Loading the Plugin

We now have the plugin script `myplugin.pure` and the `plugin.xml` file in the
same directory, say, `/some/path/myplugin`. We still need to tell Gnumeric
about the new plugin, though, so that it can find it. Unfortunately, the
mechanics of making a plugin known to Gnumeric are somewhat involved, so we
discuss the necessary steps in detail below. There are basically three ways
you can go about this:

-   If you want to keep plugin script and the `plugin.xml` file where they
    are, you'll have to change Gnumeric's plugin path so that it includes the
    *parent* directory `/some/path` (not `/some/path/myplugin`). This is done
    by adding the directory under the Directories tab in Gnumeric's
    Tools/Plug-ins dialog, after which you'll have to restart Gnumeric so that
    it picks up the changes in the plugin search path.
-   Second, you can also move or copy the entire `/some/path/myplugin`
    directory to your personal Gnumeric plugin folder (usually
    `~/.gnumeric/<version>/plugins`). Gnumeric will always search this
    directory for new plugins by default, so modifying the plugin search path
    is not necessary. However, keeping the plugin script in a hidden location
    in your home directory may not be very convenient if you want to modify
    the script later.
-   Third, you can get the best of both previous methods by keeping the plugin
    script where it is and copying just the `plugin.xml` file to your personal
    Gnumeric plugin folder.

The third method tends to be the easiest, but note that it requires that the
plugin script needs to be specified as an absolute path (as sketched out
previously). Fortunately, it's fairly easy to automate this with `make`. The
following requires GNU make to work, and you'll also need to have the Gnumeric
development files installed, so that the Gnumeric version can be determined
easily with a shell command. These rules are to be added to the end of the
Makefile described previously under [Creating a Simple
Plugin](#creating-a-simple-plugin).

``` {.sourceCode .make}
gnmversion=$(shell pkg-config --modversion libspreadsheet-1.12)
plugindir=$(HOME)/.gnumeric/$(gnmversion)/plugins/myplugin

install: $(myplugin)
        test -d $(plugindir) || mkdir -p $(plugindir)
        pure-gnm $(CURDIR)/$< > $(plugindir)/plugin.xml

uninstall:
        rm -rf $(plugindir)
```

Now you can just run `make install` to make the plugin known to Gnumeric. Note
that we also added a rule that allows you to uninstall the plugin if it isn't
needed any more.

In any case, once you fire up Gnumeric again, the new plugin should be listed
as "My Pure functions" on the Plugin List tab in the Tools/Plug-ins dialog.
Check it to enable it. The `factorial` function defined in the plugin should
now be available and ready to be called just like any other Gnumeric function.
For instance, type this into a cell to have the factorial of 10 computed:

    =factorial(10)

Also try saving the spreadsheet and loading it again after restarting
Gnumeric. The plugin will now be loaded automatically and the spreadsheet
should display the proper value of the factorial.

------------------------------------------------------------------------------

> **Note:** Once you start playing around with your own Pure plugins, you may
> run into one common mishap: You open an existing spreadsheet without having
> enabled the plugins it uses. An easily visible symptom of that is that
> you'll see cells showing the `#NAME?` error. You will then have to enable
> those plugins again *and* reload the spreadsheet afterwards, so that
> everything is recalculated properly.
>
> In contrast, just changing the body of a function in a plugin usually needs
> neither a restart of Gnumeric nor a reloading of the spreadsheet. In this
> case it's often sufficient to reload all scripts with the "Reload Pure
> Scripts" option in the Tools/Pure menu, after which you can use
> “Recalculate” (F9) to recompute the spreadsheet.

------------------------------------------------------------------------------

It is also worth mentioning here that the Pure loader can load multiple Pure
plugins (and of course each plugin can provide as many functions as you want).
You only need to tell Gnumeric about them after creating the scripts and
`plugin.xml` files and placing them into corresponding plugin directories.
Just enable the ones that you want in Tools/Plug-ins. All scripts are loaded
in the same Pure interpreter (and thus are treated like one big script) so
that functions in one script can use the function and variable definitions in
another. If you need to access the definitions in the `pure_func.pure` "mother
script", you can also just import it into your scripts with a `using` clause,
i.e.: `using pure_func;`

Another important point is that a Pure plugin script is always loaded in the
directory where it is located, as indicated by the corresponding `plugin.xml`
file, even if it is different from the plugin directory. That is, the current
working directory (which is normally the directory that Gnumeric was started
in) is temporarily set to the directory holding the plugin script while the
script is being loaded. This enables the script to find imported scripts and
other files (such as media files or scripts written in other languages) that
it may need at load time. This wasn't needed in this simple example, but you
can find other examples in the Gnumeric/Pure distribution which make good use
of this feature.

### Spicing It Up

Our plugin example is now essentially complete, but in order to make it really
convenient to use, we may want to add some information about how the
`factorial` function is to be called in Gnumeric. Gnumeric doesn't keep this
kind of information in the `plugin.xml` file, but expects it to be provided by
the plugin itself. In the Gnumeric/Pure interface this can be done by adding a
rule for the `gnm_info` function. In our example we tell Gnumeric that
`factorial` expects a single numeric argument. While we're at it, we might as
well add some helpful documentation to be displayed in Gnumeric's "f(x)"
dialog. The details of this are described in the following section, but to
give you a sneak preview, here's a beefed-up version of our script which
implements all this (you can also find this version of the example along with
a GNU Makefile in the Gnumeric/Pure distribution):

    #! N: My Pure functions
    #! C: Pure
    #! D: My Pure functions.
    #! F: factorial

    factorial x = foldl (*) 1 (1..x);

    using pure_func; // for the gnm_help function
    gnm_info "factorial" = "f", gnm_help "factorial:factorial of a number"
     ["x:number"] "Computes the factorial of @{x}." [] ["=factorial(10)"] [];

Fire up Gnumeric again, press the "f(x)" button and select `factorial` under
the `Pure` category. The "f(x)" dialog should now display the additional
information we added above. Also note that Gnumeric now knows that this
function is supposed to be called with exactly one `f` (numeric) argument.
Therefore the list brackets around the argument of `factorial` aren't needed
any more, so don't forget to remove them, as shown in the above code sample.

This completes our little example. As an exercise, you're invited to add more
functions on your own. (Don't forget to change the `#! F` line accordingly and
rerun `pure-gnm` when you do this, so that Gnumeric knows about the new
functions.) It also pays off to take a look at some of the other included
examples, you can find these in the `examples` folder of the distribution
tarball.

Gnumeric/Pure Interface
-----------------------

We already explained in the previous section that, when a Pure function is
called from Gnumeric, it receives its arguments in a list by default. However,
it is possible to tell Gnumeric about the expected arguments of the function
and also specify a help text to be displayed in the "f(x)" dialog, by giving a
definition of the `gnm_info` function as explained below.

Note that `gnm_info` is really an ordinary Pure function. Thus, rather than
hardcoding this information as static text (such as the "docstrings" used in
Gnumeric's Python extension), the function descriptions can also be
constructed dynamically in corresponding Pure code. This offers an opportunity
for programmatic customizations. But note that the `gnm_info` function will
only be invoked when the plugin script is loaded, so once that is done the
function description remains the same for the entire Gnumeric session.

### Function Descriptions

To describe a given function to Gnumeric, define `gnm_info "<name>"` (where
`<name>` is the name of the function) as a pair with the following elements:

-   The first element, a string, gives the signature of the function. E.g.,
    `""` denotes a function without arguments, `"f"` a function taking a
    single float parameter, `"fs"` a function taking a float and a string
    argument (in that order), etc. Optional parameters can be indicated using
    `|`, as in `"ff|s"` (two non-optional floats, followed by an optional
    string). See below for a complete list of the supported parameter types.
-   The second element is a list of hash pairs `key=>text` which together make
    up the help text shown in Gnumeric's "f(x)" dialog. You should at least
    specify the function name along with a short synopsis here, e.g.
    `GNM_FUNC_HELP_NAME => "frob:the frob function"`. Parameter descriptions
    take the form `GNM_FUNC_HELP_ARG => "x:integer"`. There are a number of
    other useful elements, see below for details.

Both the signature and the function description are optional. That is,
`gnm_info` may return either just a signature string, or a list of hash pairs
with the function description, or both. The signature defaults to a variadic
function which takes any number of parameters of any type (see below), and the
description defaults to some boilerplate text which says that the function
hasn't been documented yet.

Note that if no signature is given, then the function accepts any number of
parameters of any type. In that case, or if there are optional parameters, the
function becomes variadic and the (optional) parameters are passed as a Pure
list (in addition to the non-optional parameters).

Here's the list of valid parameter types, as they are documented in the
Gnumeric sources:

``` {.sourceCode .none}
f : float       no errors, string conversion attempted
b : boolean     identical to f
s : string      no errors
S : scalar      any non-error scalar
E : scalar      any scalar, including errors

r : cell range  content may not be evaluated yet
A : area        array, range (as above), or scalar
? : anything        any value (scalars, non-scalars, errors, whatever)
```

The keys used in the function description may be any of the following, along
with sample text for each type of field:

    GNM_FUNC_HELP_NAME         => "name:synopsis"
    GNM_FUNC_HELP_ARG          => "name:parameter description"
    GNM_FUNC_HELP_DESCRIPTION  => "Long description."
    GNM_FUNC_HELP_NOTE         => "Note."
    GNM_FUNC_HELP_EXAMPLES     => "=sample_formula()"
    GNM_FUNC_HELP_SEEALSO      => "foo,bar,..."

The following keys are only supported in the latest Gnumeric versions:

    GNM_FUNC_HELP_EXTREF       => "wiki:en:Trigonometric_functions"
    GNM_FUNC_HELP_EXCEL        => "Excel compatibility information."
    GNM_FUNC_HELP_ODF          => "OpenOffice compatibility information."

Note that inside the descriptions, the notation `@{arg}` (`@arg` in older
Gnumeric versions) can be used to refer to a parameter value. For instance,
here's a sample description for a binary function which also includes a help
text:

    gnm_info "pure_max" = "ff",
    [GNM_FUNC_HELP_NAME     => "pure_max:maximum of two numbers",
     GNM_FUNC_HELP_ARG      => "x:number",
     GNM_FUNC_HELP_ARG      => "y:number",
     GNM_FUNC_HELP_DESCRIPTION =>
     "Computes the maximum of two numbers @{x} and @{y}.",
     GNM_FUNC_HELP_EXAMPLES => "=pure_max(17,22)"];

As you can see, the function descriptions are a bit unwieldy, so it's
convenient to construct them using this little helper function defined in
`pure_func.pure`:

    gnm_help name::string args descr::string notes examples see_also =
      [GNM_FUNC_HELP_NAME         => name] +
      [GNM_FUNC_HELP_ARG          => x | x::string = args ] +
      [GNM_FUNC_HELP_DESCRIPTION  => descr ] +
      [GNM_FUNC_HELP_NOTE         => x | x::string = notes ] +
      [GNM_FUNC_HELP_EXAMPLES     => x | x::string = examples ] +
      (if null see_also then [] else
       [GNM_FUNC_HELP_SEEALSO     => join "," see_also]);

Now the description can be written simply as follows:

    gnm_info "pure_max" = "ff", gnm_help "pure_max:maximum of two numbers"
      ["x:number", "y:number"]
      "Computes the maximum of two numbers @{x} and @{y}."
      [] ["=pure_max(17,22)"] [];

Since this function only has fixed arguments, it will be called in curried
form, i.e., as `pure_max x y`. For instance, the actual definition of
`pure_max` may look as follows:

    pure_max x y = max x y;

Conversely, if no signature is given, then the function accepts any number of
parameters of any type, which are passed as a list. For instance:

    gnm_info "pure_sum" = gnm_help "pure_sum:sum of a collection of numbers"
      [] "Computes the sum of a collection of numbers."
      [] ["=pure_sum(1,2,3,4,5,6)"] ["pure_sums"];

Here the function will be called as `pure_sum [x1,x2,...]`, where `x1`, `x2`,
etc. are the arguments the function is invoked with. Note that in this case
there may be any number of arguments (including zero) of any type, so your
function definition must be prepared to handle this. If a function does not
have a `gnm_info` description at all then it is treated in the same fashion.
The `pure_func.pure` script contains some examples showing how to write
functions which can deal with any numbers of scalars, arrays or ranges, see
the `pure_sum` and `pure_sums` examples. These employ the following `ranges`
function to "flatten" a parameter list to a list holding all denoted values:

    ranges xs = cat [ case x of _::matrix = list x; _ = [x] end | x = xs ];

E.g., the `pure_sum` function can now be defined as follows:

    pure_sum xs = foldl (+) 0 (ranges xs);

A function may also have both fixed and optional arguments (note that in what
follows we're going to omit the detailed function descriptions for brevity):

    gnm_info "foo" = "ff|ff";

In this case the fixed arguments are passed in curried form as usual, while
the optional parameters are passed as a list. That is, `foo` may be called as
`foo x y []`, `foo x y [z]` or `foo x y [z,t]`, depending on whether it is
invoked with two, three or four arguments.

### Conversions Between Pure and Gnumeric Values

The marshalling of types between Gnumeric and Pure is pretty straightforward;
basically, Pure numbers, strings and matrices map to Gnumeric numbers, strings
and arrays, respectively. The following table summarizes the available
conversions:

  Pure                        Gnumeric
  --------------------------- -------------------------------
  `gnm_error "#N/A"`          error
  `4711`, `4711L`, `4711.0`   scalar (number)
  `"Hello world"`             string
  `()`                        empty
  `(1,2,3)`                   array
  `[1,2,3]`                   array
  `{1,2,3;4,5,6}`             array (or cell range)
  `"A1:B10"`                  cell range (`"r"` conversion)

These conversions mostly work both ways. Note that on input, cell ranges are
usually passed as matrices to Pure functions (i.e., they are passed "by
value"), unless the function signature specifies a `"r"` conversion in which
case the cell ranges themselves are passed to the function in string form.
(Such values can also be passed on to Gnumeric functions which expect a cell
range (`"r"` ) parameter, see [Calling Gnumeric from
Pure](#calling-gnumeric-from-pure) below.)

Conversely, matrices, lists and tuples all become Gnumeric arrays on output,
so usually you'll want to enter these as array functions (`Ctrl-Shift-Enter`
in Gnumeric). As a special case, the empty tuple can be used to denote empty
cell values (but note that empty Gnumeric values may become zeros when passed
as float or array arguments to Pure functions).

Another special case is a term of the form `gnm_error msg`, where `msg` is a
string value indicating a Gnumeric error value such as `"#N/A"`, `"#NAME?"`,
`"#NULL!"`, etc. When returned by a plugin function, the error text will be
displayed in the corresponding Gnumeric cell.

If a Pure function returns a value that doesn't match any of the above then it
is converted to a string in Pure expression syntax and that string is returned
as the result of the function invocation in Gnumeric. This makes it possible
to return any kind of symbolic Pure value, but note that if such a value is
then fed into another Pure function, that function will have to convert the
string value back to the internal representation if needed; this can be done
very conveniently using Pure's `eval` function, see the Pure documentation for
details.

Advanced Features
-----------------

This section explains various additional features provided by the
Gnumeric/Pure interface that should be useful for writing your own functions.
Note that for your convenience all functions discussed in this section are
declared in `pure_func.pure`.

### Calling Gnumeric from Pure

It is possible to call Gnumeric functions from Pure using the `call` function
which takes the name of the function (a string) as its first, and the
parameters as the second (list) argument. For instance:

    gnm_info "gnm_bitand" = "ff";
    gnm_bitand x y = call "bitand" [x,y];

Note that `call` is an external C function provided by Gnumeric/Pure. If you
want to use it, it must be declared in your Pure script as follows:

    extern expr* pure_gnmcall(char* name, expr* args) = call;

However, `pure_func.pure` already contains the above declaration, so you don't
have to do this yourself if you import `pure_func.pure` in your scripts.

Also note that `call` doesn't do any of Gnumeric's automatic conversions on
the parameters, so you have to pass the proper types of arguments as required
by the function.

### Accessing Spreadsheet Cells

Gnumeric/Pure provides the following functions to retrieve and modify the
contents of spreadsheet cells and ranges of such cells:

    extern expr* pure_get_cell(char* s) = get_cell;
    extern expr* pure_get_cell_text(char* s) = get_cell_text;
    extern expr* pure_get_cell_format(char* s) = get_cell_format;
    extern expr* pure_set_cell(char* s, expr *x) = set_cell;
    extern expr* pure_set_cell_text(char* s, expr *x) = set_cell_text;
    extern expr* pure_set_cell_format(char* s, expr *x) = set_cell_format;
    extern expr* pure_get_range(char* s) = get_range;
    extern expr* pure_get_range_text(char* s) = get_range_text;
    extern expr* pure_get_range_format(char* s) = get_range_format;
    extern expr* pure_set_range(char* s, expr *x) = set_range;
    extern expr* pure_set_range_text(char* s, expr *x) = set_range_text;
    extern expr* pure_set_range_format(char* s, expr *x) = set_range_format;

For instance, here's how you use these functions to write and then read some
cell values (try this in the [interactive Pure
shell](#interactive-pure-shell)):

    > set_cell "A14" 42
      ()
    > get_cell "A14"
      42.0
    > set_range "A14:G14" $ scanl (*) 1 (1..6)
      ()
    > get_range "A14:G14"
      {1.0,1.0,2.0,6.0,24.0,120.0,720.0}
    > set_cell_text "A14" "=sum(B14:G14)"
      ()
    > get_cell "A14"
      873.0
    > get_cell_text "A14"
      "=sum(B14:G14)"
    > get_range_text "A14:G14"
      {"=sum(B14:G14)","1","2","6","24","120","720"}

Note that while the `set_cell` function sets the given cell to a constant
value, `set_cell_text` also allows you to store a formula in a cell which will
then be evaluated as usual. Similarly, `get_cell` retrieves the cell value,
while `get_cell_text` yields the text in the cell, as entered by the user
(which will either be a formula or the textual representation of a constant
value). The `set_range`, `set_range_text`, `get_range` and `get_range_text`
functions work analogously, but are used to manipulate entire ranges of cells,
which can be set from Pure tuples, lists or matrices, and retrieved as Pure
matrices.

Functions to retrieve and change the cell format are also provided (watch the
contents of the cell `A14` change its color to blue on entering the first
expression):

    > set_cell_format "A14" "[Blue]0.00"
      ()
    > get_range_format "A14:C14"
      {"[Blue]0.00","General","General"}

There are also functions to get the position of the "current" cell (i.e., the
cell from which a Pure function was called), and to translate between cell
ranges in Gnumeric syntax and the corresponding internal representation
consisting of a pointer to a Gnumeric sheet and the cell or range coordinates:

    extern expr* pure_this_cell() = this_cell;
    extern expr* pure_parse_range(char* s) = parse_range;
    extern expr* pure_make_range(expr* x) = make_range;

Examples:

    > this_cell
      "B4"
    > parse_range this_cell
      #<pointer 0x875220>,1,3
    > make_range (NULL,0,0,10,10)
      "Sheet2!A1:K11"

### Asynchronous Data Sources

Gnumeric/Pure makes it easy to set up asynchronous data sources which draw
values from a Pure computation executed in a background process. This facility
is useful to carry out lengthy computations in the background while you can
continue to work with your spreadsheet. It also allows you to process incoming
data and asynchronous events from special devices (MIDI, sensors, stock
tickers, etc.) in (soft) realtime.

To do this, you simply pass an expression to the `datasource` function. This
is another external C function provided by Gnumeric/Pure, which is declared in
`pure_func.pure` as follows:

    extern expr* pure_datasource(expr* x) = datasource;

The argument to `datasource` is typically a thunk or stream (lazy list) which
is to be evaluated in the background. The call to `datasource` initially
returns a `#N/A` value (`gnm_error "#N/A"`) while the computation is still in
progress. The cell containing the data source then gets updated automatically
as soon as the value becomes available, at which point the `datasource` call
now returns the computed value. E.g., here's how you would wrap up a lengthy
calculation as a thunk and submit it to `datasource` which carries out the
computation as a background task:

    gnm_info "pure_frob" = "f";
    pure_frob x = datasource (lengthy_calculation x&);
    lengthy_calculation x = sleep 3 $$ foldl (*) 1 (1..x);

Note that a cell value may draw values from as many independent data sources
as you want, so the definition of a cell may also involve multiple invocations
of `datasource`:

    gnm_info "pure_frob2" = "ff";
    pure_frob2 x y = datasource (lengthy_calculation x&),
      datasource (lengthy_calculation y&);

Special treatment is given to (lazy) lists, in this case `datasource` returns
a new value each time a list element becomes available. For instance, the
following function uses an infinite stream to count off the seconds starting
from a given initial value:

    gnm_info "pure_counter" = "f";
    pure_counter x = datasource [sleep (i>x) $$ i | i = x..inf];

You can also try this interactively in the Pure shell:

    > datasource [sleep (i>0) $$ i | i = 0..inf]
      0
      1
      ...

Here's another example for the Pure shell which prints the prime numbers:

    > datasource primes with primes = sieve (2..inf);
        sieve (p:qs) = p : (sleep 1 $$ sieve [q | q = qs; q mod p])& end
      2
      3
      5
      ...

Note that when processing a lazy list, the cell containing the call will keep
changing as long as new values are produced (i.e., forever in this example).
The "Stop Data Sources" option in the Tools/Pure menu can be used to stop all
active data sources. "Reload Pure Scripts" also does this. You can then
restart the data sources at any time by using "Recalculate" (`F9`) to
recompute the spreadsheet.

Also note that because of the special way that `datasource` handles list
values, you cannot return a list directly as the result of `datasource`, if it
is to be treated as a single result. Instead, you'll have to wrap the result
in a singleton list (e.g.,
`datasource [[lengthy_calculation x,lengthy_calculation y]&]`), or return
another aggregate (i.e., a matrix or a tuple).

Finally, note that when the arguments of a call involving `datasource` change
(because they depend on other cells which may have been updated), the
computation is automatically restarted with the new parameters. The default
behaviour in this case is that the entire computation will be redone from
scratch, but it's also possible to wrap up calls to `datasource` in a manner
which enables more elaborate communication between Gnumeric and background
tasks initiated with `datasource`. This is beyond the scope of this manual,
however, so we leave this as an exercise to the interested reader.

### Triggers

In addition to asynchronous data sources, the `trigger` function is provided
to compute values or take actions depending on some external condition, such
as the availability of data on a special device or the creation of some widget
(see the next section):

    extern expr* pure_trigger(int timeout, expr* cond, expr *val, expr *data)
      = trigger;

Thus a typical invocation of the function looks as follows:

    trigger timeout condition value data

The `condition` and `value` arguments are callback functions which get invoked
by `trigger`, passing them the given `data` argument. The trigger reevaluates
the given condition in regular intervals (1 second in the current
implementation) and, as soon as it becomes `true`, computes the given value
and returns that value as the result of the `trigger` call. As long as the
condition doesn't hold, `trigger` returns a `#N/A` value (`gnm_error "#N/A"`).
Note that, in difference to `datasource`, both the condition and the value are
computed in Gnumeric (rather than a child process), so that it is possible to
access the current information in the loaded spreadsheet.

The `timeout` value determines how often the condition is checked. If it is
positive, the condition will be reevaluated `timeout+1` times (once initially,
and then once per second for a total duration of `timeout` seconds). If it is
negative, the trigger never times out and the condition will be checked
repeatedly until the trigger expression is removed (or Gnumeric is exited). In
either case `value data` will be recomputed each time `condition data` yields
`true`. (This is most useful if the computed value, as a side effect, arranges
for the condition to become `false` again afterwards.) Finally, if `timeout`
is zero then the trigger fires at most once, as soon as the condition becomes
`true`, at which point `value data` is computed just once.

Here's a (rather useless) example of a trigger which fires exactly once, as
soon as a certain cell goes to a certain value, and then modifies another cell
value accordingly:

    > trigger 0 (\_->get_cell "A14"==="Hello") (\_->set_cell "A15" "World") ()

Now, as soon as you type `Hello` in the cell A14, the trigger will print
`World` in cell A15. Note that the `data` argument isn't used here. A more
useful example will be discussed in the following section.

### Sheet Objects

Gnumeric offers some kinds of special objects which can be placed on a sheet.
This comprises the chart and image objects which can be found in the "Insert"
menu, as well as a number of useful graphical elements and GUI widgets on the
"Object" toolbar, accessible via "View/Toolbars". The latter are also useful
for providing control input to Pure functions.

Gnumeric/Pure provides the following function to retrieve information about
the special objects in a spreadsheet:

    extern expr* pure_sheet_objects() = sheet_objects;

For instance, with one button object in your spreadsheet, the output of
`sheet_objects` might look like this:

    > sheet_objects
      [("Sheet1","button","Push Me!","A11",[#<pointer 0x2a1dcd0>])]

Each object is described by a tuple which lists the name of the sheet on which
the object is located, the type of object, the object's content or label (if
applicable), the cell which the object is linked to (if applicable), and a
list of pointers to the corresponding `GtkWidgets` (if any). Note that in
general a GUI object may be associated with several widgets, as Gnumeric
allows you to have multiple views on the same spreadsheet, so there will be
one widget for each view an object is visible in. Also note that the
content/label information depends on the particular type of object:

-   List and combo widgets return the content link (referring to the cells in
    the spreadsheet holding the items shown in the list).
-   Frame and button widgets return the label shown on the widget.
-   Graphic objects like rectangles and ellipses return the text content of
    the object.
-   Image objects (type `"image/xyz"`, where `xyz` is the type of image, such
    as `svg` or `png`) return a pointer to the image data in this field.

The `sheet_objects` function is a bit tricky to use, since some of the objects
or their associated widgets might not have been created yet when the
spreadsheet is loaded. Therefore it is necessary to use a trigger to make sure
that the information is updated once all objects are fully displayed. The
`pure_func.pure` script contains the following little wrapper around
`sheet_objects` which does this:

    pure_objects = trigger 0 (\_->all realized sheet_objects)
      (\_->matrix$map list sheet_objects) ()
    with realized (_,_,_,_,w) = ~listp w || ~null w && ~any null w end;

See the `widgets.gnumeric` spreadsheet in the distribution for an example.

Possible uses of this facility are left to your imagination. Using Gnumeric's
internal APIs and Pure's Gtk interface, you might manipulate the GUI widgets
in various ways (add icons to buttons or custom child widgets to frames,
etc.).