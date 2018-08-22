This is a minimalist CouchDB (HTTP) API
It provides exactly what this module needs, but no more.

    class CouchDB

      constructor: (uri) ->
        if uri.match /\/$/
          @uri = uri
        else
          @uri = "#{uri}/"

      put: (doc) ->
        {_id} = doc
        uri = new URL ec(_id), @uri
        agent
        .put uri.toString()
        .type 'json'
        .accept 'json'
        .send doc
        .then ({body}) -> body

      get: (_id,{rev} = {}) ->
        uri = new URL ec(_id), @uri
        uri.searchParams.set 'rev', rev if rev?
        agent
        .get uri.toString()
        .accept 'json'
        .then ({body}) -> body

      delete: ({_id,_rev}) ->
        uri = new URL ec(_id), @uri
        uri.searchParams.set 'rev', _rev if _rev?
        agent
        .delete uri.toString()
        .accept 'json'
        .then ({body}) -> body

      changes: (options = {}) ->
        {live,include_docs,since} = options
        live ?= true
        throw new Error 'Only live streaming is supported' unless live is true

        since ?= 'now'
        uri = new URL '_changes', @uri
        uri.searchParams.set 'feed', 'eventsource'
        uri.searchParams.set 'heartbeat', true
        uri.searchParams.set 'include_docs', include_docs ? false

The stream might end because the server disconnects or other errors.
In all cases we let it finish cleanly.

        s = ->
          uri.searchParams.set 'since', since
          fromEventSource new EventSource uri.toString()
          .continueWith ->
            console.error 'retry', uri.host, uri.pathname, since
            most.never()
          .recoverWith (error) ->
            console.error 'retry', (error.stack ? error), uri.host, uri.pathname, since
            most.never()

        autoRestart(s)
        .map ({data}) -> data
        .map JSON.parse
        .tap ({seq}) -> since = seq if seq?

    module.exports = CouchDB
    ec = encodeURIComponent
    most = require 'most'
    EventSource = require 'eventsource'
    {fromEventSource} = require 'most-w3msg'
    {URL} = require 'url'
    agent = require 'superagent'

When the stream finishes, we restart it with a small delay.

    sleep = (timeout) -> new Promise (resolve) -> setTimeout resolve, timeout

    autoRestart = (s) ->

      most
      .generate -> yield 200
      .startWith 0
      .map (delay) ->
        await sleep delay
        s()
      .chain most.fromPromise
      .switchLatest()
