# Personal Notes for the precice/systemtests repo


**local_test.py**

1. Use argparser to determine precice branch, precice dockerfile and systemtests to execute. Additional boolean `keep` for not deleting container after test. `force_rebuild` can be used to avoid caches when building. By default all tests are executed on Ubuntu1604.home using precice/develop without rebuilding or keeping containers.
> for the force_rebuild argument, the default is set as [] (i.e. when flag is not present). The `nargs='+'` setting prevents using the flag without any of the choices ["precice", "tests"]. Maybe it would be nice to have it rebuild all of the choices if the flag is given without arguments, as this is allowed by setting the option `-f "precice" "tests"` anyways.

2. Tests to be executed are filtered with the supplied dockerfile.

3. Check for existing "st_xxxx" containers and remove them.

4. build precice image from branch arg and corresponding dockerfile in `systemtests/precice`.

5. run each test and categorize each as success or failed.

**common.py**

- `call()` run command in shell, return exit code.
- `ccall()` call call(). The `check` flag causes exception for return values != 0.
- `capture_output()` run command and pipe output to stdout.
- `determine_test_name()` return first word in '.' delimited string.
- `get_tests()` return list of systemtests available (are in tests directory).
- `get_test_variants()` list variants of test.
- `get_test_participants()` lists solvers participating.
- `chdir()` using a contextmanager decorator to be used like `with chdir(<path>):` in a local block.
- `get_diff_files()` supply 2 directories, get three lists of files inside the directories which belong to the following: [content difference] [in left dir only] [in right dir only]
- `test_is_considered()` check if all characteristics (spec. degree 1) of a test are included in a list of features.
- `determine_specialization()` return number of '.' separated words.
- `filter_for_most_specialized_tests()` return dict with **one** highest spec. degree entry for each test. If 2 test specs with same base have same degree takes first in list.
- `filter_tests()` filter list of tests against a dockerfile with features. Returns list of the most specialized tests for each base test, whereby all of their features are covered by the dockerfile.

**docker.py**

- `get_namespace()` returns "st_".
- `get_dockername()` returns "precice".
- `get_images()` call docker image ls and return list of docker images which have the namespace prefix.
- `get_containers()` call docker container ls and return list of containers which have the namespace prefix.
- `build_image()` call docker build with some tag, dockerfile and build args. The force_rebuild boolean sets the CACHEBUST variable in the dockerfile, preventing use of cached images. Runs in a shell and raises exception on error.

**system_testing.py**

- `build()` call `build_image` with the supplied arguments. `local` boolean controls if name has `precice/` prefix or not. Build has name `systest-tag-branch`, derives from `precice-tag-branch:latest` image.
- `run()` start container with docker setting -itd of image `systest-tag-branch`. After the "docker run" command terminates the hosts Output folder is cleared and the containers Output content is copied over.
- `build_adapters()` move to adapters folder and call `build_image` with supplied arguments.
    > i tried executing this function as:
    ```
    system_testing.build_adapters("of-of", "ubuntu1604.home", "develop", local=True, force_rebuild=False)
    ```
    and got the following error:
    ```
    docker build --network=host --file Dockerfile.openfoam-adapter --tag st_openfoam-adapter-ubuntu1604.home-develop --build-arg from=st_precice-ubuntu1604.home-develop:latest .
    unable to prepare context: unable to evaluate symlinks in Dockerfile path: lstat /home/ederk/precice/systemtests/Dockerfile.openfoam-adapter: no such file or directory
    ```
    I dont see how, but it seems that the path here is outdated since the dockerfiles were put into seperate folders in the latest version.

- `run_compose()` build commands to setup env vars and docker-compose, execute silent_compose.sh and copy output data from container. Try all commands in test_path directory and compare reference with output.
- `class IncorrectOutput` exception class holding info about file differences.
- `comparison()` execute `get_diff_files()` on output and reference. If files differ, check via compare_results.sh if numerical difference are within tolerance, otherwise raise IncorrectOutput.
- `build_run_compare()`
