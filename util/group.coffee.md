    group_as_stream = (db_uri,app,view,level,min,max) ->

      params =
        sorted: false
        inclusive_end: true

      level = parseInt level
      if isNaN(level) or level < 0
        return most.empty()

      params.group_level = level

Use level=0 to simulate CouchDB's `group=false`.
Use level=999 to simulate CouchDB's `group=true` ("exact" grouping).

      if min?
        params.startkey = JSON.stringify min

      if max?
        params.endkey = JSON.stringify max

      view_stream db_uri,app,view, params

    module.exports = group_as_stream
    view_stream = require './view-stream'
    most = require 'most'
