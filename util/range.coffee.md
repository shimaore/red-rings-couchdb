Query on a range
----------------

Note: `min` can also be interpreted as `ge` and `max` as `le`.

    range = (min,max) ->

      params =
        reduce: false
        sorted: false
        inclusive_end: true

      if min?
        params.startkey = min
      if max?
        params.endkey = max

      params

    module.exports = range
