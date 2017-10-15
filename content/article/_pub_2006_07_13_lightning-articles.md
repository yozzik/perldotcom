{
   "authors" : [
      "chromatic",
      "phil-crow",
      "josh-mcadams",
      "steven-schubiger"
   ],
   "slug" : "/pub/2006/07/13/lightning-articles.html",
   "title" : "Still More Perl Lightning Articles",
   "description" : "Using Module::Build, using Swing from Perl, turning modules into scripts, adding mocks to test fixtures-it's time for more Perl lightning articles!",
   "thumbnail" : "/images/_pub_2006_07_13_lightning-articles/111-perl-lit-art.gif",
   "draft" : null,
   "tags" : [
      "java-swing",
      "mock-objects",
      "module-build-convert",
      "perl-lightning-articles",
      "perl-modules",
      "perl-scripts",
      "perl-tricks",
      "refactoring-tests",
      "test-organization",
      "test-class",
      "test-mockobject-extends"
   ],
   "image" : null,
   "categories" : "development",
   "date" : "2006-07-13T00:00:00-08:00"
}





It has been common practice within the Perl community for ages to ship
distributions with a *Makefile.PL* so that the user will be able to
install the packages when he retrieves them, either via the shell which
the `CPAN/CPANPLUS` modules offer or via manual CPAN download.

