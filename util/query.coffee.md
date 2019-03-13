Query on the key
----------------

    query = (key) ->

      params =
        reduce: false
        sorted: false

      if key?
        params.key = key

      params

    module.exports = query
