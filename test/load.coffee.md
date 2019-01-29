    ({expect} = require 'chai').should()
    CouchDB = require 'most-couchdb'

    describe 'The module', ->
      it 'should load', ->
        require '../backend'
        require '../attachments'
      it 'should load utils', ->
        require '../util/apply-command'
        require '../util/attachments-proxy'
        require '../util/build-backend'
        require '../util/count'
        require '../util/group'
        require '../util/query'
        require '../util/range'
        require '../util/semantic-for'
        require '../util/view'

    {Transform} = require 'stream'

    describe 'View-as-stream', ->
      m = require '../util/view'

      the_db = new CouchDB "http://#{process.env.COUCHDB_USER}:#{process.env.COUCHDB_PASSWORD}@127.0.0.1:5984/test"

      before ->
        await the_db.create()

      after ->
        await the_db.destroy()

      it 'should get a document', ->
        await the_db.put _id:'hello', content:'world'

        v = m the_db, null, '_all_docs', 'hello'

        await v
          .take 1
          .observe (row) ->
            (expect row).to.have.property 'key', 'hello'
            (expect row).to.have.property 'id', 'hello'
            (expect row).to.have.property 'value'

      it 'should get a document with a limiter', ->
        await the_db.put _id:'hello2', content:'world!'

        Limiter = require './limiter'

        limiter = new Limiter 0, 10

        v = m the_db, null, '_all_docs', 'hello2', -> limiter

        received = 0
        await v
          .observe (row) ->
            (expect row).to.have.property 'key', 'hello2'
            (expect row).to.have.property 'id', 'hello2'
            (expect row).to.have.property 'value'
            limiter.__current.should.eql 1
            limiter.__start.should.gt 0
            received++
        received.should.eql 1
