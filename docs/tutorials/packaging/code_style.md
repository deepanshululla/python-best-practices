!!! Summary

    :white_check_mark: Use [ruff] or [black] code formatting.

    :white_check_mark: Use either [napoleon] docstring type annotations or [PEP-484] built-in type annotations.
    
    :white_check_mark: Use [ruff] for linting (static code analysis).

    :white_check_mark: Use [mypy] for type checking.


# Code Style

A consistent code style makes code easier to read and understand. The main 
benefit of increased legibility and beautiful code is saving developer time. A 
consistent code style allows anyone to quickly familiarize themselves with a 
codebase even if it is being worked on by many people. A recommended read is the [clean code python]. 

There are many tools/guidelines which can be used to for code style in python 
so a common question is which one to use. The reality is that most of the 
tools/guidelines are fairly good and have generally useful default behavior. 

In practice, for most tools, warnings end up being ignored or specific 
linting options are turned off because they are too restrictive. If a developer 
of a repository disagrees with an option, then they will just remove it 
entirely so that it does not bother them anymore.

When using a set of guidelines which does not have a tool associated with it, 
rules are often forgotten or implemented differently by different developers.

## Goals

Due to the reasons above, the goal of a styling specification would be to 
have the following properties:

- **Automation**: The code style must have a tool. It must have automated 
    formatting so that users do not need to worry about styling the code 
    manually.
- **Minimal Configuration**: Minimize options so that users do not need to make
    decisions about style. More decisions leads to more configuration and more
    inconsistencies in code style.
- **Customizable Rules**: The tool should allow leads to set the type of rules
    going to be enforced and rules can be ignored.

## Example Tools

The following are the most commonly used tools for python code quality and ensure consistency.

### Linters

Linters are usually broken into two types: Logical and Stylistic linters.

Logical Linter:

- Code Errors
- Code with potentially unintended results
- Dangrous code patterns

Stylistic Liners:

- Code not conforming to defined conventions.

Below are some of the popular tools

- [ruff]; A modern(Rust) linter that combines the best features of flake8, pylint, and isort. It's highly customizable and offers fast performance.

- [flake8]: the wrapper which verifies [PEP-8], pyflakes, and circular complexity “. It has a low rate of false positives.


### Formatters

Formatters will format the actual python file based on rules.

- [ruff] A drop-in replacement for [black], but much faster.

- [black] An automated code formatter with no configuration options.

    A huge benefit of black is that unlike the previously mentioned tools this 
    does not have formatting options. Black does not strictly follow 
    [PEP-8] guidelines but is generally compliant.  It is also important to 
    understand that [black] is only an opinionated `formatter`, it does not
    check all of the styles problem that other `linters` do. 

- [isort] Format imports order.

    Sort imports alphabetically, and automatically separated into sections and by type.

In a very controlled development environment, each may be useful tools by just
enabling the default behavior and forbidding custom options.

## Recommendation

For a comprehensive and efficient code style and linting solution, we highly recommend using `ruff`. It offers a powerful combination of features and performance benefits:

- **Fast Performance**: Leverages Rust for speed, making it ideal for large codebases.
- **Comprehensive Linting**: Checks for a wide range of style issues, potential bugs, and performance problems.
- **Customization**: Easily tailor the linting rules to your specific needs.
- **Compatibility**: Seamlessly integrates with popular tools like `black` and `isort`.

For formatting, `black` remains a solid choice for its simplicity and strict adherence to a specific style. However, for those seeking a faster alternative, `ruff` can also be used as a drop-in replacement.

To ensure type safety, `mypy` should be used to statically type-check your code.


## Style Guides

Beyond automated checks and warnings for code, style guides provide good 
recommendations for how you should write code.

Three style guides in particular provide almost universally useful advice:

- [PEP-8]: The official python style guide provides the basis for almost all 
  formatters and other style guides. This provides the broadest and most 
  foundational rules for writing legible python code.  
