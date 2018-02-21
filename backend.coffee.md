    legacy_backend = (options) ->
      (source) ->
        legacy options

    module.exports = legacy_backend
    legacy = require './legacy'
