Upload package
==============

Finally, you can deploy the package on the :term:`Python Package Index`
(:term:`PyPI`) or another index, for example :doc:`gitlab` or :term:`devpi`.

For this you should register on *Test PyPI*. *Test-PyPI* is a separate instance
that is intended for testing and experimentation. To set up an account there, go
to  https://test.pypi.org/account/register/. For more information, see `Using
TestPyPI <https://packaging.python.org/guides/using-testpypi/>`_.

Now you can create the ``~/.pypirc`` file:

.. code-block:: ini

    [distutils]
    index-servers=
        test

    [test]
    repository = https://test.pypi.org/legacy/
    username = veit

.. seealso::
    If you’d like to automate PyPI registration, please read `Careful With That
    PyPI
    <https://glyph.twistedmatrix.com/2017/10/careful-with-that-pypi.html>`_.

After you are registered, you can upload your :term:`Distribution Package` with
:term:`twine`. To do this, however, you must first install twine with:

.. code-block:: console

    $ python -m pip update pip build twine
    …
    All dependencies are now up-to-date!

.. note::
   Run this command before each release to ensure that all release tools are up
   to date.

Now you can create your :term:`Distribution Packages <Distribution Package>`
with:

.. code-block:: console

    $ cd /path/to/your/distribution_package
    $ rm -rf build dist
    $ pyproject-build .

After installing Twine you can upload all archives in ``/dist`` to the Python
Package Index with:

.. code-block:: console

    $ twine upload -r test -s dist/*

``-r``, ``--repository``
    The repository to upload the package.

    In our case, the ``test`` section from the ``~/.pypirc`` file is used.

``-s``, ``--sign``
    signs the files to be uploaded with GPG.

You will be asked for the password you used to register on *Test PyPI*. You
should then see a similar output:

.. code-block:: console

    Uploading distributions to https://test.pypi.org/legacy/
    Enter your username: veit
    Enter your password:
    Uploading example-0.0.1-py3-none-any.whl
    100%|█████████████████████| 4.65k/4.65k [00:01<00:00, 2.88kB/s]
    Uploading example-0.0.1.tar.gz
    100%|█████████████████████| 4.25k/4.25k [00:01<00:00, 3.05kB/s]

.. note::
   If you get an error message similar to

   .. code-block:: console

    The user 'veit' isn't allowed to upload to project 'example'

   you have to choose a unique name for your package:

   #. change the ``name`` argument in the ``setup.py`` file
   #. remove the ``dist`` directory
   #. regenerate the archives

Check
-----

Installation
~~~~~~~~~~~~

You can use ``pip`` to install your package and check if it works. Create
a new :term:`virtual environment` and install your package on *Test PyPI*:

.. code-block:: console

    $ python3 -m venv test_env
    $ source test_env/bin/activate
    $ pip install -i https://test.pypi.org/simple/ minimal_example

.. note::
   If you have used a different package name, replace it with your package name
   in the command above.

:term:`pip` should install the package from *Test PyPI* and the output should
look something like this:

.. code-block:: console

    Looking in indexes: https://test.pypi.org/simple/
    Collecting minimal_example
      ...
    Installing collected packages: minimal_example
    Successfully installed minimal_example-0.0.1

You can test whether your package has been installed correctly by importing the
module and referencing the ``name`` property that was previously ntered in
``__init__.py``:

.. code-block:: console

    $ python
    Python 3.7.0 (default, Aug 22 2018, 15:22:29)
    …
    >>> import minimal_example
    >>> minimal_example.name
    'minimal_example'

.. note::

    The packages on *Test-PyPI* are only stored temporarily. If you want to
    upload a package to the real :term:`Python Package Index` (:term:`PyPI`),
    you can do so by creating an account on :term:`pypi.org` and following the
    same instructions, but using ``twine upload dist/*``.

README
~~~~~~

Also check whether the ``README.rst`` is displayed correctly on the test PyPI
page.

PyPI
----

Now register on the :term:`Python Package Index` (:term:`PyPI`) and make sure
that `two-factor authentication
<https://blog.python.org/2019/05/use-two-factor-auth-to-improve-your.html>`_
is activated by adding the following to the ``~/.pypirc`` file:

.. code-block:: ini

    [distutils]
    index-servers=
        pypi
        test

    [test]
    repository = https://test.pypi.org/legacy/
    username = veit

    [pypi]
    username = __token__

With this configuration, the name/password combination is no longer used for
uploading but an upload token.

.. seealso::
    * `PyPI now supports uploading via API token
      <https://pyfound.blogspot.com/2019/07/pypi-now-supports-uploading-via-api.html>`_
    * `What is two factor authentication and how does it work on PyPI?
      <https://pypi.org/help/#twofa>`_

Finally, you can publish your package on PyPI:

.. code-block:: console

    $ twine upload -r pypi -s dist/*

.. note::
    You cannot simply replace releases as you cannot re-upload packages with the
    same version number.

    Do not remove old versions from the Python Package Index.This only causes
    work for those who want to keep using that version and then have to switch
    to old versions on GitHub. PyPI has a `yank
    <https://pypi.org/help/#yanked>`_ function that you can use instead. This
    will ignore a particular version if it is not explicitly specified with
    ``==`` or ``===``.

.. seealso::
    * `PyPI Release Checklist
      <https://cookiecutter-namespace-template.readthedocs.io/en/latest/pypi-release-checklist.html>`_

GitHub Action
-------------

You can also create a GitHub action, which creates a package and uploads it to
PyPI at every time a release is created. Such a ``.github/workflows/pypi.yml``
file could look like this:

.. code-block:: yaml

    name: pypi
    on:
      push:
        tags:
        - '*'

    jobs:
      package-and-deploy:

        runs-on: ubuntu-latest

        steps:
          - name: Checkout
            uses: actions/checkout@v2
            with:
              fetch-depth: 0

          - name: Set up Python
            uses: actions/setup-python@v2
            with:
              python-version: 3.8

          - name: Install dependencies
            run: |
              python -m pip install -U pip
              python -m pip install -U setuptools twine wheel

          - name: Build and publish
            env:
              TWINE_PASSWORD: ${{ secrets.TWINE_PASSWORD }}
              TWINE_USERNAME: ${{ secrets.TWINE_USERNAME }}
            run: |
              python setup.py sdist bdist_wheel
              twine upload dist/*

.. seealso::

   * `GitHub Actions <https://docs.github.com/en/actions>`_
