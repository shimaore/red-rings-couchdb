    apply_cmd = (doc,cmd) ->
      op = cmd.get 0
      path = cmd.get 1
      args = cmd.skip 2
      json_path doc, path, op, args

    module.exports = apply_cmd
    json_path = require 'red-rings-path'
