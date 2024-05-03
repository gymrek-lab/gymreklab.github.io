Lab websites
============

Main lab website
----------------

The lab website (gymreklab.com) is generated using Github pages from this repository: https://github.com/gymrek-lab/gymreklab.github.io

If you are not yet part of our GitHub organization, you should :ref:`fork our repository and add your photo to the lab webpage <labwebsite-add-your-photo>`.
Once your pull request is approved and merged, you will automatically become added to our GitHub org and will no longer need to fork this repository to make edits to it.

.. _labwebsite-add-your-photo:

Adding your photo
+++++++++++++++++

Below is the workflow for adding your photo to the lab webpage. Follow a similar worfklow to make other types of changes via Github.

1. Go to https://github.com/gymrek-lab/gymreklab.github.io/

2. Click "Fork" on the top far right to make your own copy of the repository. This will create a new repo https://github.com/<yourgitusername>/gymreklab.github.io/.

3. In your copy, you can edit things directly in the web browser for small edits. There are only two small changes you need to make:

  * Upload your photo to the images/ folder.

  * Edit the people.html page and change "images/unknown_photo.jpg" by your name to "images/<name of your photo file>.jpg"

4. On the home page of your repo, click "Pull request" to submit a pull request which someone can then review and merge to the final site.

Ideally, if you're doing more than minor edits, first preview changes locally using the steps below before submitting a pull request.

Making edits to the internal lab website
----------------------------------------

The lab internal website (this site) is generated from the ``doc/`` folder within the repository https://github.com/gymrek-lab/gymreklab.github.io using sphinx. It is hosted on readthedocs.

To make changes, you can either edit files directly in the web browser from GitHub, or clone the repo and edit files locally.

Editing pages on GitHub
+++++++++++++++++++++++
1. First, log into GitHub and navigate to our GitHub repository here: https://github.com/gymrek-lab/gymreklab.github.io/tree/master/doc

2. .. dropdown:: If you haven't already, create a new branch so that your changes do not conflict with anyone else's

  a. Click on "master" in the menu on the left side
  b. Type your new branch name in the search box
  c. Then click "Create branch from master"

  For example, to create a new branch named tutorial:

  .. image:: https://raw.githubusercontent.com/bioinfo-ucsd/wiki/3f99ade31c5f2daaaed93cf00d1b67980a9e28da/wiki/images/creating_new_github_branch.png

  If you've already created a new branch, you can instead switch to it by selecting it from the search menu.

3. Next, click :code:`.` (the period symbol) on your keyboard. This will open a VSCode-style editor within your browser.

4. Make your edits. The pages in our wiki are under the :code:`doc/` folder. All of our documentation is written in `the reStructuredText formatting/markup language <https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html#rst-primer>`_.

  If you create a new page, make sure to list it in our table of contents in the :code:`doc/index.rst` file.

5. Save your changes to GitHub:

  a. Click on the "Source Control" tab in the menu on the right
  b. Add a short commit message that describes your changes
  c. Click "Commit and Push"

  .. image:: https://raw.githubusercontent.com/bioinfo-ucsd/wiki/3f99ade31c5f2daaaed93cf00d1b67980a9e28da/wiki/images/github_commit_and_push.png

6. Feel free to make more changes and commits as needed. Once you're ready, open a pull request by choosing your branch from the dropdown under "compare: master":

  https://github.com/gymrek-lab/gymreklab.github.io/compare

  .. image:: https://raw.githubusercontent.com/bioinfo-ucsd/wiki/3f99ade31c5f2daaaed93cf00d1b67980a9e28da/wiki/images/creating_pr.png

7. Click "Create pull request" and then add an informative title and description of your changes. Then click "Create pull request" again.

8. Ask someone to review your pull request:

  a. Click on the gear icon in the top/right corner
  b. Start typing the GitHub username of someone capable of giving you helpful feedback on your changes
  c. Choose that person from the search results

  .. image:: https://raw.githubusercontent.com/bioinfo-ucsd/wiki/3f99ade31c5f2daaaed93cf00d1b67980a9e28da/wiki/images/choosing_a_reviewer.png

9. A bot will automatically build a preview of the wiki with your changes. If the build succeeds, you should see a green check mark next to your latest commit. Otherwise, there should be a red X.

  If your build succeeded, you can view the built preview by clicking on the green check mark and then the "Details" link

  Otherwise, you can view a log of the error messages by clicking on the red X. If the build failed, you should add another commit to fix it.

  .. image:: https://raw.githubusercontent.com/bioinfo-ucsd/wiki/3f99ade31c5f2daaaed93cf00d1b67980a9e28da/wiki/images/previewing_the_docs.png

10. Once the build succeeds and a reviewer has approved your pull request, scroll down to the bottom of the pull request and click "Merge". You're done! Just wait a few minutes for your changes to go live.

Editing pages locally
+++++++++++++++++++++
To build the docs locally, make sure sphinx is installed (e.g. :code:`conda install sphinx`) then within the :code:`doc/` directory, type :code:`make html`. This 
will build the html files in the :code:`_build/` directory, which you can preview them in your web browser.

Making changes to gymreklab.com
-------------------------------

To make edits to the main website, first clone the repo to your local computer.

.. code-block:: bash

   git clone https://github.com/gymrek-lab/gymreklab.github.io/
   cd gymreklab.github.io/

You'll need to have jekyll installed (https://jekyllrb.com/docs/installation/).

Then type

.. code-block:: bash

   bundle exec jekyll build
   bundle exec jekyll serve

If all went well, you can navigate to: localhost:4000 in your web browser to preview the changes.