- [Google Style Guide]: This has a lot of useful recommendations, however many of 
  their guidelines were chosen to maximize compatibility between python 2 and 
  python 3. Since python 2 is officially deprecated, all of the compatibility
  guidelines should be ignored. For these reasons the main
- [Hitchhiker's Guide to Python]: This provides its own style guide which
  is universally regarded as good practice.

Though none of these style guides can programmatically format code for you, they
are essential reference for how to write clean, simple python code.

## Type Annotations

Type annotations of some kind are recommended but can be implemented in 
different ways. Some form of type annotation is always recommended so that
code usage is less ambiguous.

The recommended strategies for annotating the types of your code:

### [PEP-484] Style type annotations

This places the type annotation directly in code. 

Pros:

- Code can be checked using automated code checkers (See: [mypy] & [pyre-check])

Cons:

- Tends to make function declarations verbose
- Can be difficult to document data structures (Requires importing [typing])

```python
def subtract(minuend: int, subtrahend: int) -> int:
    """
    Subtract the subtrahend from the minuend.
    
    Args:
        minuend: The basis to subtract from.
        subtrahend: The value to subtract.
    Result:
        difference: The difference between the numbers.
    """
    return minuend - subtrahend
```


## Variable Naming

In the style guides mentioned above, the naming conventions are primarily
concerned with the case used (snake_case vs CamelCase, etc.) However, this
section is in regards to the actual words which should be used in a name.

Naming is one of the hardest problems in programming. Good naming can 
drastically increase the readability of your code. To name things well, 
variables should have the following properties:

- Descriptive
- Unambiguous
- Should not contain the type. This is what type annotations are for
- Should be short if possible. Long names make code more difficult to read

Abbrivations for names is ok if it is well-known but should be reframed.
ie. `n`, `sz`, `cnt`, `idx`, `dt`, `ts`, `env`, `cfg`, `ctx` etc. 

### Dictionaries

Dictionaries should have information about both the **key and value** in 
the name. 

Acceptable Forms:

- `{singular-key}_{plural-values}` *(Preferred) Somewhat ambiguous, but succinct*
- `{key}_to_{value}` *Somewhat verbose*
- `{value}_by_{key}` *Somewhat verbose, less direct than `{key}_to_{value}`*
- `{key}_{value}_lookup` *Somewhat verbose, but describes how it should be used*

Each of the forms can be more appropriate in different scenarios where the 
meaning is more clear in context.

!!! success "Use"

    ```python
    word_lengths = {"hello": 5, "world": 5}
    index_to_word = {1: "hello", 2: "world"}
    word_by_index = {1: "hello", 2: "world"}
    index_word_lookup = {1: "hello", 2: "world"}
    ```

!!! fail "Avoid"

    ```python
    words = {1: "hello", 2: "world"}        # Ambiguous, No key information
    word_lookup = {1: "hello", 2: "world"}  # No key information
    word_dict = {1: "hello", 2: "world"}    # Type included in name
    ```

In some cases there are well-understood mappings that are meant to be iterated
over. In these cases it makes sense to just use the pluralized version of the 
word and name the variable after the contents. Anything that is meant to be 
iterated over should be pluralized.

!!! attention "Exception"

    ```python
    headers = {"Content-Type": "application/json"}
    cookies = {"tz": "America/Los_Angeles"}
    hyperparameters = {"min_samples_leaf": 50}
    gunicorn_options = {"workers": 8} # Gunicorn uses `options`
    ```
    
!!! fail "Avoid"

    ```python
    header_name_to_value = {"Content-Type": "application/json"}  # Too verbose
    cookie_name_to_value = {"tz": "America/Los_Angeles"}         # Too verbose
    hyperparameter_name_to_value = {"min_samples_leaf": 50}      # Too verbose
    ```

In other cases, there are libraries that have predefined names for their 
arguments that do not follow the conventions above. In this case it is 
acceptable to follow their conventions when interacting with their code. This
makes the code less ambiguous because the library name and the application-code 
variable name match.

!!! attention "Exception"

    ```python
    feed_dict = {"name": tensor} # TensorFlow uses this name so it is acceptable
    ```

### Lists/Series/Arrays/Sets

Collections (non-dictionary) should always be the plural version of whatever is 
contained within. In the case where the value type is ambiguous, try to name
the collection so it is possible to determine what is contained within.

!!! success "Use"

    ```python
    zip_codes = [92127, 12345]
    names = {"Johnny", "Lisa", "Mark", "Denny"}
    column_names = ["zip_code", "wages"] # `names` suffix indicates string value
    ``
    
!!! fail "Avoid"

    ```python
    zip_list = [92127, 12345]        # Do not include type
    list = [92127, 12345]            # Shadows built-in list, Ambiguous
    items = [92127, 12345]           # Ambiguous
    columns = ["zip_code", "wages"]  # Value type is ambiguous
    ```

### Integers/Floats

When possible, number names should indicate the how the value should be 
used. Contrary to the rules regarding collections, there are a many 
well-understood values that indicate a number which should be unambiguous. 

!!! success "Use"

    ```python
    limit = 10
    threshold = 0.7
    size = 10       # Potentially ambiguous
    index = 10      # Potentially ambiguous
    count = 10      # Potentially ambiguous
    n_items = 10    # `n` prefix always indicates a number, preferred `num_items`
    min_items = 10  # `min` prefix always indicates a number
    max_items = 10  # `max` prefix always indicates a number
    buy_price = 10  # Domain specific words always indicate a number 
    ```

!!! fail "Avoid"

    ```python
    items = 10      # Ambiguous, could indicate a collection
    n = 10          # Not Descriptive
    n_value = 10    # `value` suffix is not descriptive
    n_quantity = 10 # `quantity` suffix is not descriptive
    ```

### Strings

When possible, string names should indicate how the value should be used. The
naming conventions of strings are similar to the conventions of numbers. There 
are many well-understood names that could indicate a string type.

Common Forms:

- `{descriptor}_key` *May indicate lookup key*
- `{descriptor}_name` *May indicate lookup key*

!!! success "Use"

    ```python
    environment_name = 'cdev'
    environment_key = 'cdev'
    environment = 'cdev'      # Well-known string value, Potentially ambiguous
    env = 'cdev'              # Well-known abbreviation, Potentially ambiguous
    ```

!!! fail "Avoid"

    ```python
    value = 'cdev'  # Ambiguous
    config = 'cdev' # Ambiguous could indicate an object
    ```

### Classes

Class instances should always be named after the class itself. For the purposes
of describing the forms which a variable should be named the following class
name will be used `DescriptionNoun`.

Acceptable Forms:

- `{description}_{noun}` *(Preferred) CamelCase to snake_case conversion*
- `{noun}` *More succinct, potentially ambiguous*

!!! success "Use"

    ```python
    class ExampleContext:
        pass
    
    example_context = ExampleContext()
    context = ExampleContext()
    ```

### Functions

Function should always be named in lower case and separated with underscore (`_`). The words that you use to name your function should clearly describe the function’s intent (what the function does).  All functions should clearly indicates the input and output variable types.
When returning from function avoid generic container types like `List` or `Dict`, use additional
hints if you must return such types, ie. `List[int]` or `Dict[str, int]`.  

Acceptable Forms:

- `def {verb}_{intent}({nouns}:T) -> U: ` *(Preferred)*
- `def {verb}({noun}:T}) -> U: ` *More succinct, potentially ambiguous*

!!! success "Use"

    ```python
    def sum(iterable: Iterable) -> Number:
    ```

    ```python
    def remove_underscore(s: str) -> str:
    ```

!!! fail "Avoid"

    ```python
    def foo(a):
    ```

    ```python
    def process(parameters:Dict) -> Dict:
    ```

    ```python
    def calculate(results:Dict[Any, Dict]) -> Dict:
    ```


## Ruff Setup

It is recommend to setup ruff in your `pyproject.toml` file. 

```toml
[tool.ruff]
target-version = "py312"

