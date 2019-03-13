Reduce the view
---------------

This assumes the view's reducer is `_count`, obviously.

    count = (key,detail) ->

      params =
        reduce: true

      if detail? and Immutable.List.isList key
        detail = parseInt detail
        if isNaN(detail) or detail < 0
          return null

        params.group_level = key.size + detail
        params.startkey = key
        params.endkey = key.concat Immutable.Repeat Immutable.Map(), 1
      else
        params.key = key
        params.group = true

      params

    module.exports = count
    Immutable = require 'immutable'
