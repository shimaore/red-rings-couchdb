Web server
----------

    DOWNLOAD = 'get'
    UPLOAD = 'put'
    TOKEN = 'token'
    REV = 'rev'
    UNTIL = 'until'

    attachments_proxy = ( the_proxy_url, the_target_url, {secret,hash,timeout}, request_options = {}) ->

      hash ?= 'sha256'
      timeout ?= 3600*1000 # one hour

      signature = (method,pathname,rev,limit) ->
        {strictEqual} = require 'assert'
        strictEqual 'string', typeof method, 'method'
        strictEqual 'string', typeof pathname, 'pathname'
        strictEqual 'string', typeof rev, 'rev'
        strictEqual 'string', typeof limit, 'limit'
        crypto
        .createHmac hash, secret
        .update method
        .update pathname
        .update rev
        .update limit
        .digest 'hex'

      handler = (req,res,next) ->

        next ?= (msg) ->
          res.writeHead 404
          res.end msg
          return

        proxy = ->
          {method} = req
          target_url = new URL the_target_url
          target_url.pathname = pathname
          target_url.searchParams.set 'rev', rev
          url = target_url.toString()

          the_proxy = request Object.assign {
            method
            url
            followRedirects: false
            maxRedirects: 0
          }, request_options
          req.pipe the_proxy
          the_proxy.pipe res

        {pathname,searchParams} = new URL req.url, the_proxy_url

        token = searchParams.get TOKEN
        return next 'Invalid token' unless token?

        rev = searchParams.get REV
        return next 'Invalid rev' unless rev?

        limit = searchParams.get UNTIL
        return next "Invalid limit #{JSON.stringify limit}" unless limit?.match(/^\d+$/) and parseInt(limit) > Date.now()

        switch req.method
          when 'PUT'
            if token is signature UPLOAD, pathname, rev, limit
              return proxy()
            else
              return next "Invalid token"

          when 'GET', 'HEAD'
            if token is signature DOWNLOAD, pathname, rev, limit
              return proxy()
            else
              return next "Invalid token"

        next()
        return

      uri_maker = (direction) ->
        (path,rev) ->
          url = new URL path, the_proxy_url
          url.searchParams.set REV, rev
          limit = Date.now() + timeout
          url.searchParams.set UNTIL, limit
          url.searchParams.set TOKEN, signature direction, path, rev, limit.toString()
          url.toString()

      download_uri = uri_maker DOWNLOAD
      upload_uri = uri_maker UPLOAD

      {handler,download_uri,upload_uri}

    module.exports = attachments_proxy
    {URL} = require 'url'
    crypto = require 'crypto'
    request = require 'request'
