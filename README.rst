===========================
Reproducible Science @ TACC
===========================

Quickstart
-----------

1. Create a Python 3 environment
2. Install dependencies ``pip install -r requirements.txt``
3. Build using ``make html`` (Mac/Linux) or ``make.bat html`` (Windows)

If you ``pip install sphinx-autobuild``, you can use ``make livehtml`` which
will start a server that watches for source changes and will rebuild/refresh
automatically. Go to http://localhost:8000/ to see its output.

reStructuredText help
---------------------

rST is a bit more onerous than Markdown, but it includes more advanced features
like inter-page references/links and a suite of directives.

- `Sphinx's primer <http://www.sphinx-doc.org/en/stable/rest.html>`_
- `Full Docutils reference <http://docutils.sourceforge.net/rst.html>`_

  - also see its `Quick rST
    <http://docutils.sourceforge.net/docs/user/rst/quickref.html>`_ cheat sheet.

- `Online rST editor <http://rst.ninjs.org/>`_ (it gets some things wrong)
- Other projects that use rST/Sphinx

  - `Python <https://docs.python.org/3/library/index.html>`_: click the "Show
    Source" under "This Page" in the sidebar.
  - `Sphinx <http://www.sphinx-doc.org/en/stable/rest.html>`_: similar
  - Numpy; note that the landing pages are usually coded in HTML and can be
    found in the templates directory, e.g. `Numpy's landing page
    <https://github.com/numpy/numpy/blob/master/doc/source/_templates/indexcontent.html>`_
