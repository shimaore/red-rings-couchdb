Query on the key
----------------

    query_as_stream = (db_uri,app,view,key) ->

      params =
        reduce: false
        sorted: false

      if key?
        params.key = JSON.stringify key

      view_stream db_uri, app, view, params

    module.exports = query_as_stream
    view_stream = require 'most-couchdb/view-stream'
