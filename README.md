# `argparse2tool`

Galaxy argparse aims to be a drop-in replacement for argparse, quite literally.
You can write use your normal import

```python
import argparse
```

and continue writing code. All functions are passed straight through to
argparse, but `argparse2tool` captures them and copies some information along the
way. This information captured is used to produce [Galaxy Tool XML](https://github.com/erasche/galaxyxml) when it's
requested with the `--generate_galaxy_xml` flag, or [CWL Tools](http://www.commonwl.org/v1.0/CommandLineTool.html) when requested
with the `--generate_cwl_tool` flag.

For our [example python script](./example.py) you can see the generated [Galaxy
XML](./test/example.xml) and [CWL Tools](./test/example.cwl).

## Running

To generate XML or CWL, run your tool with the appropriate command line flag

```console
$ <tool command> --generate_galaxy_xml <other options> > tool.xml
$ <tool command> --generate_cwl_tool <other options> > tool.cwl
```

The project inclues a sample `example.py` file which uses as many argparse features as possible. CWL and Galaxy XML support different portions feature sets which will be visible in the generated outputs.

```console
$ python example.py --generate_galaxy_xml
$ python example.py --generate_cwl_tool
```

### CWL Specific Functionality

Example for [CNVkit](https://github.com/etal/cnvkit) toolkit

```console
$ cnvkit.py batch --generate_cwl_tool -d ~/cnvkit-tools/ --generate_outputs
```

If there are subcommands in the provided command, all possible tools will be generated, for instance, for CNVkit

```console
$ cnvkit.py --generate_cwl_tool
```

will produce CWL tool descriptions for `cnvkit.py batch`, `cnvkit.py access`, `cnvkit.py export bed`, `cnvkit.py export cdt` and all other subcommands.

Other options (which work only with `--generate_cwl_tool` provided, except for help message) are:

* `-o FILENAME`, `--output_section FILENAME`: File with manually filled output section which is put to a formed CWL tool. `argparse2tool` is not very good at generating outputs, it recognizes output files only if they have type `argparse.FileType('w')`, so output section is often blank and should be filled manually.

* `-go`, `--generate_outputs`: flag for generating outputs not only from arguments that are instances of `argparse.FileType('w')`, but also from every argument which contains `output` keyword in its name. For instance, argument `--output-file` with no type will also be placed to output section. However, '--output-directory' argument will also be treated like File, so generated tools must be checked carefully if when this option is selected.

* `-b`, `basecommand`: command which appears in `basecommand` field in a resulting tool. It is handy to use this option when you run tool with shebang, but want `python` to be in `basecommand` field and the file amidst arguments.
Example:

	```$ .search.py --generate_cwl_tool -b python```. 

Basecommand of the formed tool will be `['python']`, and `search` will be a positional argument on position 0.

* `-d`, `--directory`: directory for storing tool descriptions.

* `--help_arg2cwl`: prints this help message.


## How it works

Internally, `argparse2tool`, masquerading as `argparse` attempts to find and
import the **real** argparse. It then stores a reference to the code module for
the system argparse, and presents the user with all of the functions that
stdlib's argparse provides. Every function call is passed through the system
argparse. However, argparse2tool captures the details of those calls and when Tool
XML is requested, it builds up the tool definition according to IUC tool
standards.

## Examples

You can see the `example.py` file for an example with numerous types of
arguments and options that you might see in real tools. Accordingly there is an `example.xml` file with the output.

## It doesn't work!!

If you are not able to use the `--generate_galaxy_xml`/`--generate_cwl_tool`
flags after installing, it is probably because of module load order. `argparse2tool`
must precede `argparse` in the path. Certain cases seem to work correctly
(`python setup.py install` in a virtualenv) while other cases do not (`pip
install argparse2tool`).

To easily correct this, run the tool `argparse2tool_check_path` which is installed
as part of this package. Correctly functioning paths will produce the
following:

```console
$ argparse2tool_check_path
Ready to go!
```

while incorrectly ordered paths will produce a helpful error message:

```console
$ argparse2tool_check_path
Incorrect ordering, please set

    PYTHONPATH=/home/users/esr/Projects/test/.venv/local/lib/python2.7/site-packages

```

This can even be used inline:

```console
user@host:$ PYTHONPATH=$(argparse2tool_check_path -q) python my_script.py --generate_galaxy_xml
```

## Limitations

This code doesn't cover the entirety of the `argparse` API yet, and there are some bugs to work out on the XML generation side:

- argparse
    - groups not supported (in galaxy, everything should still work in argparse)
    - some features like templating of the version string (please submit bugs)
- galaxyxml
    - bugs in conditionals/whens (probably)
- argparse2tool Galaxy XML Output
    - support declaring output files in an `argparse`-esque manner
- argparse2tool CWL Output
	- Some of argparse features can not be ported to CWL.
		1. `nargs=N`. Number of arguments can not be specified in CWL (yet).
		2. `const` argument of `add_argument()`. All constants must be specified in job files.
		3. Custom types and custom actions are not supported.
		4. Argument groups don't work in CWL as arguments are sorted with a [special algorithm](http://www.commonwl.org/draft-3/CommandLineTool.html#Input_binding)
		5. Mutual exclusion is not supported.

## License

Apache License, v2
