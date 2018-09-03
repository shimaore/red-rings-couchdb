Query on a range
----------------

Note: `min` can also be interpreted as `ge` and `max` as `le`.

    range_as_stream = (db_uri,app,view,min,max) ->

      params =
        reduce: false
        sorted: false
        inclusive_end: true

      if min?
        params.startkey = JSON.stringify min
      if max?
        params.endkey = JSON.stringify max

      view_stream db_uri, app, view, params

    module.exports = range_as_stream
    view_stream = require 'most-couchdb/view-stream'