# Line length configuration
line-length = 100  # Black default to 88 
indent-width = 4

# Same as Black.
[tool.ruff.format]
quote-style = "double"
indent-style = "space"
skip-magic-trailing-comma = false
docstring-code-format = true
line-ending = "auto"

[tool.ruff.lint]
select = [
    "A",  # Assignment expressions
    "ARG",  # Argument-related issues
    "B",  # Builtin usage
    "C",  # Complexity
    "COM",  # Comments
    "DTZ",  # Datetime and time zone issues
    "E",  # Pythonic style guide violations
    "EM",  # Empty blocks
    "F",  # Formatting
    "FBT",  # False positives and benign issues
    "I",  # Imports
    "ICN",  # Inconsistent naming
    "ISC",  # Inconsistent spacing
    "N",  # Naming conventions
    "PLC",  # Potential logical errors
    "PLE",  # Potential library errors
    "PLR",  # Potential runtime errors
    "PLW",  # Potential performance warnings
    "Q",  # Quality of life suggestions
    "RUF",  # Ruff-specific checks
    "TID",  # Type checker issues
    "UP",  # Unused code
    "W",  # Warnings
    "YTT",  # Yield type hints
]
ignore = [
    "FBT003",  # Ignore a specific false positive or benign issue (boolean-positional-value-in-call)
]

