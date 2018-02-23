    legacy = (options) ->

      redis = new Redis options

      s = most
        .fromEvent 'pmessageBuffer', redis
        .map ([pattern,channel,message]) ->
          [sender,{type,data:[event,data],nsp},{rooms,flags}] = notepack.decode message
          data
        .chain (data) ->
          most
            .from data._in
            .map (key) ->
              op: NOTIFY
              key: "legacy-#{key}"
              value: data

      redis.psubscribe 'socket.io#/#'

      s

    module.exports = legacy

    most = require 'most'
    Redis = require 'ioredis'
    notepack = require 'notepack.io'
    {NOTIFY} = require 'red-rings/operations'
