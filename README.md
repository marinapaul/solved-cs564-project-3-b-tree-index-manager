Download Link: https://assignmentchef.com/product/solved-cs564-project-3-b-tree-index-manager
<br>
<h1>INTRODUCTION</h1>

Relations are stored in files. Each file has a particular or-ganization. Each organization lends itself to efficient evaluation of some (not all) of the following operations: scan, equality search, range search, insertion, and deletion. When it is important to access a relation quickly in more than one way, a good solution is to use an index. For this assignment, the index will store data entries in the form <strong>&lt;key,</strong> <strong>rid&gt;</strong> <strong>pair</strong>. These data entries in the index will be stored in a file that is separate from the data file. In other words, the index file “points to” the data file where the actual records are stored. Two primary kinds of indexes are hash-based and tree-based, and the most commonly implemented tree-based index is the B+ Tree.

For this project, you need to implement a B+ Tree index. To help get you started, we will provide you with an implementation of few new classes: <strong>PageFile</strong>, <strong>BlobFile</strong>, and <strong>FileScan</strong>.

The PageFile and BlobFile classes are derived from the File class. These classes implement a file interface in two different ways. The PageFile class implements the file interface for the File class. Hence, we use the PageFile class to store all the relations.

The BlobFile class implements the file interface for a file organization in which the pages in the file are not linked by prevPage/nextPage links, as they are in the case of the PageFile class. When reading/writing pages, the BlobFile class treats the pages as blobs of 8KB size and hence does not require these pages to be valid objects of the Page class. We will use the BlobFile class to store the B+ index file, where every page in the file is a node from the B+ T ree. Since no other class requires BlobFile pages to be valid objects of the Page class, we can modify these pages as we wish without worrying that these pages will not be valid after their arbitrary modification. Inside the file btree.cpp you will treat the pages from a BlobFile as your B+ Tree index nodes, and the BlobFile class will read/write pages for you from disk without modifying/using them in any way. BufMgr class has also been changed so that it does not use page objects to find out their page numbers.

1

<h1>THE FILESCAN CLASS</h1>

The FileScan class is used to scan records in a file. We will use this class for the base relation, and not for the index file. The file main.cpp file contains code which shows how to use this class. The public member functions of this class are described below.

<ul>

 <li><strong>FileScan (const std::string &amp;relationName, BufMgr *bufMgr);</strong></li>

</ul>

The constructor takes the relationName and buffer manager instance as parameters. The methods described below are then used to scan the relation.

<ul>

 <li><strong>~FileScan()</strong></li>

</ul>

Shuts down the scan and unpins any pinned pages.

<ul>

 <li><strong>void scanNext(RecordId&amp; outRid);</strong></li>

</ul>

Returns (via the outRid parameter) the RecordId of the next record from the relation being scanned. It throws EndOfFileException() when the end of relation is reached.

<ul>

 <li><strong>std::string getRecord();</strong></li>

</ul>

Returns the record identified by rid. The rid is obtained by a preceding scanNext() call.

<ul>

 <li><strong>void markDirty()</strong></li>

</ul>

Marks the current page being scanned as dirty, in case the page was being modified.

(You don’t need this for this assignment, but the method is here for completeness).

<h1>B+ TREE INDEX</h1>

Your assignment is to implement a B+ Tree index. This B+ Tree will be simplified in certain ways:

<ol>

 <li>You can assume that all records in a file have the same length (so for a given attribute its offset in the record is always the same).</li>

 <li>The B+ Tree only needs to support single-attribute indexing (not composite attribute).</li>

 <li>The indexed attribute may be only one data type: integer.</li>

 <li>You may assume that we never insert two data entries into the index with the same key value. The last part simplifies the B+ Tree implementation (think about why, and put that in your report).</li>

</ol>

The index will be built directly on top of the I/O Layer (the BlobFile and the Page classes). An index will need to store its data in a file on disk, and the file will need a name (so that the DB class can identify it). The convention for naming an index file is specified below. To create a disk image of the index file, you simply use the BlobFile constructor with the name of the index file. The file that you create is a “raw” file, i.e., it has no page structure on top of it. You will need to implement a structure on top of the pages that you get from the I/O Layer to implement the nodes of the B+ Tree. Note the PageFile class that we provide superimposes a page structure on the “raw” page. Just as the File class uses the first page as a header page to store the metadata for that file, you will dedicate a header page for the B+ Tree file too for storing metadata of the index.

We’ll start you off with an interface for a class, BTreeIndex. You will need to implement the methods of this interface as described below. You may add new public methods to this class if required, but you should not modify the interfaces that are described here:

<ul>

 <li><strong>BTreeIndex</strong></li>

</ul>

The constructor first checks if the specified index file exists. And index file name is constructed by concatenating the relational name with the offset of the attribute over which the index is built. The general form of the index file name is as follows: relName.attrOffset. The code for constructing an index name is shown below:

