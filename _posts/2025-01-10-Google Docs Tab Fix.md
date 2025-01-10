---
title: "How to fix the PDF export issues in Google Docs with Document Tabs"
categories:
  - Google
tags:
  - tech
comments: true
---

Since the introduction of Document Tabs in Google Docs, downloading a PDF file without including blank pages
introduced by these tabs has been a challenge, particularly on the Android app.
This issue complicates tasks like exporting a CV, which is typically formatted to fit on a single page.

After some exploration using the API and network inspection tools, I discovered a straightforward way to download the rendered PDF
version of a document without the blank pages:

```
https://docs.google.com/document/u/0/export?format=pdf&id=<DOCUMENT ID>&tab=t.0
```
The document ID is the identifier of the doc, it can be seen when opened on desktop:
```
https://docs.google.com/document/d/<DOCUMENT ID>/edit?tab=t.0
```

Going back to my Android, I can create an icon with a shortcut to the export url and download the file automatically.



