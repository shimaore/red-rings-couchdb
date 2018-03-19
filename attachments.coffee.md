Creates a thru stream (that should be applied to the output of the backend).

    ec = encodeURIComponent
    dc = decodeURIComponent

In the parameter we need:
- .url — base url (used to build URIs provided to external users)
- .target — a function that takes and id and returns a set of request options (esp. `.url` which must be a `new URL` object)
- .secret
- .hash (defaults to `sha256`)
- .timeout (defaults to one hour)

    attachments = (our_proxy) ->
      target = our_proxy.target

      our_proxy.target = (pathname) ->
        return null unless $ = pathname.match ///
          ^ / ([^/]+) / (.+) $
        ///
        target? pathname, dc($[1]), $[2]

      {handler,download_uri,upload_uri} = proxy our_proxy

      thru = (stream) ->
        stream.map (msg) ->
          rev = msg.getIn ['value','rev']
          attachments = msg.getIn ['doc','_attachments']
          return msg unless rev? and attachments?

          id = msg.get 'id'
          ec_id = ec id
          msg.withMutations (msg) ->
            attachments.forEach (rec,name) ->
              path = "/#{ec_id}/#{ec name}"
              if rec.has 'stub'
                msg = msg.setIn ['doc','_attachments',name,'download_uri'], download_uri path, rev
              msg = msg.setIn ['doc','_attachments',name,'upload_uri'], upload_uri path, rev
              return
            msg

      {handler,thru}

The `handler` can be used either as a `createServer` request-listener, or as middleware (using Express conventions).
The `thru` can be used as a most.js `.thru` processor on (e.g.) the output of the backe

    module.exports = attachments
    proxy = require './util/attachments-proxy'
    {URL} = require 'url'
