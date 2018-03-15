This is a minimalist CouchDB (HTTP) API
It provides exactly what this module needs, but no more.

    class CouchDB

      constructor: (uri) ->
        if uri.match /\/$/
          @uri = uri
        else
          @uri = "#{uri}/"

      put: ({_id} = doc) ->
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
        console.log uri.toString()
        agent
        .get uri.toString()
        .accept 'json'
        .then ({body}) -> body

      delete: ({_id,_rev}) ->
        uri = new URL ec(_id), @uri
        uri.searchParams.set 'rev', rev if rev?
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

        retry = (s) ->
          console.log 'retry', since
          s()
          .continueWith -> retry s # necessary?
          .recoverWith -> retry s

        retry ->
          uri.searchParams.set 'since', since
          fromEventSource new EventSource uri.toString()
        # MessageEvent {type,data,lastEventId,origin}
        .tap ({seq}) -> since = seq
        .map ({data}) -> data
        .map JSON.parse

    module.exports = CouchDB
    ec = encodeURIComponent
    most = require 'most'
    EventSource = require 'eventsource'
    {fromEventSource} = require 'most-w3msg'
    {URL} = require 'url'
    agent = require 'superagent'
