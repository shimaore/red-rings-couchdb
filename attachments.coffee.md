Creates a thru stream (that should be applied to the output of the backend).

    ec = encodeURIComponent

    attachments = (the_proxy_url,more...) ->

      {handler,download_uri,upload_uri} = proxy the_proxy_url, more...

      {pathname} = new URL the_proxy_url

      thru = (stream) ->
        stream.map (msg) ->
          return msg unless msg.value?.rev? and msg.doc?._attachments?

          _attachments = {}
          rev = msg.value.rev
          for own name, rec of msg.doc._attachments
            do (name,rec) ->
              if rec.stub
                path = "#{pathname}/#{ec msg.id}/#{ec name}"
                rec = Object.assign {
                  download_uri: download_uri path, rev
                  upload_uri: upload_uri path, rev
                }, rec
              _attachments[name] = rec

          doc = Object.assign {}, msg.doc, {_attachments}
          Object.assign {}, msg, {doc}

      {handler,thru}

The `handler` can be used either as a `createServer` request-listener, or as middleware (using Express conventions).
The `thru` can be used as a most.js `.thru` processor on (e.g.) the output of the backe

    module.exports = attachments
    proxy = require './util/attachments-proxy'
    {URL} = require 'url'
