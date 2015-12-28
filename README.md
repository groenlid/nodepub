# Nodepub v1.0.0

Nodepub is a **Node** module which can be used to create **EPUB** documents. The resultant files are designed to pass the [IDPF online validator](http://validator.idpf.org) and Sigil's preflight checks. They should also open correctly in iBooks (as this is the most picky mainstream reader) and be suited for upload into the Amazon KDP and Kobo Writing Life sites. For those using KDP who wish to do a conversion locally, it should also satisfy KindleGen.

Resultant EPUBs can be generated to one of three levels of completeness:

1. A complete .epub file ready for distribution
2. A folder containing all the files necessary to build the final EPUB
3. A Javascript object containing all the filenames and content needed for the final EPUB


The module has no concept of how the original content is obtained; it is passed a metadata object at creation after which content is added sequentially by the caller. There is no automatic pre-processing of content files such as Markdown as the module expects to be given *HTML* to work with and it is trivial for the caller to pre-process in advance by requiring a Markdown module, the Jade rendering engine or similar.

### Progress ###

The generation of EPUBs to all three levels of completeness mentioned above is in place along with a sample script to show the usage. The codebase also includes both the resulting sample EPUB and the files that were joined to create it.

EPUBs generated pass the IDPF validator and display in the iBooks library (list view shows more metadata).

Cover images are included and must be in PNG format; I recommend 600x800 or an alternative of the same aspect ratio.

If you use the raw generated files to write the EPUB yourself, bear in mind that the **mimetype** file MUST be the first file in the archive and also MUST NOT be compressed. In simple terms, an EPUB is a renamed ZIP file where compression is optional apart from the mimetype file which should be added first using the 'store' option.

ALL functions of the module are synchronous EXCEPT if you choose to use the option to create the complete EPUB (*writeEPUB*), which is internally synchronous whilst generating the file contents but for external purposes is asynchronous with a callback due to the nature of the *archiver* dependency used to create the final output.
As the use of this third output option is expected to be mutually exclusive of the other two, this latter one being asynchronous is not currently considered an issue.

*Upcoming Changes*

* Allow CSS overriding; the current EPUBs are simple yet attractive, but I appreciate you may want to add your own styles.
* Inline images; you can already have a cover image so the base functionality is there, but I will be adding an option to insert an image at any point in the text. As each chapter's HTML is provided in advance by the caller, the links will already be in the markup so all that should be required is to copy the images themselves into a suitable place in the file and ensure they appear in the relevant EPUB structural files.

### Recent Updates ###

The following two changes do not break the API but *will* materially affect the output pages of current projects:

* Added *includeBackMatter* option to definition file.
* Renamed *copyright* page to *front matter* page.

### Tests and Example ###

There are both tests and also serve as an example generator.

To run the tests, in the top folder (containing the *package.json* file) run the following and check the inner test subfolder for both a resulting final EPUB and a subfolder of constituent files:

``` javascript
npm test
```

*Important Note about the Tests*

The tests stub *fs* where necessary.
However they do actually write a final EPUB document as this also serves as an *example* of the resulting files.
This means that (a) the test process needs writes to the test folder and (b) an actual file is generated.
Whilst the *process* is tested, the final EPUB is not; it is manually tested via the [IDPF Validator](http://validator.idpf.org/).
The actual testing of an EPUB file is already sufficiently covered by the *epubcheck* that site uses.
It would be pretty simple to include the use of *epubcheck* as part of integration testing should the need arise.

### Usage ###

Using **nodepub** is straightforward. The HTML you provide for chapter contents should be for the body contents only (typically a sequence of *&lt;p>one paragraph of text&lt;/p>* lines).
You can use header tags, but I recommend no higher than *h3* be used. This may mean that if you produce HTML from a Markdown file you need to demote headers with a search and replace (*h1* becomes *h3*, *h2* becomes *h4* and so on - remember to demote the closing tags too, as incorrect content markup confuses some ereader software [eg iBooks]).

The sequence for creation is:

1. Require the module and call *document()* with a metadata object detailing your book.
1. Repeatedly call *addChapter()* with a title and HTML contents.

And for production is one of:

1. Call *getFilesForEPUB()* if you want a simple array of file description and contents for storing in a database or further working through. This array will contain all needed files for a valid EPUB, along with their name, subfolder (if any) and a flag as to whether they should be compressed (a value of *false* should **not** be ignored).
1. Call *writeFilesForEPUB()* if you want to create a folder containing the complete files as mentioned above. You can edit these and produce an EPUB; simply zip the files and change the extention. For a valid EPUB the *mimetype* file **must** be added first and *must not* be compressed. Some validators will pass the file anyway; some ereaders will fail to read it.
1. Call *writeEPUB()*. This is the easiest way and also the only one guaranteed to produce valid EPUB output simply because the other two methods allow for changes and compression issues.


#### The Simplest Way ####

```javascript
// Create a book.
var epub = require('nodepub').document({
	id: '12345678',
	title: 'Unnamed Document',
	series: 'My Series',
	sequence: 1,
	author: 'Anonymous',
	fileAs: 'Anonymous',
	genre: 'Non-Fiction',
	tags: 'Sample,Example,Test',
	copyright: 'Anonymous, 1980',
	publisher: 'My Fake Publisher',
	published: '2000-12-31',
	language: 'en',
	cover: 'test-cover.png',
	description: 'A test book.',
	thanks: "Thanks for reading <em>[[TITLE]]</em>. If you enjoyed it please consider leaving a review where you purchased it.",
	linkText: "See more books and register for special offers here.",
	bookPage: "https://github.com/kcartlidge/nodepub",
	showChapterNumbers: true,
	includeCopyrightPage: true,
	includeBackMatter: true
});

// Add some content.
var lipsum = "<p>Lorem ipsum dolor sit amet adipiscing.</p>";
epub.addChapter('In the Beginning', lipsum);
epub.addChapter('Setting the Scene', lipsum);
epub.addChapter('A Moment of Conflict', lipsum);
epub.addChapter('The Conclusion of Things', lipsum);

// Write a complete EPUB.
epub.writeEPUB('.', 'sample', 'sample-output', function() { console.log('\nFinished.\n'); });
```

Note that this example will require a *cover image* called *test-cover.png*.

#### More Manual Output Options ####

```javascript
// Create a book.
var epub = require('nodepub').document({
	id: '12345678',
	title: 'Unnamed Document',
	series: 'My Series',
	sequence: 1,
	author: 'Anonymous',
	fileAs: 'Anonymous',
	genre: 'Non-Fiction',
	tags: 'Sample,Example,Test',
	copyright: 'Anonymous, 1980',
	publisher: 'My Fake Publisher',
	published: '2000-12-31',
	language: 'en',
	cover: 'test-cover.png',
	description: 'A test book.',
	thanks: "Thanks for reading <em>[[TITLE]]</em>. If you enjoyed it please consider leaving a review where you purchased it.",
	linkText: "See more books and register for special offers here.",
	bookPage: "https://github.com/kcartlidge/nodepub",
	showChapterNumbers: true,
	includeCopyrightPage: true,
	includeBackMatter: true
});

// Add some content.
var lipsum = "<p>Lorem ipsum dolor sit amet adipiscing.</p>";
epub.addChapter('In the Beginning', lipsum);
epub.addChapter('Setting the Scene', lipsum);
epub.addChapter('A Moment of Conflict', lipsum);
epub.addChapter('The Conclusion of Things', lipsum);

// List the files generated.
console.log('\nFiles created:\n');
var files = epub.getFilesForEPUB();
for(var i in files) {
	console.log('  ', files[i].name, ' -- ', files[i].content.length, 'bytes');
};

// Write the contents of the EPUB file into a folder (as an FYI or if you wish to modify it).
epub.writeFilesForEPUB('./sample');
```

Note that this example will require a *cover image* called *test-cover.png*.

*This is a utility module, not a user-facing one. In other words it is assumed that the caller has already validated the inputs.*
