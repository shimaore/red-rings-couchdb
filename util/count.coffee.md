Reduce the view
---------------

This assumes the view's reducer is `_count`, obviously.

    count_as_stream = (db_uri,app,view,key,detail) ->

      params =
        reduce: true

      if detail? and Immutable.List.isList key
        detail = parseInt detail
        if isNaN(detail) or detail < 0
          return most.empty()

        params.group_level = key.size + detail
        params.startkey = JSON.stringify key
        params.endkey = JSON.stringify key.concat Immutable.Repeat Immutable.Map(), 1
      else
        params.key = JSON.stringify key
        params.group = true

      view_stream db_uri, app, view, params

    module.exports = count_as_stream
    view_stream = require 'most-couchdb/view-stream'
    most = require 'most'
    Immutable = require 'immutable'
