    build_backend = ({changes,view_changes,semantic,get_key,view_key,fromJS}) ->

The backend return a function that can be provided to `backend_join`.

      (sources) ->

For `update` events it will update the document.

        route = sources
          .filter operation UPDATE
          .thru changes_semantic

        route.create.forEach semantic.create
        route.update.forEach semantic.update
        route.delete.forEach semantic.delete

For `subscribe` events it will GET the document and then notify on that current value
and on any future changes. (This is equivalent to a `@most/hold` on the entire database
viewed as a stream, basically, but uses little ressources.)

        subscriptions_keys =
          sources
          .filter operation SUBSCRIBE
          .map Key
          .multicast()

No action is needed for `unsubscribe` events.

Fetch current value, return a message similar to a row from `_all_docs`.

        values =
          subscriptions_keys
          .filter is_string # Can only retrieve string keys from a database
          .map get_key
          .chain most.fromPromise
          .filter not_null

Compute values for wandering-country-view/all by querying the server-side view.

        view_values =
          subscriptions_keys
          .chain view_key


The output is the combination of:

        most.mergeArray [

- subscriptions to document changes in the database

          changes.map (msg) ->
            Immutable.fromJS msg
            .merge
              op: NOTIFY
              value: rev: msg.doc._rev # or msg.changes[0].rev
              doc: fromJS msg.doc
              key: msg.id

- requested documents in the database

          values.map (doc) ->
            Immutable.Map
              op: NOTIFY
              id: doc._id
              key: doc._id
              value: rev: doc._rev
              doc: fromJS doc

- subscriptions to changes in the view

          view_changes.map (msg) ->
            Immutable.fromJS msg
            .set 'op', NOTIFY

- requested entries in the view

          view_values.map (msg) ->
            Immutable.fromJS msg
            .set 'op', NOTIFY

        ]

    module.exports = build_backend
    changes_semantic = require 'red-rings-semantic'
    {operation,Key,is_string,is_object,not_null,has_key} = require 'abrasive-ducks-transducers'
    Immutable = require 'immutable'
    most = require 'most'
    {UPDATE,SUBSCRIBE,UNSUBSCRIBE,NOTIFY} = require 'red-rings/operations'
