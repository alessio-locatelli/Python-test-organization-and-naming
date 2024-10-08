# Python Test Organization and Naming

## Folder Structure

- All tests must be placed outside the application folder, in the `tests/` folder.
  ```
  src/
  tests/
  ```
- The repository may have a dedicated `testing/` folder to store files that are not part of an application, but are needed for testing (sample data, helper functions, utilities for setup/teardown of the test environment).
  ```
  src/
  testing/
  tests/
  ```
- The `tests/` folder can have several subfolders named after different types of tests:
  ```
  src/
  tests/
      unit/
      integration/
      performance/
  ```
- The `tests/` (or `tests/unit/`) folder must mirror the application folder.

  ```
  src/
      module_a/
          foo.py
          bar.py
      module_b/
          spam.py
          eggs.py
  tests/
      test_module_a/
          test_foo.py
          test_bar.py
      test_module_b/
          test_spam.py
          test_eggs.py
  ```

  If it exists, the `tests/integration/` folder should try to be consistent, buy may have a different structure.

- If you rename a file, you must also remember to use find/replace in the `tests/` folder to maintain consistency.

## File Structure

- Files in the `tests/` (or `tests/unit/`) folder must mirror the application files.

  ```py
  # src/module_a/foo.py

  def func(): ...

  def ExampleClass:
    def example_method(): ...
  ```

  ```py
  # tests/test_module_a/test_foo.py

  def test_func(): ...

  def TestExampleClass:
    def test_example_method(): ...
  ```

  If it exists, the `tests/integration/` folder should try to be consistent, buy may have a different structure.

- If you rename a function or class, you must also remember to use find/replace in the `tests/` folder to maintain consistency.

## Naming Convention

#### Naming Schema

```
    test_  +  function_name  +  _extra_information
[MANDATORY]    [MANDATORY]         [OPTIONAL]
```

Where:

- `test_` is a prefix that is required to [discover a test](https://docs.pytest.org/en/stable/explanation/goodpractices.html#conventions-for-python-test-discovery).
- `function_name` is a function, a class method, or a class name.
- `_extra_information` (usually in integration and other non-unit tests) is an optional suffix that describes a test scenario. This suffix can make a test name more verbose and provide some information about the test. In unit tests, the prefix should be avoided unless absolutely necessary. This helps to keep names short and simple.

#### Naming Examples

##### Function

- `src/module_a/app.py`

  ```py
  def compute_foobars():
    ...
  ```

- `tests/module_a/test_app.py`

  ```py
  # "test_" + "func_name"
  def test_compute_foobars():
    ...

  # "test_" + "func_name" + "_scenario"
  def test_compute_foobars_when_ratelimit_reached():
    ...

  # "test_" + "func_name" + "_scenario"
  def test_compute_foobars_with_real_spameggs():
    ...
  ```

##### Class

- `app/module_b/foobar.py`

  ```py
  class MyClass:
      def start(self): ...
      def _compute_foo(self): ...
      def _compose_bar(self): ...
  ```

- `tests/unit/module_b/test_foobar.py`

  ```py
  class TestMyClass:
      def test_start(self): ...
      def test__compute_foo(self): ...
      def test__compose_bar(self): ...
  ```

- `tests/integration/module_b/test_foobar.py`
  ```py
  class TestMyClass:
    # "test_" + "name" + "_scenario".
    def test_start_with_real_user(self): ...
  ```

## Setup and Teardown

Setup and teardown logic must be outside of a test.

- Bad:
  ```py
  # This is a bad example. Do not copy!
  def test_compute_something():
    database.set("foo": "bar")  # Setup.
    assert compute_something()
    database.remove("foo")  # Teardown.
  ```
- Better (option 1):

  ```py
  @pytest.fixture()
  def setup_database():
      # Setup.
      database.set("foo": "bar")
      yield
      # Teardown.
      database.remove("foo")

  @pytest.mark.usefixtures("setup_database")
  def test_compute_something():
      assert compute_something()
  ```

- Better (option 2):

  ```py
  @pytest.fixture()
  def fill_database_with_fake_foobars():
      database.set("foo": "bar")
      yield

  @pytest.fixture()
  def cleanup_testing_database():
      yield
      database.remove("foo")


  @pytest.mark.usefixtures("fill_database_with_fake_foobars", "cleanup_testing_database")
  def test_compute_something():
      assert compute_something()
  ```

## Variable Naming

As with all your code, avoid using meaningless variable names like `data` or `result` within a test.

Bad:

```py
data = request(url)  # This is a bad example. Do not copy!
result = process(data)  # This is a bad example. Do not copy!
```

Better:

```py
response = request(url)
parsed_response = process(response)
```

## Comments Inside a Test

An additional rule specific to comments within a test is that you should avoid starting a comment with `Test that ...`. Below is an example of a comment that contains redundant and duplicate information.

```py
# Bad.
def test_func():
    # Test that a function can handle big values.  # <-- DO NOT WRITE LIKE THIS!
    assert func(42)
```

This is obvious from the code around what we are testing. Let's rewrite the comment:

```py
# Better.
def test_func():
    # Can handle big values.
    assert func(42)

# ...or create `def test_func_specific_scenario()`.
```
