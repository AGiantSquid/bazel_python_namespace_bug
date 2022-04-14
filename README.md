# Bazel Python Namespace Bug

This project is attempting to use a python namespace package `acme_corp`.

Here is the directory structure:
```
├── python_projects
│   └── acme_corp
│       ├── app1
│       │   ├── BUILD.bazel
│       │   ├── pyproject.toml
│       │   └── src
│       │       └── acme_corp
│       │           └── app1
│       │               ├── app1_module1.py
│       │               ├── app1_module2.py
│       │               └── __init__.py
│       └── lib1
│           ├── BUILD.bazel
│           ├── pyproject.toml
│           └── src
│               └── acme_corp
│                   └── lib1
│                       ├── __init__.py
│                       ├── lib1_module1.py
│                       └── lib1_module2.py
└── WORKSPACE.bazel
```



It is possible to run app1_module1 with the following command on Linux:
```
$ bazel run python_projects/acme_corp/app1:app1_module1
INFO: Analyzed target //python_projects/acme_corp/app1:app1_module1 (0 packages loaded, 0 targets configured).
INFO: Found 1 target...
Target //python_projects/acme_corp/app1:app1_module1 up-to-date:
  bazel-bin/python_projects/acme_corp/app1/app1_module1
INFO: Elapsed time: 0.365s, Critical Path: 0.01s
INFO: 1 process: 1 internal.
INFO: Build completed successfully, 1 total action
INFO: Build completed successfully, 1 total action
3.7.10 (default, Mar 10 2021, 10:12:32)
[GCC 8.3.0]
app1_func1!
app1_func2!
lib1_func1!
lib1_func3!
```

This does not work on Windows, however.

```
Starting local Bazel server and connecting to it...
INFO: Analyzed target //python_projects/acme_corp/app1:app1_module1 (36 packages loaded, 179 targets configured).
INFO: Found 1 target...
Target //python_projects/acme_corp/app1:app1_module1 up-to-date:
  bazel-bin/python_projects/acme_corp/app1/app1_module1.exe
  bazel-bin/python_projects/acme_corp/app1/app1_module1.zip
INFO: Elapsed time: 7.861s, Critical Path: 0.47s
INFO: 6 processes: 5 internal, 1 local.
INFO: Build completed successfully, 6 total actions
INFO: Build completed successfully, 6 total actions
3.7.9 (tags/v3.7.9:13c94747c7, Aug 17 2020, 18:58:18) [MSC v.1900 64 bit (AMD64)]
Traceback (most recent call last):
  File "\\?\C:\Users\ASHLEY~1\AppData\Local\Temp\Bazel.runfiles_j5rp0o76\runfiles\acme_corp\python_projects\acme_corp\app1\src\acme_corp\app1\app1_module1.py", line 4, in <module>
    from acme_corp.app1.app1_module2 import app1_func2
ModuleNotFoundError: No module named 'acme_corp.app1'
```

This seems to be due to an extra `__init__.py` file that gets added to the zip file that bazel creates. When I extract the `app_module.zip`, I see this:
```
└───runfiles
    └───acme_corp
        │   __init__.py
        │
        └───python_projects
            └───acme_corp
                ├───app1
                │   └───src
                │       └───acme_corp
                │           └───app1
                │                   app1_module1.py
                │                   app1_module2.py
                │                   __init__.py
                │
                └───lib1
                    └───src
                        └───acme_corp
                            └───lib1
                                    lib1_module1.py
                                    lib1_module2.py
                                    __init__.py
```

If I delete the `__init__.py` file at `runfiles/acme_corp/__init__.py` and rezip the contents, the file runs like normal when I invoke it directly.

```
bazel-bin\python_projects\acme_corp\app1\app1_module1.exe
3.7.9 (tags/v3.7.9:13c94747c7, Aug 17 2020, 18:58:18) [MSC v.1900 64 bit (AMD64)]
app1_func1!
app1_func2!
lib1_func1!
lib1_func3!
```
