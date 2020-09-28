.. image:: https://img.shields.io/github/downloads/Nextomics/NextDenovo/total?logo=github
   :target: https://github.com/Nextomics/NextDenovo/releases/download/v2.3.1/NextDenovo.tgz
   :alt: Download
.. image:: https://img.shields.io/github/release/Nextomics/NextDenovo.svg
   :target: https://github.com/Nextomics/NextDenovo/releases
   :alt: Version
.. .. image:: https://img.shields.io/github/issues/Nextomics/NextDenovo.svg
..    :target: https://github.com/Nextomics/NextDenovo/issues
..    :alt: Issues
.. .. image:: https://img.shields.io/badge/切换-中文版本-9cf
..    :target: https://github.com/Nextomics/NextDenovo/issues
..    :alt: 中文版本

==========
NextDenovo
==========

NextDenovo is a string graph-based *de novo* assembler for long reads. It uses a "correct-then-assemble" strategy similar to canu, but requires significantly less computing resources and storages. After assembly, the per-base error rate is about 98-99.8%, to further improve single base accuracy, please use `NextPolish <https://github.com/Nextomics/NextPolish>`_.

NextDenovo contains two core modules: NextCorrect and NextGraph. NextCorrect can be used to correct long noisy reads with approximately 15% sequencing errors, and NextGraph can be used to construct a string graph with corrected reads. It also contains a modified version of `minimap2 <https://github.com/lh3/minimap2>`_ and some useful utilities (see `here <./doc/UTILITY.md>`__ for more details).

We benchmarked NextDenovo against other assemblers using Oxford Nanopore long reads from human and drosophila melanogaster, and PacBio continuous long reads (CLR) from arabidopsis thaliana. NextDenovo produces more contiguous assemblies with fewer contigs compared to the other tools. NextDenovo also shows a high assembly accurate level in terms of assembly consistency and single-base accuracy, see `here <#benchmark>`__ for details.

.. Table of Contents
.. -----------------

.. -  `Installation <#install>`_
.. -  `Quick start <#start>`_
.. -  `Tutorial <./doc/TEST1.md>`_
.. -  `Parameters <./doc/OPTION.md>`_
.. -  `Benchmark <#benchmark>`_
.. -  `Utilities <./doc/UTILITY.md>`_
.. -  `Getting help <#help>`_
.. -  `Copyright <#copyright>`_
.. -  `Cite <#cite>`_
.. -  `Limitations <#limit>`_
.. -  `FAQ <#faq>`_
.. -  `Star <#star>`_

Installation
~~~~~~~~~~~~

-  **DOWNLOAD**  

   click `here <https://github.com/Nextomics/NextDenovo/releases/download/v2.3.1/NextDenovo.tgz>`__ or use the following command:

   .. code:: console

      wget https://github.com/Nextomics/NextDenovo/releases/download/v2.3.1/NextDenovo.tgz

   .. note:: If you get an error like ``version 'GLIBC_2.14' not found`` or ``liblzma.so.0: cannot open shared object file``, Please download `this version <https://github.com/Nextomics/NextDenovo/releases/download/v2.3.1/NextDenovo-CentOS6.9.tgz>`_.

-  **REQUIREMENT**

   -  `Python <https://www.python.org/download/releases/>`_ (Support python 2 and 3):
   
      -  `Psutil <https://psutil.readthedocs.io/en/latest/>`_
      -  `Drmaa <https://github.com/pygridtools/drmaa-python>`_ (Only required by running under non-local system)

-  **INSTALL**

      |ss| ``tar -vxzf NextDenovo.tgz && cd NextDenovo && make`` |se|

-  **UNINSTALL**
   
      |ss| ``cd NextDenovo && make clean`` |se|

-  **TEST**
   
   .. code:: console

      nextDenovo test_data/run.cfg 


Quick Start
~~~~~~~~~~~

#. Prepare input.fofn

   .. code:: console

      ls reads1.fasta reads2.fastq reads3.fasta.gz reads4.fastq.gz ... > input.fofn
