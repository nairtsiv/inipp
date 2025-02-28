# IniPP

Simple header-only C++ ini parser and generator.

[![Build Status](https://github.com/mcmtroffaes/inipp/actions/workflows/build.yml/badge.svg)](https://github.com/mcmtroffaes/inipp/actions/workflows/build.yml)

## Features

* Header-only.
* Both parsing and generating.
* Wide character support for native unicode on Windows.
* Default section support (similar to Python's ConfigParser).
* Interpolation support (i.e. variable substitution, similar to Python's ConfigParser).
* Trailing comments support.
* Simple design and implementation.
* Permissive MIT license.

## Example

```cpp
#include <fstream>
#include "inipp.h"

int main() {
	inipp::Ini<char> ini;
	std::ifstream is("example.ini");
	ini.parse(is);
	std::cout << "raw ini file:" << std::endl;
	ini.generate(std::cout);
	ini.strip_trailing_comments();
	ini.default_section(ini.sections["DEFAULT"]);
	ini.interpolate();
	std::cout << "ini file after default section and interpolation:" << std::endl;
	ini.generate(std::cout);
	int compression_level = -1;
	inipp::get_value(ini.sections["bitbucket.org"], "CompressionLevel", compression_level);
	std::cout << "bitbucket.org compression level: " << compression_level << std::endl;
	return 0;
}
```

To include in a cmake project:

```
FetchContent_Declare(inipp GIT_REPOSITORY https://github.com/mcmtroffaes/inipp.git)
FetchContent_MakeAvailable(inipp)
target_link_libraries(MyTarget PRIVATE inipp::inipp)
```

## Parsing algorithm

* The *section* is set to the empty string.

* Every *line* is read from the file and trimmed from whitespace.

    * If *line* is empty or starts with ``;`` then nothing happens.

    * Otherwise, if *line* starts with ``[`` then *section* is changed
      to the string between ``[`` and ``]``. If *line* does not end
      with ``]`` then an error is reported.

    * Otherwise, if *line* contains an ``=`` sign, then all characters
      before ``=`` are treated as *variable* and all characters
      following ``=`` are treated as *value*. Both are trimmed. If the
      variable was already assigned earlier, an error is
      reported. Otherwise, the corresponding assigment is added to the
      *section*.

    * Otherwise, the *line* is reported as an error.

## Default section algorithm

Insert every variable from the default section into every other section, without overwriting existing variables.

## Interpolation algorithm

1. Locally within each section, every occurrence "${variable}" is replaced by "${section:variable}".
2. Every occurrence of "${section:variable}" is replaced by its value.
3. The previous step is repeated until no more replacements are possible, or until the recursion depth (by default, 10) is reached.

