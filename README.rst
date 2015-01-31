Converting GlusterFS GFID to Path using Historical Changelogs
#############################################################

GlusterFS will save a hardlink to each file and softlink to each directory inside ".glusterfs" directory of each brick. Name of these hardlinks/softlinks are GFIDs.
Crawling ".glusterfs" directory is very fast, but hard part is to convert GFID to Path.

We can use ``find -samefile <FILENAME>`` to find the hardlinked file, but is very expensive since it crawls entire filesystem to find hardlinks.

.. code-block:: bash

   find . -samefile .glusterfs/b6/7b/b67b95a4-8bdc-44d5-aedd-b9b66fb0af95

`Venky <https://github.com/vshankar>`__ suggested that we can use Changelogs if it is enabled from the begining or atleast enabled before the files creation.

Idea
====
Will consider ``b67b95a4-8bdc-44d5-aedd-b9b66fb0af95`` as GFID for all illustrations and file name as ``file1``

1. If ``.glusterfs/b6/7b/b67b95a4-8bdc-44d5-aedd-b9b66fb0af95`` is Symlink, Readlink and Convert to relative path and Print(Easy conversion, done :))
2. If not Symlink, Stat on file ``.glusterfs/b6/7b/b67b95a4-8bdc-44d5-aedd-b9b66fb0af95`` and get st_ctime, and decriment by 5. (For example ctime is ``1422694569``)
3. Search for CHANGELOG.1422694565, CHANGELOG.1422694566.. CHANGELOG.1422694590. Break when you get actual Changelog file, else break after MAX try.
4. If we didn't get valid Changelog file for that GFID, log GFID in stderr.
5. If Changelog file is available, Search in Changelog file for the ENTRY pattern. If found extract Parent GFID and basename. Convert Parent GFID to Path using readlink and add basename.
6. Get xattr ``trusted.gfid`` on the path we got and compare with input GFID. If both are same then we are Good, else log in stderr. (File may be changed after Create, may be Rename)
7. Collect all GFIDs from stderr and convert to Path using ``find . -samefile <GFID FILE>``

Usage
=====

Download the Python Script(gfid_to_path) from https://github.com/aravindavk/gfid_to_path and place it in any ``bin`` directory which is available in ``PATH``

.. code-block:: bash

   chmod +x /usr/local/bin/gfid_to_path

To convert single GFID to Path

.. code-block:: bash

   echo <GFID> | gfid_to_path <BRICK_PATH>

.. code-block:: bash

   echo b67b95a4-8bdc-44d5-aedd-b9b66fb0af95 | gfid_to_path /exports/bricks/b1

To convert list of GFIDs into Path,

.. code-block:: bash

   gfid_to_path <BRICK_PATH> <GFIDS LIST FILE>

.. code-block:: bash

   gfid_to_path /exports/bricks/b1 ~/collected_gfids.txt

To record Converted paths in ~/good.txt and failures in ~/bad.txt

.. code-block:: bash

   gfid_to_path <BRICK_PATH> <GFIDS LIST FILE> 1> GOOD_FILE 2> BAD_FILE

.. code-block:: bash

   gfid_to_path /exports/bricks/b1 ~/collected_gfids.txt 1> ~/good.txt 2> ~/bad.txt


C & S Welcome.

Source
======

git clone https://github.com/aravindavk/gfid_to_path.git
