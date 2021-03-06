View dispatcher
---------------

    view_as_stream = (db,app,view,key,limiter) ->

      params = null
      tap = null

The `key` field (in the `SUBSCRIBE` message) is used to determine what type of query is performed:

      switch

- Map (JSON object) is used to provide one-time queries. These do not support `changes` streaming.

        when Immutable.Map.isMap key

For view queries we need to keep the original `key` field (for routing purposes).

          switch

For `group` queries we send the view's `key` in the `value` field of the message.

            when key.has 'level'
              params = group(
                key.get('level'),
                key.get('min'),
                key.get('max')
              )
              tap = (row) ->
                [ row.key, row.value ] = [ key, row.key ]
                return

For count queries we set the `_key` field to the key generated by the view (the `value` field contains the view's `value`, i.e. the `count`).

            when key.has 'count'
              params = count(
                key.get('count'),
                key.get('detail')
              )
              tap = (row) ->
                if row.key?
                  [ row.key, row._key ] = [ key, row.key ]
                else
                  row.key = key
                return

For range queries we set the `_key` field to the key generated by the view (the `value` field contains the view's `value`).

            when key.has('min') or key.has('max')
              params = range(
                key.get('min'),
                key.get('max')
              )
              tap = (row) ->
                [ row.key, row._key ] = [ key, row.key ]
                return

- string, List, etc., generate regular view queries (and are subject to `changes` streaming, see `most-couchdb/changes-view`).

        else
          params = query key

      return most.empty() unless params?

      S = db.queryStream app, view, params
      if limiter?
        throw new Error "limiter is #{typeof limiter}, expected a function" unless 'function' is typeof limiter
        S = S.pipe limiter()
      S.on 'error', console.error

      M = most
            .fromEvent 'data', S
            .until most.fromEvent 'end', S
      M = M.tap tap if tap?
      M.multicast()

    module.exports = view_as_stream
    query = require './query'
    count = require './count'
    range = require './range'
    group = require './group'
    most = require 'most'
    Immutable = require 'immutable'