#. Create run.cfg

   .. code:: console

      cp NextDenovo/doc/run.cfg ./
   
   .. note:: Please change the value of seed\_cutoff using `bin/seq\_stat <./doc/UTILITY.md#seq_stat>`_ and refer to `doc/OPTION <doc/OPTION.md>`_ to optimize parallel computing parameters.

#. Run

   .. code:: console

      nextDenovo run.cfg

#. Result

   -  Sequence: ``01_rundir/03.ctg_graph/nd.asm.fasta``
   -  Statistics: ``01_rundir/03.ctg_graph/nd.asm.fasta.stat``

Benchmark
~~~~~~~~~

-  `CHM13hTERT human cell line with 120x Oxford Nanopore data <./doc/TEST2.md>`_
-  `Arabidopsis thaliana with 192X PacBio CLR data (~1% heterozygosity) <./doc/TEST3.md>`_
-  `Drosophila melanogaster with 69X Oxford Nanopore data <./doc/TEST4.md>`_

Getting Help
~~~~~~~~~~~~

-  **HELP**

   Please raise an issue at the `issue page <https://github.com/Nextomics/NextDenovo/issues/new/choose>`_. They would also be helpful to other users.

-  **CONTACT**
   
   For additional help, please send an email to huj\_at\_grandomics\_dot\_com.

Copyright
~~~~~~~~~

NextDenovo is freely available for academic use and other non-commercial use. For commercial use, please contact `NextOmics <https://www.nextomics.cn/en/>`_.

Cite
~~~~

We are now preparing the manuscript of NextDenovo, so if you use NextDenovo now, please cite the official website (https://github.com/Nextomics/NextDenovo)

Limitations
~~~~~~~~~~~

#. The current version of NextDenovo is not suitable for assembly with PacBio HiFi reads, becasue Minimap2 does not optimize for HiFi reads overlapping.
#. NextDenovo is optimized for assembly with seed\_cutoff >= 10kb. This should not be a big problem because it only requires the longest 30x-45x seeds length >= 10kb. For shorter seeds, it may produce unexpected results for some complex genomes and need be careful to check the quality.

Frequently Asked Questions
~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Which job scheduling systems are supported by NextDenovo?

   NextDenovo uses `DRMAA <https://en.wikipedia.org/wiki/DRMAA>`__ to submit, control, and monitor jobs, so theoretically it supports all DRMAA-compliant systems, such as LOCAL, SGE, PBS, SLURM. See `ParallelTask <https://github.com/moold/ParallelTask>`_ to configure drmaa.
#. **How to continue running unfinished tasks?**
   
   No need to make any changes, simply run the same command again.
#. How to reduce the total number of subtasks?
   
   Please increase blocksize and reduce seed\_cutfiles.
#. How to speed up NextDenovo?
   
   Currently, the bottlenecks of NextDenovo are minimap2 and IO. For minimap2, please see `here <https://github.com/lh3/minimap2/issues/322>`__ to accelerate minimap2, besides, you can increase -l to reduce result size and disk consumption. For IO, you can check how many activated subtasks using top/htop, in theory, it should be equal to the -p parameter defined in correction\_options. Use usetempdir will reduce IO wait, especially if usetempdir is on a SSD driver.
#. How to specify the queue cpu/memory/bash to submit jobs?
   
   Please use cluster\_options, NextDenovo will replace {vf}, {cpu}, {bash} with specific values needed for each jobs.
#. RuntimeError: Could not find drmaa library. Please specify its full path using the environment variable DRMAA\_LIBRARY\_PATH.
   
   Please setup the environment variable: DRMAA\_LIBRARY\_PATH, see `here <https://github.com/pygridtools/drmaa-python>`__ for more details.
#. ERROR: drmaa.errors.DeniedByDrmException: code 17: error: no suitable queues.
   
   This is usually caused by a wrong setting of cluster\_options, please check cluster\_options first. If you use SGE, you also can add '-w n' to cluster\_options, it will switch off validation for invalid resource requests. Please add a similar option for other job scheduling systems.

Star
~~~~

You can track updates by tab the "Star" button on the upper-right corner at this page.

.. |ss| raw:: html

   <strike>

.. |se| raw:: html

   </strike>
