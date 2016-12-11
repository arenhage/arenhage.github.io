---
layout: post
title: tmp-space a node.js express application
date: 2014-11-06 15:02
author: arenhage
comments: true
categories: [express, gridfs, Home, mongodb, node.js, nodejs]
---
Recently i got interested in trying out the gridfs specification for storing blobs in mongodb. Also it is always fun to play around with node.js so i ended using this for my server setup.

[http://tmp-space.com](http://tmp-space.com)

<!--more-->

<strong>The Application</strong>
The server application is a running node.js application with express as its web-framework. The application exposes a series of express routes which allows the user to create spaces, upload and remove files. Each space created is created with a timer which represents the life-cycle of the newly created space. When the space timer has run out, the space is removed along with all files uploaded to that space, simply put.. its a tmp-space.

There is really no special handling in this application what so ever, it is merely a statement of how easy it actually is to store and manage files within mongodb using the gridfs implementation.

<strong>Just how easy is it?</strong>
Using express we can simply upload a file by form and serverside utilize the temporary files generated to get a hold of our uploaded file data.
In this example lets assume i have an open connection to mongodb using mongoose, which is passed to gridfs to instantiate the gridfs driver. In this example I am using <a href="https://github.com/aheckmann/gridfs-stream" title="gridfs-stream">gridfs-stream</a>.

```javascript
var gfs = Grid(conn.db, mongoose.mongo);
var tempfile = req.files.file.path;
var originalFilename = req.files.file.name;
var writestream = gfs.createWriteStream({
  filename: originalFilename
  }).on('close', function(file) {
    // will send a success to caller
    res.send(file);
  });

// open a stream to the temporary file created by Express...
fs.createReadStream(tempfile)
  .on('end', function() {
  })
  .on('error', function() {
  })
  .pipe(writestream); //pipe it to gfs writestream and store it in the database
```

gridfs will handle all in between logic, adding and removing file metadata and the actual file chunks in our database. The files is stored in two separate collections:

fs.files    -> contains file metadata

fs.chunks   -> contains file chunks

Using gridfs, we can now easily grab files and stream them back with the user request:

```javascript
var readstream = gfs.createReadStream({
  _id: req.params.filesId,
});
readstream.pipe(res);
```

also we can simply modify the response headers depending on how we want to respond to the caller

```javascript
res.setHeader('Content-type', data.contentType);
res.setHeader('Content-disposition', 'attachment; filename=' + data.filename);
```

have a look and try it out! [http://tmp-space.com](http://tmp-space.com)

