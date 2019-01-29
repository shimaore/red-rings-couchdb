Query on the key
----------------

    query = (key) ->

      params =
        reduce: false
        sorted: false

      if key?
        params.key = JSON.stringify key

      params

    module.exports = query