# Allow autofix for all enabled rules (when `--fix`) is provided.
fixable = ["ALL"]
unfixable = []

# Files to exclude
exclude = [
    ".bzr",
    ".direnv",
    ".eggs",
    ".git",
    ".git-rewrite",
    ".hg",
    ".mypy_cache",
    ".nox",
    ".pants.d",
    ".pytype",
    ".ruff_cache",
    ".svn",
    ".tox",
    ".venv",
    "__pypackages__",
    "_build",
    "buck-out",
    "build",
    "dist",
    "node_modules",
    "venv",
]


[tool.ruff.lint.per-file-ignores]
# Ignore `E402` (import violations) in all `__init__.py` files
"__init__.py" = ["E402"]
"tests/**/*" = ["D100", "D101", "D102", "D103", "D104"]

[tool.ruff.lint.mccabe]
max-complexity = 10

[tool.ruff.lint.pydocstyle]
convention = "google"  # Follows Google-style docstrings

[tool.ruff.lint.pylint]
max-args = 5  # Maximum number of arguments for functions
max-returns = 5  # Maximum number of return statements

[tool.ruff.lint.pycodestyle]
max-doc-length = 100  # Same as line length

[tool.mypy]
files = ["your_modules", "tests"]
no_implicit_optional = true
check_untyped_defs = true
```


[yapf]: https://github.com/google/yapf/
[Google-style docstrings]: http://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings
[Hitchhiker's Guide to Python]: https://docs.python-guide.org/writing/style/
[PEP-8]: https://www.python.org/dev/peps/pep-0008/
[pylint]: https://pypi.org/project/pylint/
[flake8]: https://flake8.pycqa.org/en/latest/
[PyFlakes]: https://github.com/PyCQA/pyflakes
[pycodestyle]: https://github.com/PyCQA/pycodestyle
[autopep8]: https://github.com/hhatto/autopep8
[PEP-484]: https://www.python.org/dev/peps/pep-0484/
[mypy]: http://mypy-lang.org/
[ruff]: https://github.com/astral-sh/ruff
[pyre-check]: https://pyre-check.org/
[typing]: https://docs.python.org/3/library/typing.html
[napoleon]: https://sphinxcontrib-napoleon.readthedocs.io/en/latest/#type-annotations
[Google Style Guide]: http://google.github.io/styleguide/pyguide.html
[black]: https://github.com/psf/black
[isort]: https://github.com/timothycrosley/isort
[gofmt]: https://golang.org/cmd/gofmt/
[McCabe]: https://github.com/PyCQA/mccabe
[clean code python]: https://github.com/zedr/clean-code-python
