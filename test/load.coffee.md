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
