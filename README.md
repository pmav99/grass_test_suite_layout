Proposal for GRASS GIS test layout
==================================

Prerequisites
-------------

```
pytest
```

Introduction
------------

Of course, this is not the only way to organize a testsuite, but for codebases of
non-trivial complexity, I think that it makes sense to place all the tests in a single
directory, grouping the various tests into subdirectories (using multiple levels of
subdirectories if needed). This way you always know where to look.

A nice example of such a testsuite is IMHV
[QGIS](https://github.com/qgis/QGIS/tree/master/tests)

Sample layout
-------------

In order to move this discussion forward, I created a sample layout (obviously, there
are going to be more directories in the real one):

```
$ tree -d -C -v -I '*__pycache__' tests

tests
├── integration
├── modules
│   ├── general
│   ├── raster
│   └── vector
└── pygrass
```

I also added some sample test files to showcase how pytest can work in this context.
You can see the full tree with the following command:

```
$ tree -C -v --dirsfirst -P '*.py' -I '*__pycache__|*.pyc' tests
```

### `pytest` integration

pytest can print on stdout the tests it can discover by using:

```
pytest tests --collect-only
```

By default, pytest runs all the tests it can find:

```
$ pytest tests

======================== test session starts ========================
platform linux -- Python 3.7.3, pytest-4.4.1, py-1.8.0, pluggy-0.9.0
collected 12 items

tests/modules/general/g_copy_test.py::test_foo PASSED        [  8%]
tests/modules/general/g_filename_test.py::test_foo PASSED    [ 16%]
tests/modules/general/g_list_test.py::test_foo PASSED        [ 25%]
tests/modules/general/g_proj_test.py::test_foo PASSED        [ 33%]
tests/modules/general/g_region_test.py::test_foo PASSED      [ 41%]
tests/modules/raster/r_flow_test.py::test_foo PASSED         [ 50%]
tests/modules/raster/r_grow_test.py::test_foo PASSED         [ 58%]
tests/modules/raster/r_his_test.py::test_foo PASSED          [ 66%]
tests/modules/raster/r_info_test.py::test_foo PASSED         [ 75%]
tests/modules/vector/v_build_test.py::test_foo PASSED        [ 83%]
tests/modules/vector/v_class_test.py::test_foo PASSED        [ 91%]
tests/modules/vector/v_info_test.py::test_foo PASSED         [100%]

===================== 12 passed in 0.05 seconds =====================
```

You can selectively run

- the tests from a single directory (e.g. all the GRASS modules related tests, all the
    vector related tests, all the integration tests, etc)
- the tests from a single module
- or even a single test (e.g. a failing one)

by copy-pasting the relevant line from the output. To test this out, you can try the
following:

- `pytest tests/modules`
- `pytest tests/modules/vector`
- `pytest tests/modules/integration`
- `pytest tests/modules/vector/v_build_test.py`
- `pytest tests/modules/vector/v_build_test.py::test_foo`

Apart from that you can also specify a pattern to run matching tests. E.g. to only run
`info`-related tests:

```
pytest -vk info tests

======================== test session starts ========================
platform linux -- Python 3.7.3, pytest-4.4.1, py-1.8.0, pluggy-0.9.0
collected 12 items / 10 deselected / 2 selected

tests/modules/raster/r_info_test.py::test_foo PASSED         [ 50%]
tests/modules/vector/v_info_test.py::test_foo PASSED         [100%]

============== 2 passed, 10 deselected in 0.04 seconds ==============
```

So if we want to run all tests that have to do with `r.info` we could do it like this:

```
pytest -vk r_info tests
```

#### Failing tests

Edit `tests/modules/raster/r_info_test.py` and add the following function:

``` python
def test_goo():
    assert False
```

Now run the testsuite and specify the `-x` flag:

```

pytest -vx

======================== test session starts ========================
platform linux -- Python 3.7.3, pytest-4.4.1, py-1.8.0, pluggy-0.9.0
collected 13 items

tests/modules/general/g_copy_test.py::test_foo PASSED         [  7%]
tests/modules/general/g_filename_test.py::test_foo PASSED     [ 15%]
tests/modules/general/g_list_test.py::test_foo PASSED         [ 23%]
tests/modules/general/g_proj_test.py::test_foo PASSED         [ 30%]
tests/modules/general/g_region_test.py::test_foo PASSED       [ 38%]
tests/modules/raster/r_flow_test.py::test_foo PASSED          [ 46%]
tests/modules/raster/r_grow_test.py::test_foo PASSED          [ 53%]
tests/modules/raster/r_his_test.py::test_foo PASSED           [ 61%]
tests/modules/raster/r_info_test.py::test_foo PASSED          [ 69%]
tests/modules/raster/r_info_test.py::test_goo FAILED          [ 76%]

============================= FAILURES ==============================
_____________________________ test_goo ______________________________

    def test_goo():
>       assert False
E       assert False

tests/modules/raster/r_info_test.py:6: AssertionError
================ 1 failed, 9 passed in 0.16 seconds =================
```

As you can see, the testrunner stopped as soon as it encountered the first failing test.
So no need to wait until the completion of all the tests to see if there are errors.

#### Other features

There are many more features, including:

* [ignoring paths](https://docs.pytest.org/en/latest/example/pythoncollection.html#ignore-paths-during-test-collection)
* ["deselecting" individual tests](https://docs.pytest.org/en/latest/example/pythoncollection.html#deselect-tests-during-test-collection))
* [marking "slow" tests as such](https://docs.pytest.org/en/2.9.1/example/simple.html#control-skipping-of-tests-according-to-command-line-option) which is useful to only run them on CI
* [running multiple tests in parallel](https://github.com/pytest-dev/pytest-xdist) (i don't think that this is currently possible due to `GISRC`, but it is doable if we make changes to the test data structure).
