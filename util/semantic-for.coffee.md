    semantic_for = (db,fromJS) ->

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
          old_doc = doc

The entries in the `.doc` field are interpreted as simple `set` operations.

          doc = doc.merge msg_doc if msg_doc?

The entries in the `.operations` field are executed by `json_path`.
Notice that these might fail, in which case the update will not be saved.

          doc = operations.reduce apply_cmd, doc if operations?

Do not modify the database if the document was not modified.

          return if doc.equals old_doc

          db.put doc.toJS()
        .catch -> yes

      delete: (msg) ->
        id = msg.get 'id'
        rev = msg.get 'rev'
        db
        .delete _id: id, _rev: rev
        .catch -> yes

    module.exports = semantic_for
    apply_cmd = require './apply-command'
