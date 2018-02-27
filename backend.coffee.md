CouchDB back-end
----------------

TBD / FIXME : attachments
TBD / FIXME : only handle known record types ?

- `db` and `db_uri` refer to the same database.
- `all_map`, `app`, `view` refer to the same view.

    couchdb_backend = (db,db_uri,all_map,app,view,fromJS) ->

      apply_cmd = (doc,cmd) ->
        op = cmd.get 0
        path = cmd.get 1
        args = cmd.skip 2
        json_path doc, path, op, args

      semantic =

        create: (msg) ->
          doc = msg
            .get 'doc'
            .set '_id', msg.get 'id'
          db
          .put doc.toJS()
          .catch -> yes

        update: (msg) ->
          id = msg.get 'id'
          rev = msg.get 'rev'
          msg_doc = msg.get 'doc'
          operations = msg.get 'operations'

          db
          .get id, {rev}
          .then (doc) ->

            doc = fromJS doc

The entries in the `.doc` field are interpreted as simple `set` operations.

            doc = doc.merge msg_doc if msg_doc?

The entries in the `.operations` field are executed by `json_path`.
Notice that these might fail, in which case the update will not be saved.

            doc = operations?.reduce apply_cmd, doc

            db.put doc.toJS()
          .catch -> yes

        delete: (msg) ->
          id = msg.get 'id'
          rev = msg.get 'rev'
          db
          .delete _id: id, _rev: rev
          .catch -> yes

Document changes

      changes = all_changes db
        .multicast()

Changeset for wandering-country-view/all

      view_changes = changes_view all_map, changes
        .multicast()

The backend return a function that can be provided to `backend_join`.

      (sources) ->

For `update` events it will update the document.

FIXME: figure out the pattern to push in bulk

The semantic really is 'UPDATE', not 'OVERWRITE'
  → we could either do bulk-get / bulk-put (aka `_all_docs` and `_bulk_docs`),
  → or use an [update function](http://docs.couchdb.org/en/2.1.1/api/ddoc/render.html#db-design-design-doc-update-update-name)
For now the code (above) does single GET/PUT on updates.

        route = sources
          .filter operation UPDATE
          .thru changes_semantic

        route.create.forEach semantic.create
        route.update.forEach semantic.update
        route.delete.forEach semantic.delete

For `subscribe` events it will GET the document. (This is equivalent to a `@most/hold` on the entire database viewed as a stream, basically.)

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
          .map (key) -> db.get(key).catch -> null
          .chain most.fromPromise
          .filter not_null

Compute values for wandering-country-view/all by querying the server-side view.

        view_values =
          subscriptions_keys
          .chain (key) ->
            view_as_stream db_uri, app, view, key

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

    module.exports = couchdb_backend

    changes_view = require './util/changes-view'
    view_as_stream = require './util/view'
    changes_semantic = require 'red-rings-semantic'
    all_changes = require './util/all-changes'
    json_path = require 'red-rings-path'
    {operation,Key,is_string,is_object,not_null,has_key} = require 'abrasive-ducks-transducers'
    most = require 'most'
    Immutable = require 'immutable'
    {UPDATE,SUBSCRIBE,UNSUBSCRIBE,NOTIFY} = require 'red-rings/operations'
