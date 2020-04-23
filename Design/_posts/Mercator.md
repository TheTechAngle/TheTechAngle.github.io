## Introduction 

Mercator (1999) is a scalable, extensive web crawler entirely in Java.

"By scalable, we mean that Mercator is designed to scale up to the entire web, and has been used to fetch tens of millions of
web documents. We achieve scalability by implementing our data structures so that they use a bounded amount of memory,
regardless of the size of the crawl. Hence, the vast majority of our data structures are stored on disk, and small parts of them
are stored in memory for efficiency."

"By extensible, we mean that Mercator is designed in a modular way, with the expectation that new functionality will be
added by third parties."

## Architecture of a Scalable Web Crawler

The code starts with a list of seed URLs added.

### Steps
Remove a URL from the URL list, determine the IP address of its host name, download the corresponding
document, and extract any links contained in it. For each of the extracted links, ensure that it is an absolute URL, 
and add it to the list of URLs to download, provided it has not been encountered before. If
desired, process the downloaded document in other ways (e.g., index its content). This basic algorithm requires a number of
functional components:

● a component (called the URL frontier) for storing the list of URLs to download; A FIFO Queue does this. 
There is one FIFO subqueue per worker thread. That is, each worker thread removes URLs from exactly one of
the FIFO subqueues. Second, when a new URL is added, the FIFO subqueue in which it is placed is determined by the
URL's canonical host name. Together, these two points imply that at most one worker thread will download documents from
a given web server. The size of the crawl's frontier numbers in the hundreds of millions of URLs. Hence, the
majority of the URLs must be stored on disk. To amortize the cost of reading from and writing to disk, our FIFO subqueue
implementation maintains fixed-size enqueue and dequeue buffers in memory; 

● a component for resolving host names into IP addresses;

● a component for downloading documents using the HTTP/FTP depending on the protocol specified by the url;

● a component for extracting links from HTML documents; and

● a component for determining whether a URL has been encountered before.

### Mercator's Components
Crawling is performed by multiple worker threads, typically numbering in the
hundreds. Each worker repeatedly performs the steps needed to download and process a document.
