Lab websites
============

Main lab website
----------------

The lab website (gymreklab.com) is generated using Github pages from this repository: https://github.com/gymrek-lab/gymreklab.github.io

.. _labwebsite-add-your-photo:

Adding your photo
+++++++++++++++++

Below is the workflow for adding your photo to the lab webpage. Follow a similar worfklow to make other types of changes via Github.

1. Go to https://github.com/gymrek-lab/gymreklab.github.io/

2. Click "Fork" on the top far right to make your own copy of the repository. This will create a new repo https://github.com/<yourgitusername>/gymreklab.github.io/.

3. In your copy, you can edit things directly in the web browser for small edits. There are only two small changes you need to make:

  * Upload your photo to the images/ folder.

  * Edit the people.html page and change "images/unknown_photo.jpg" by your name to "images/<name of your photo file>.jpg"

4. On the home page of your repo, click "Pull request" to submit a pull request which I can then review and merge to the final site.

Ideally, if you're doing more than minor edits, first preview changes locally using the steps below before submitting a pull request.

Previewing website changes
++++++++++++++++++++++++++

To preview your changes, clone your copy of the repo to your local computer.

.. code-block:: bash

   git clone https://github.com/<yourgitusername>/gymreklab.github.io/
   cd gymreklab.github.io/

You'll need to have jekyll installed (https://jekyllrb.com/docs/installation/).

Then type

.. code-block:: bash

   bundle exec jekyll build
   bundle exec jekyll serve

If all went well, you can navigate to: localhost:4000 in your web browser to preview the changes.

Internal lab website
--------------------

The lab internal website (this site) is generated from the ``doc/`` folder within the repository https://github.com/gymrek-lab/gymreklab.github.io using sphinx. It is hosted on readthedocs.

To make changes, you can either edit files directly in the web browser from github, or clone the repo and edit files locally using the instructions above. If you are a member of `our Github organization <https://github.com/gymrek-lab>`_, you shouldn't need to fork the repository but you'll need to create a new branch, instead.

To build the docs locally, make sure sphinx is installed (e.g. :code:`conda install sphinx`) then within the :code:`doc/` directory, type :code:`make html`. This 
will build the html files in the :code:`_build/` directory, which you can preview in your web browser.

Once you've finished making changes, create a pull request to merge your branch back into the master branch.

At this stage, Github actions will automatically generate a preview of the documentation. To access it, click on the green check-mark next to your most recent commit listed on the pull request. Then click "Details". If you see a red X instead of a green check-mark, it means there was an error or warning while building the documentation. Follow the same directions to view it.
