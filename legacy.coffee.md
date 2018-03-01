    CHANNEL = 'socket.io#/#'
    NSP = '/'

    legacy = (source,options) ->

      redis = new Redis options
      io = Emitter redis

      source
      .filter operation SUBSCRIBE
      .filter (msg) -> Key(msg).match /^legacy-[^:]+/
      .forEach (msg) ->

        $ = Key(msg).match /^legacy-([^:]+)(?::(\S+))$/
        event = $[1]
        romm = {
          location: 'locations'
        }[event]
        data = $[2]
        io.emit event, data
        # message = notepack.encode [sender,{type,data:[event,data],NSP},{rooms,flags}]
        # redis.publish CHANNEL, message

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

      redis.psubscribe CHANNEL

      s

    module.exports = legacy

    most = require 'most'
    Redis = require 'ioredis'
    notepack = require 'notepack.io'
    Emitter = require 'socket.io-emitter'
    {NOTIFY} = require 'red-rings/operations'
