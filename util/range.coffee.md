Query on a range
----------------

Note: `min` can also be interpreted as `ge` and `max` as `le`.

    range = (min,max) ->

      params =
        reduce: false
        sorted: false
        inclusive_end: true

      if min?
        params.startkey = JSON.stringify min
      if max?
        params.endkey = JSON.stringify max

      params

    module.exports = range
