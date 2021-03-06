.. _changed_page_tracking:

=============================
XtraDB changed page tracking
=============================

|XtraDB| now tracks the pages that have changes written to them according to the redo log. This information is written out in special changed page bitmap files.  This information can be used to speed up incremental backups using `Percona XtraBackup <http://www.percona.com/doc/percona-xtrabackup/>`_ by removing the need to scan whole data files to find the changed pages. Changed page tracking is done by a new |XtraDB| worker thread that reads and parses log records between checkpoints. The tracking is controlled by a new read-only server variable :variable:`innodb_track_changed_pages`.

Bitmap filename format used for changed page tracking is ``ib_modified_log_<seq>_<startlsn>.xdb``. The first number is the sequence number of the bitmap log file and the *startlsn* number is the starting LSN number of data tracked in that file. Example of the bitmap log files should look like this: :: 

 ib_modified_log_1_0.xdb
 ib_modified_log_2_1603391.xdb

Sequence number can be used to easily check if all the required bitmap files are present. Start LSN number will be used in |XtraBackup| and ``INFORMATION_SCHEMA`` queries to determine which files have to be opened and read for the required LSN interval data. The bitmap file is rotated on each server restart and whenever the current file size reaches the predefined maximum. This maximum is controlled by a new :variable:`innodb_max_bitmap_file_size` variable.

This feature will be used for implementing faster incremental backups that use this information to avoid full data scans in |Percona XtraBackup|.

User statements for handling the XtraDB changed page bitmaps
============================================================

In |Percona Server| :rn:`5.6.11-60.3` new statements have been introduced for handling the changed page bitmap tracking. All of these statements require ``SUPER`` privilege.

 * ``FLUSH CHANGED_PAGE_BITMAPS`` - this statement can be used for synchronous bitmap write for immediate catch-up with the log checkpoint. This is used by innobackupex to make sure that XtraBackup indeed has all the required data it needs.
 * ``RESET CHANGED_PAGE_BITMAPS`` - this statement will delete all the bitmap log files and restart the bitmap log file sequence.
 * ``PURGE CHANGED_PAGE_BITMAPS BEFORE <lsn>`` - this statement will delete all the change page bitmap files up to the specified log sequence number.

Additional information in SHOW ENGINE INNODB STATUS
===================================================
When log tracking is enabled, the following additional fields are displayed in the LOG section of the ``SHOW ENGINE INNODB STATUS`` output:

 * "Log tracked up to:" displays the LSN up to which all the changes have been parsed and stored as a bitmap on disk by the log tracking thread
 * "Max tracked LSN age:" displays the maximum limit on how far behind the log tracking thread may be.

INFORMATION_SCHEMA Tables
=========================

This table contains a list of modified pages from the bitmap file data.  As these files are generated by the log tracking thread parsing the log whenever the checkpoint is made, it is not real-time data.

.. table:: INFORMATION_SCHEMA.INNODB_CHANGED_PAGES

   :column INT(11) space_id: space id of modified page
   :column INT(11) page_id: id of modified page
   :column BIGINT(21) start_lsn: start of the interval
   :column BIGINT(21) end_lsn: end of the interval 

The ``start_lsn`` and the ``end_lsn`` columns denote between which two checkpoints this page was changed at least once. They are also equal to checkpoint LSNs.

Number of records in this table can be limited by using the variable :variable:`innodb_changed_pages_limit`.

System Variables
================

.. variable:: innodb_max_changed_pages

   :version 5.6.11-60.3: Variable :variable:`innodb_max_changed_pages` introduced
   :cli: Yes
   :conf: Yes
   :scope: Global
   :dyn: Yes
   :vartype: Numeric
   :default: 1000000
   :range: 1 - 0 (unlimited)

.. variable:: innodb_track_changed_pages

   :version 5.6.11-60.3: Variable introduced
   :cli: Yes
   :conf: Yes
   :scope: Global
   :dyn: No
   :vartype: Boolean
   :default: 0 - False
   :range: 0-1

.. variable:: innodb_max_bitmap_file_size

   :version 5.6.11-60.3: Variable introduced
   :cli: Yes
   :conf: Yes
   :scope: Global
   :dyn: Yes
   :vartype: Numeric 
   :default: 104857600 (100 MB)
   :range: 4096 (4KB) - 18446744073709551615 (16EB)
