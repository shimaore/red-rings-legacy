    legacy_backend = (options) ->
      (source) ->
        legacy source, options

    module.exports = legacy_backend
    legacy = require './legacy'