The *Makefile.PL* consists of meta-information, which in the case of the
distribution
[`HTML::Tagset`](http://search.cpan.org/perldoc?HTML::Tagset) is:

     # This -*-perl-*- program writes the Makefile for installing this distribution.
     #
     # See "perldoc perlmodinstall" or "perldoc ExtUtils::MakeMaker" for
     # info on how to control how the installation goes.

     require 5.004;
     use strict;
     use ExtUtils::MakeMaker;

     WriteMakefile(
         NAME            => 'HTML::Tagset',
         AUTHOR          => 'Andy Lester <andy@petdance.com>',
         VERSION_FROM    => 'Tagset.pm', # finds $VERSION
         ABSTRACT_FROM   => 'Tagset.pm', # retrieve abstract from module
         PMLIBDIRS       => [qw(lib/)],
         dist            => { COMPRESS => 'gzip -9f', SUFFIX => 'gz', },
         clean           => { FILES => 'HTML-Tagset-*' },
     );

Of interest are the arguments to `WriteMakefile()`, because they
influence the *Makefile* written by `ExtUtils::MakeMaker` after the user
has invoked the usual build and install procedure:

     % perl Makefile.PL
     % make
     % make test
     # make install

### `Module::Build`, Successor of `ExtUtils::MakeMaker`?

As Ken Williams grew tired of
[`ExtUtils::MakeMaker`](http://search.cpan.org/perldoc?ExtUtils::MakeMaker)
and its portability issues, he invented
[`Module::Build`](http://search.cpan.org/perldoc?Module::Build), a
successor of `ExtUtils::MakeMaker`. One goal of `Module::Build` is to
run smoothly on most operating systems, because it takes advantage of
creating Perl-valid syntax files only and does not rely upon crufty
*Makefiles*, which are often subject to misinterpretation, because so
many incompatible flavors of `make` exist in the wild.

The current maintainer of `ExtUtils::MakeMaker`, Michael G. Schwern,
elaborated about this problem in his talk reachable via "[MakeMaker is
DOOMED](http://mungus.schwern.org/~schwern/talks/MakeMaker_Is_DOOMED/slides/)."

### `Module::Build` Distribution "Skeleton"

If you take in consideration the distribution `HTML::Tagset` again, the
rough skeleton suitable for `Module::Build` having converted the
*Makefile.PL* by
[`Module::Build::Convert`](http://search.cpan.org/perldoc?Module::Build::Convert)
into a *Build.PL*, the output would be:

     # This -*-perl-*- program writes the Makefile for installing this distribution.
     #
     # See "perldoc perlmodinstall" or "perldoc ExtUtils::MakeMaker" for
     # info on how to control how the installation goes.
     # Note: this file has been initially generated by Module::Build::Convert 0.24_01

     require 5.004;
     use strict;
     use warnings;

     use Module::Build;

     my $build = Module::Build->new
       (
        module_name => 'HTML::Tagset',
        dist_author => 'Andy Lester <andy@petdance.com>',
        dist_version_from => 'Tagset.pm',
        add_to_cleanup => [
                            'HTML-Tagset-*'
                          ],
        license => 'unknown',
        create_readme => 1,
        create_makefile_pl => 'traditional',
       );
      
     $build->create_build_script;

As you can see, while `ExtUtils::MakeMaker` prefers uppercased
arguments, `Module::Build` goes by entirely lowercased arguments, which
obey the rule of least surprise by being as intuitive as a description
can be.

The build and installation procedure for a `Module::Build` distribution
is:

     % perl Build.PL
     % perl Build
     % perl Build test
     # perl Build install

### `Module::Build::Convert`'s State of Operation

`Module::Build::Convert` actually does all of the background work and
can be safely considered the back end, whereas `make2build` is the
practical front-end utility. `Module::Build::Convert` currently exposes
two kinds of operation: static approach and dynamic execution. The
static approach parses the arguments contained within the
*Makefile.PL's* `WriteMakefile()` call, whereas dynamic execution runs
the *Makefile.PL* and captures the arguments provided to
`WriteMakefile()`.

`Module::Build::Convert` parses statically by default, because the
dynamic execution has the downside that code will be interpreted and the
interpreted output will be written to the *Build.PL*, so you have to
conclude that the user of the distribution will end up with predefined
values computed on the author's system. This is something to avoid,
whenever possible! If the parsing approach fails, perhaps looping
endlessly on input, `Module::Build::Convert` will reinitialize to
perform dynamic execution of the *Makefile.PL* instead.

### [Data Section]{#data_section}

`Module::Build::Convert` comes with a rather huge data section
containing the argument conversion table, default arguments, sorting
order, and begin and end code. If you wish to change this data, consider
making a *\~/.make2buildrc* file by launching `make2build` with the
`-rc` switch. *Do not* edit the `Data` section within
`Module::Build::Convert` directly, unless you are sure you want to
submit a patch.

#### Argument Conversion

On the left-hand side is the `MakeMaker`'s argument name, and on the
right-hand side the `Module::Build`'s equivalent.

     NAME                  module_name
     DISTNAME              dist_name
     ABSTRACT              dist_abstract
     AUTHOR                dist_author
     VERSION               dist_version
     VERSION_FROM          dist_version_from
     PREREQ_PM             requires
     PL_FILES              PL_files
     PM                    pm_files
     MAN1PODS              pod_files
     XS                    xs_files
     INC                   include_dirs
     INSTALLDIRS           installdirs
     DESTDIR               destdir
     CCFLAGS               extra_compiler_flags
     EXTRA_META            meta_add
     SIGN                  sign
     LICENSE               license
     clean.FILES           @add_to_cleanup

#### Default Arguments

These are default `Module::Build` arguments to added. Arguments with a
leading `#` are ignored.

     #build_requires       HASH
     #recommends           HASH
     #conflicts            HASH
     license               unknown
     create_readme         1
     create_makefile_pl    traditional

#### Sorting Order

This is the sorting order for `Module::Build` arguments.

     module_name
     dist_name
     dist_abstract
     dist_author
     dist_version
     dist_version_from
     requires
     build_requires
     recommends
     conflicts
     PL_files
     pm_files
     pod_files
     xs_files
     include_dirs
     installdirs
     destdir
     add_to_cleanup
     extra_compiler_flags
     meta_add
     sign
     license
     create_readme
     create_makefile_pl

#### Begin Code

Code that precedes converted `Module::Build` arguments. `$(UPPERCASE)`
are stubs being substituted by `Module::Build` code.

     use strict;
     use warnings;

     use Module::Build;

     $MAKECODE

     my $b = Module::Build->new
     $INDENT(

#### End Code

Code that follows converted `Module::Build` arguments. `$(UPPERCASE)`
are stubs being substituted by `Module::Build` code.

     $INDENT);

     $b->create_build_script;

     $MAKECODE

### `make2build` Basic Usage

Using `make2build` is as easy as launching it in the directory of the
distribution of which *Makefile.PL* you wish to convert.

For example:

    % make2build

You may also provide the full path to the distribution, assuming, for
example, you didn't `cd` directly into the distribution directory.

    % make2build /path/to/HTML-Tagset*

In both cases, the command will convert any found *Makefile.PL* files
and will generate no output because `make2build` acts quiet by default.

### `make2build` Switches

As `make2build` aims to be a proper script, it of course, provides both
the `-h` (help screen) and `-V` (version) switches.

     % make2build -h
     % make2build -V

In case you end up with a mangled *Build.PL* written, you can examine
the parsing process by launching `make2build` with the `-d` switch,
enabling the pseudo-interactive debugging mode.

     % make2build -d

Should you not like the indentation length or judge it to be too small,
increase it via the `-l` switch followed by an integer.

     % make2build -l length

If you don't agree with the sorting order predefined in
`Module::Build::Convert`, you may enforce the native sorting order,
which strives to arrange standard arguments with those seen available in
the `Makefile.PL`.

     % make2build -n

The argument conversion table, default arguments to add, the sorting
order of the arguments, and the begin and end code aren't absolute,
either. Change them by invoking `make2build` with the `-rc` switch to
create a resource configuration file in the home directory of the
current user; that is likely *\~/.make2build.rc*.

     % make2build -rc

While `make2build` is quiet by default, there are two verbosity levels.
To enforce verbosity level 1, launch `make2build` with `-v`. To enforce
verbosity level 2, use `-vv`.

With `-v`, the code will warn about *Makefile.PL* options it does not
understand or skips. With `-vv`, it will accumulate `-v` output and the
entire generated *Build.PL*.

     % make2build -v
     % make2build -vv

You may execute the *Makefile.PL* in first place, but such usage is
deprecated because `Module::Build::Convert` downgrades automatically
when needed.

     % make2build -x (deprecated)

### Swinging with Perl

Phil Crow

Perl does not have a native graphical user interface (GUI) toolkit. So
we use all manner of existing GUI tools in front of our Perl
applications. Often we use a web browser. We have long had Perl/Tk and
other libraries based on C/C++. Now we can also use Java's Swing toolkit
with similar ease.

In my sample application, when the user presses a button, Perl evaluates
an arithmetic expression from the input text box. The result appears in
another text box. I'll show the code for this application a piece at a
time with a discussion after each piece. To see the whole thing, look in
the examples directory of the
[`Java::Swing`](http://search.cpan.org/perldoc?Java::Swing)
distribution.

        #!/usr/bin/perl
        use strict; use warnings;

        BEGIN {
            $ENV{CLASSPATH} .= ':/path/to/Java/Swing/java'
        }

`Java::Swing` needs certain Java classes to be in the class path before
it loads, so I've appended a path to those classes in a `BEGIN` block
(this block must come before using `Java::Swing`).

        use Java::Swing;

This innocuous statement magically sets up namespaces for each Java
Swing component, among other things.

        my $expression  = JTextField->new();
        my $answer      = JTextField->new( { columns => 10 } );
        my $submit      = JButton   ->new("Evaluate");
        my $frame       = JFrame    ->new();
        my $root_pane   = $frame->getContentPane();
        my $south_panel = JPanel->new();

After using `Java::Swing`, you can refer to Swing components as Perl
classes. You can even pass named parameters to their constructors, as
shown for the second `JTextField`.

        $south_panel->add(JLabel->new("Answer:"), "West");
        $south_panel->add($answer,                "Center");
        $south_panel->add($submit,                "East");

        $root_pane->add($expression,  "North");
        $root_pane->add($south_panel, "South");

        $frame->setSize(300, 100);
        $frame->show();

Most work with the components is the same as in any Java program. If you
don't understand the above code, consult a good book on Swing (like the
one from O'Reilly).

        my $swinger = Java::Swing->new();

This creates a `Java::Swing` instance to connect event listeners and to
control the event loop.

        $swinger->connect(
            "ActionListener", $submit, { actionPerformed => \&evaluate }
        );

        $swinger->connect(
            "WindowListener", $frame, { windowClosing => \&ending }
        );

Connection is simple. Pass the listener type, the object to listen to,
and a hash of code references to call back as events arrive.

        $swinger->start();

Start the event loop. After this, the program passively waits for event
callbacks. It stops when one of the callbacks stops the event loop.

        sub evaluate {
            my $sender_name = shift;
            my $event       = shift;

            $answer->setText(eval $expression->getText());
        }

My evaluation is simple. I retrieve the text from the expression
`JTextField`, `eval` it, and pass the result to `setText` on the answer
`JTextField`. Using `eval` raises possible security concerns, so use it
wisely.

        sub ending {
            $swinger->stop();
        }

When the user closes the window, I stop the event loop by calling `stop`
on the `Java::Swing` instance gained earlier. This kills the program.

With `Java::Swing`, you can build Swing apps in Perl with some important
bits of syntactic sugar. First, you don't need to have separate Java
files or inline sections. Second, you can pass named arguments to
constructors. Finally, you can easily connect event listeners to Perl
callback code.

### Scriptify Your Module

Josh McAdams

Recently during an MJD talk at Chicago.pm, I saw a little Perl trick
that was so amazingly simple and yet so useful that it was hard to
believe that more mongers in the crowd hadn't heard of it. The trick
involved taking your module and adding a driver routine to it so the
module could run as a script.

To illustrate, start with an example module that contains two utility
subroutines that convert weights between pounds and kilograms. The
subroutines accept some number and multiplies it by a conversion factor.

      package WeightConverter;
      
      use strict;
      use warnings;
      use constant LB_PER_KG => 2.20462262;
      use constant KG_PER_LB => 1/LB_PER_KG;
      
      sub kilograms_to_pounds { $_[0] * LB_PER_KG; }
      
      sub pounds_to_kilograms { $_[0] * KG_PER_LB; }

Assuming that the real module has a little error checking and POD, this
module would serve you just fine. However, what if you decided that we
needed to be able to easily do weight conversions from the command line?
One option would be to write a Perl script that used `WeightConverter`.
If that seems like too much effort, there is a one-liner that would do
conversions.

      perl -MWeightConverter -e 'print WeightConverter::kilograms_to_pounds(1),"\n"'

This would do the trick, but it is a lot to remember and isn't very fun
to type. There is a lot of benefit available from saving some form of
script, and believe it or not, the module can hold that script. All that
you have to do is write some driver subroutine and then call that
subroutine if the module is not being used by another script. Here is an
example driver for `WeightConverter`.

This example driver script just loops through the command-line arguments
and tries to find instances where the argument contains either a `k` or
`p` equal to some value. Based on whether or not you are starting with
pounds or kilograms, it calls the appropriate subroutine and prints the
results.

      sub run {
        for (@ARGV) {
          if(/^[-]{0,2}(k|p)\w*=(.+)$/) {
            $1 eq 'k' ?
              print "$2 kilograms is ", kilograms_to_pounds($2), " pounds\n" :
              print "$2 pounds is ", pounds_to_kilograms($2), " kilograms\n" ;
          }
        }
      }

Now all that is left is to tell the module to run the `run` subroutine
if someone has run the module on its own. This is as easy as adding one
line somewhere in the main body of the module.

      run unless caller;

All this statement does is execute the `run` subroutine unless the
`caller` function returns a value. `caller` will only return true if
`WeightConverter` is being used in another script. Now, this module is
usable in other scripts as well as on the command line.

      $> perl WeightConverter.pm -kilos=2 -pounds=145 -k=.345
      2 kilograms is 4.40924524 pounds
      145 pounds is 65.7708937051548 kilograms
      .345 kilograms is 0.7605948039 pounds

### Mocks in Your Test Fixtures

by chromatic

Since writing `Test::MockObject`, I've used it in nearly every complex
test file I've written. It makes my life much easier to be able to
control only what I need for the current group of tests.

I wish I'd written `Test::MockObject::Extends` earlier than I did; that
module allows you to decorate an existing object with a mockable
wrapper. It works just as the wrapped object does, but if you add any
mocked methods, it will work like a regular mock object.

This is very useful when you don't want to go through all of the
overhead of setting up your own mock object but do want to override one
or two methods. (It's almost always the right thing to do instead of
using `Test::MockObject.`.)

Another very useful test module is `Test::Class`. It takes more work to
understand and to use than `Test::More`, but it pays back that
investment by allowing you to group, reuse, and organize tests in the
same way you would group, reuse, and organize objects in your code.
Instead of writing your tests procedurally, from the start to the end of
a test file, you organize them into classes.

This is most useful when you've organized your code along similar lines.
If you have a base class with a lot of behavior and a handful of
subclasses that add and override a little bit of behavior, write a
`Test::Class`-based test for the base class and smaller tests that
inherit from the base test for the subclasses.

Goodbye, duplicate code.

#### Fixtures

`Test::Class` encourages you to group related tests into test methods.
This allows you to override and extend those groups of tests in test
subclasses. (Good OO design principles apply here; tests are still just
code, after all.) One of the benefits of grouping tests in this way is
that you can use test fixtures.

A test fixture is another method that runs before every test method. You
can use them to set up the test environment--creating a new object to
test, resetting test data, and generally making sure that tests don't
interfere with each other.

A standard test fixture might resemble:

      sub make_fixture :Test( setup )
      {
          my $self        = shift;
          $self->{object} = $self->test_class()->new();
      }

Assuming that there's a `test_class()` method that returns the name of
the class being tested, this fixture creates a new instance before every
test method and stores it as the `object` attribute. The test methods
can then fetch this as normal.

#### Putting Them Together

I recently built some tests for a large system using `Test::Class`. Some
of the tests had mockable features--they dealt with file or database
errors, for example. I found myself creating a lot of little
`Test::MockObject::Extends` instances within most of the tests.

Then inspiration struck. Duplication is bad. Repetition is bad. Factor
it out into one place.

The insight was quick and sudden. If `Test::MockObject::Extends` is
transparent (and if it isn't, please file a bug--I'll fix it), I can use
it in the test fixture all the time and then be able to mock whenever I
want without doing any setup. I changed my fixture to:

      sub make_fixture :Test( setup )
      {
          my $self        = shift;
              my $object      = $self->test_class()->new();
          $self->{object} = Test::MockObject::Extends->new( $object );
      }

The rest of my code remained unchanged, except that now I could delete
several identical lines from several test methods.

Do note that, for this to work, you must adhere to good OO design
principles in the code being tested. Don't assume that `ref` is always
what you think it should be (and use the `isa()` method instead).

Sure, this is a one-line trick, but it removed a lot of busy work from
my life and it illustrates two interesting techniques for managing
tests. If you need simpler, more precise mocks, use
`Test::MockObject::Extends`. If you need better organization and less
duplication in your test files, use `Test::Class`. Like all good test
modules, they work together almost flawlessly.

