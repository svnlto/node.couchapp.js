# Installation

Install node.

Install npm.

<pre>
$ git clone repo
$ cd node.couchapp.js
$ npm install
$ npm link .
</pre>

<pre>
$ couchapp help
couchapp -- utility for creating couchapps

Usage:
  couchapp &lt;command> app.js http://localhost:5984/dbname

Commands:
  push : Push app once to server.
  sync : Push app then watch local files for changes.
</pre>

app.js example:

<pre>
  var couchapp = require('couchapp')
    , path = require('path');

  ddoc = {
      _id: '_design/app'
    , views: {}
    , lists: {}
    , shows: {} 
  }

  module.exports = ddoc;

  ddoc.views.byType = {
    map: function(doc) {
      emit(doc.type, null);
    },
    reduce: '_count'
  }

  ddoc.views.peopleByName = {
    map: function(doc) {
      if(doc.type == 'person') {
        emit(doc.name, null);
      }
    }
  }

  ddoc.lists.people = function(head, req) {
    start({
      headers: {"Content-type": "text/html"}
    });
    send("&lt;ul id='people'>\n");
    while(row = getRow()) {
      send("\t&lt;li class='person name'>" + row.key + "&lt;/li>\n");
    }
    send("&lt;/ul>\n")
  }

  ddoc.shows.person = function(doc, req) {
    return {
      headers: {"Content-type": "text/html"},
      body: "&lt;h1 id='person' class='name'>" + doc.name + "&lt;/h1>\n"
    }
  }
  
  ddoc.validate_doc_update = function (newDoc, oldDoc, userCtx) {
    function require(field, message) {
      message = message || "Document must have a " + field;
      if (!newDoc[field]) throw({forbidden : message});
    };

    if (newDoc.type == "person") {
      require("name");
    }
  }

  couchapp.loadAttachments(ddoc, path.join(__dirname, '_attachments'));
</pre>
