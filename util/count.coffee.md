Reduce the view
---------------

This assumes the view's reducer is `_count`, obviously.

    count_as_stream = (db_uri,app,view,key,detail) ->

      detail ?= 0

      params =
        reduce: true
        key: JSON.stringify key
        sorted: false

      if Array.isArray key
        params.group = true
        params.group_level = key.length + detail

      view_stream db_uri, app, view, params

    module.exports = count_as_stream
    view_stream = require './view-stream'
