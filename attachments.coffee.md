Creates a thru stream (that should be applied to the output of the backend).

    ec = encodeURIComponent

    attachments = (the_proxy_url,more...) ->

      {handler,download_uri,upload_uri} = proxy the_proxy_url, more...

      {pathname} = new URL the_proxy_url

      thru = (stream) ->
        stream.map (msg) ->
          rev = msg.getIn ['value','rev']
          attachments = msg.getIn ['doc','_attachments']
          return msg unless rev? and attachments?

          base = "#{pathname}/#{ec msg.get 'id'}"
          msg.withMutations (msg) ->
            attachments.forEach (rec,name) ->
              if rec.has 'stub'
                path = "#{base}/#{ec name}"
                msg = msg.setIn ['doc','_attachments',name,'download_uri'], download_uri path, rev
                msg = msg.setIn ['doc','_attachments',name,'upload_uri'], upload_uri path, rev
            msg

      {handler,thru}

The `handler` can be used either as a `createServer` request-listener, or as middleware (using Express conventions).
The `thru` can be used as a most.js `.thru` processor on (e.g.) the output of the backe

    module.exports = attachments
    proxy = require './util/attachments-proxy'
    {URL} = require 'url'