<table width="593">

 <tbody>

  <tr>

   <td width="436">std : : ostringstream idxStr ; idxStr &lt;&lt; relationName &lt;&lt; ’ . ’ &lt;&lt; attrByteOffset ; std : : string indexName = idxStr . str ( ) ; // indexName is index f i l e</td>

   <td width="157">the name of          the</td>

  </tr>

 </tbody>

</table>

If the index file exists, the file is opened. Else, a new index file is created.

<strong>Input to this constructor function:</strong>

<table width="587">

 <tbody>

  <tr>

   <td width="210">const string&amp; relationName</td>

   <td width="377">The name of the relation on which to build the index. The constructor should scan this relation (using FileScan) and insert entries for all the tuples in this relation into the index. You can insert an entry one-by-one, i.e., don’t worry about implementing a bottom-up bulkloading index construction mechanism.</td>

  </tr>

  <tr>

   <td width="210">String&amp; outIndexName</td>

   <td width="377">The name of the index file; determine this name in the constructor as shown above, and return the name.</td>

  </tr>

  <tr>

   <td width="210">BufMgr *bufMgrIn</td>

   <td width="377">The instance of the global buffer manager.</td>

  </tr>

  <tr>

   <td rowspan="3" width="210">const int attrByteOffset</td>

   <td width="377">The byte offset of the attribute in the tuple on which to build the index. For instance, if we are storing the following structure as a record in the original relation:</td>

  </tr>

  <tr>

   <td width="377">struct RECORD { int i ;double d;char     s [ 6 4 ] ;} ;</td>

  </tr>

  <tr>

   <td width="377">And, we are building the index over the double d, then the attrByteOffset value is <strong>0 + offsetof (RECORD, i)</strong>, where offsetof is the offset position provided by the standard C++ library “offsetoff”.</td>

  </tr>

  <tr>

   <td width="210">const Datatype attrType</td>

   <td width="377">The data type of the attribute we are indexing. Note that the Datatype enumeration INTEGER, DOUBLE, STRING is defined in btree.h.</td>

  </tr>

 </tbody>

</table>

<ul>

 <li><strong>~BTreeIndex</strong></li>

</ul>

The destructor. Perform any cleanup that may be necessary, including clearing up any state variables, unpinning any B+ Tree pages that are pinned, and flushing the index file (by calling bufMgr-&gt;flushFile()). Note that this method does not delete the index file! But, deletion of the file object is required, which will call the destructor of File class causing the index file to be closed.

<ul>

 <li><strong>insertEntry</strong></li>

</ul>

This method inserts a new entry into the index using the pair &lt;key, rid&gt;.

<strong>Input to this function:</strong>

<table width="589">

 <tbody>

  <tr>

   <td width="208">const void* key</td>

   <td width="381">A pointer to the value (integer) we want to insert.</td>

  </tr>

  <tr>

   <td width="208">const RecordId&amp; rid</td>

   <td width="381">The corresponding record id of the tuple in the base relation.</td>

  </tr>

 </tbody>

</table>

<ul>

 <li><strong>startScan</strong></li>

</ul>

This method is used to begin a “filtered scan” of the index. For example, if the method is called using arguments (1,GT,100,LTE), then the scan should seek all entries greater than 1 and less than or equal to 100.

<strong>Input to this function:</strong>

<table width="589">

 <tbody>

  <tr>

   <td width="208">const void* lowValue</td>

   <td width="381">The low value to be tested.</td>

  </tr>

  <tr>

   <td width="208">const Operator lowOp</td>

   <td width="381">The operation to be used in testing the low range. You should only support GT and GTE here; anything else should throw BadOpcodesException. Note that the Operator enumeration is defined in btree.h.</td>

  </tr>

  <tr>

   <td width="208">const void* highValue</td>

   <td width="381">The high value to be tested.</td>

  </tr>

  <tr>

   <td width="208">const Operator highOp</td>

   <td width="381">The operation to be used in testing the high range. You should only support LT and LTE here; anything else should throw BadOpcodesException.</td>

  </tr>

 </tbody>

</table>

Both the high and low values are in a binary form, i.e., for integer keys, these point to the address of an integer.

If lowValue &gt; highValue, throw the exception BadScanrangeException.

<ul>

 <li><strong>scanNext</strong></li>

</ul>

This method fetches the record id of the next tuple that matches the scan criteria. If the scan has reached the end, then it should throw the following exception: IndexScanCompletedException. For instance, if there are two data entries that need to be returned in a scan, then the third call to scanNext must throw IndexScanCompletedException. A leaf page that has been read into the buffer pool for the purpose of scanning, should not be unpinned from buffer pool unless all records from it are read or the scan has reached its end. Use the right sibling page number value from the current leaf to move on to the next leaf which holds successive key values for the scan.

<strong>Input to this function:</strong>

<table width="589">

 <tbody>

  <tr>

   <td width="208">RecordId&amp; outRid</td>

   <td width="381">An output value; this is the record id of the next entry that matches the scan filter set in startScan.</td>

  </tr>

 </tbody>

</table>

<ul>

 <li><strong>endScan</strong></li>

</ul>

