    {Transform} = require 'stream'

Roughly, no more than `count` messages per `interval` milliseconds.

    class Limiter extends Transform
      constructor: (count,interval,options = {}) ->
        options.objectMode = true
        super options
        @__count = count
        @__interval = interval
        @__current = 0
        @__start = Date.now()

      _transform: (chunk,encoding,next) ->
        now = Date.now()
        end_of_period = @__start + @__interval
        @__current++

Reset the state if the interval expired.

        if @__current >= @__count and now < end_of_period
          setTimeout ( =>
            @__start = Date.now()
            @__current = 1
            @push chunk
            next()
          ), end_of_period-now
          return

        if now >= end_of_period
          @__start = Date.now()
          @__current = 1

        @push chunk
        next()
        return

    module.exports = Limiter