This method terminates the current scan and unpins all the pages that have been pinned for the purpose of the scan. It throws ScanNotInitializedException when called before a successful startScan call.

<h1>ADDITIONAL NOTES</h1>

<ol>

 <li>When you implement these methods, you will need to call upon the buffer pool to read/write pages. Make sure you don’t keep the pages pinned in the buffer pool unless you need to. If you keep some pages pinned, make sure you have a good reason that you justify in your design report.</li>

 <li>For the scan methods, you will need to remember the “state” of the scan specified during the startScan call. Use appropriate member variables in the BTreeIndex class to remember this state. Make sure you reset these state variables in the endScan and the destructor.</li>

 <li>The insert algorithm does not need to redistribute entries, i.e., always prefer splits over key redistribution during inserts. (It is easier to implement inserts this way too).</li>

 <li>At the leaf level, you do not need to store pointers to both siblings. The leaf nodes only point to the “next” (the right) sibling.</li>

 <li>The constructor and destructor should not throw any exceptions.</li>

 <li>In real B+ Tree implementations, when an error occurs, special care is taken to make sure that the index does not end up in an inconsistent state. As you will quickly realize handling errors can be hard in some cases. For example, if you have split the leaf page and are propagating the split upwards, and then encounter a buffer manager error, exiting the method without cleaning up could corrupt the B+ Tree structure. To keep the assignment simple, don’t worry about this type of cleanup, simply return the error code. Make sure you don’t artificially create such problems by incorrectly using the other components of BadgerDB. For example, if you keep pages pinned in memory unnecessarily, you will quickly encounter a buffer exceeded exception. We will not test your implementation with very small buffer pool sizes (such as 1 or 2 pages). If it makes your implementation easier, you may assume that you have enough free buffer pages to hold 1-2 pages from each level of the index. But UNPIN THE PAGES as soon as you can.</li>

</ol>

<h1>GETTING STARTED</h1>

In the zipped folder on canvas, you will find the files listed below. Follow these instructions to complete your assignment. This directory contains the following files that are relevant to this part of the project (in addition to other files which were created while developing the lower layers):

<ul>

 <li>h: Add your own methods and structures as you see fit but don’t modify the public methods that we have specified.</li>

 <li>cpp: Implement the methods we specified and any others you choose to add.</li>

 <li>h(cpp): Implements the PageFile and BlobFile classes.</li>

 <li>cpp: Use to test your implementation. Add your own tests here or in a separate file. This file has code to show how to use the FileScan and BTreeIdnex classes.</li>

 <li>h(cpp): Implements the Page class.</li>

 <li>h(cpp), bufHashTbl.h(cpp): Implementation of the buffer manager.</li>

 <li>Exceptions/* : Implementation of exception classes that you might need.</li>

 <li>Makefile : Makefile for this project.</li>

</ul>

<strong><em>Please do not create any additional files.</em></strong>

In addition to the B+ Tree source files, you must also turn in

<ol>

 <li>New test cases that you wrote to test your B+ Tree index.</li>

 <li>Design report describing your tests and design choices.</li>

</ol>

<h1>DELIVERABLES</h1>

To submit your work, please go to Canvas to upload your files. Your submission should include the source code, new test cases and the design report. Submit-ting btree.cpp is mandatory. If you have also modified the header file, submit btree.h as well. Your new test case can be written in main.cpp, or in separate file. Clearly indicate what your new test case and design report files are called, for example by putting in a outline.txt file that describes your test and design report file locations. Your files must be uploaded by the deadline.

<h1>GRADING</h1>

The breakup of the grading for this assignment is as follows:

<ol>

 <li><strong>Correctness: 75%</strong>. The correctness part of the grade will be based on the tests that we have provided, and additional (more rigorous) tests that we will run on your submitted projects.</li>

 <li><strong>Programming Style: 5%</strong>. For your style points, we will check your code for readability (how easy is it to read and understand the code), and for the code organization (do you repeat code over and over again, do you use unnecessary globals, etc.).</li>

 <li><strong>Test design: 5%</strong>. Designing tests cases that test various code paths rigorously. This will not only get you the test points, but will most likely also get you the correctness points.</li>

 <li><strong>Design report: 15%</strong>. Your design report must describe the following design choices that you make:

  <ul>

   <li>Any implementation choices that you make. How often do you keep pages pinned? How efficient is your implementation? We are not going to run a speed test, but will look at the code to check if you are performing operations that are inefficient, such as unnecessarily traversing the tree up and down multiple times during range searches</li>

   <li>Please use your report to justify any additional design choices that you make.</li>

   <li>Finally, your design report must also explain how your design/implementation would change if you were to allow duplicate keys in the B+ Tree, i.e., allow multiple data entries with the same key value.</li>

  </ul></li>

</ol>

<h1>A FINAL NOTE OF CAUTION</h1>

There are number of design choices that you need to make, and you probably need to reserve a big chunk of time for testing and debugging. So, start working on this assignment early – you are unlikely to finish this project if you start just a week or so before the deadline.