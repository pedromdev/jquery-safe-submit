(function ($) {
  $.fn.lockableSubmit = function (handler) {
    var $that = $(this)

    if (!$that.data('lockableSubmitHandlers')) {
      $that.data('lockableSubmitHandlers', [])

      $that.submit(function (e) {
        var handlers = $that.data('lockableSubmitHandlers')
        var unlockedHandlers = handlers.filter(function (h) {
          return h.unlocked
        })

        if (unlockedHandlers.length < handlers.length) {
          e.preventDefault()
          e.stopPropagation()

          handlers.filter(function (h) {
            return !h.flagged
          }).forEach(function (h) {
            h.flagged = true

            h(e, function () {
              h.unlocked = true
              $that.submit()
            }, function () {
              h.flagged = false
            })
          })
        } else {
          handlers.forEach(function (h) {
            h.flagged = false
            h.unlocked = false
          })
        }
      })
    }

    $that.data('lockableSubmitHandlers').push(handler.bind(this))

    return this
  }

  $.fn.serviceUrl = function (url, methodOrOptions = 'GET') {
    var $that = $(this)
    var allowedMethods = /^(GET|POST|PUT|PATCH|DELETE)$/i

    if (!$that.data('service:actions')) {
      $that.data('service:actions', [])
    }

    if (typeof methodOrOptions === 'string' && allowedMethods.test(methodOrOptions)) {
      methodOrOptions = { method: methodOrOptions }
    } else if (typeof methodOrOptions !== 'object' || methodOrOptions === null) {
      methodOrOptions = {}
    }

    methodOrOptions.url = url
    $that.data('service:actions').push(methodOrOptions)

    return this
  }

  $.fn.fieldsMap = function (fieldsMap, handlers) {
    var that = this
    var $that = $(this)

    if (!$that.data('service:actions') || $that.data('service:actions').length === 0) {
      console.error('É necessário fazer uma chamada ao método \'serviceUrl\' para configurar um URL de envio')
      return this
    } else if (typeof fieldsMap !== 'object' || fieldsMap === null) {
      console.error('É necessário informar um objeto dos campos a serem enviados para o serviço')
      return this
    }

    if (typeof handlers !== 'object' || handlers === null) {
      handlers = {}
    }

    var serviceAction = $that.data('service:actions').shift()

    if (typeof handlers.onStart !== 'function') {
      handlers.onStart = noop
    }

    if (typeof handlers.onError !== 'function') {
      handlers.onError = noop
    }

    if (typeof handlers.onSucess !== 'function') {
      handlers.onSucess = noop
    }

    return $that.lockableSubmit(function (e, unlock, unflag) {
      try {
        handlers.onStart.call(this, e)

        var fields = parseFieldsMap.call(this, fieldsMap)
        var body = parseFields(fields, serviceAction)

        serviceAction.data = body

        $.ajax(serviceAction).done(function (data) {
          handlers.onSuccess.call(that, data)
          unlock()
        }).fail(function (jqXHR, status, err) {
          unflag()
          handlers.onError.call(that, err)
        })
      } catch (err) {
        unflag()
        handlers.onError.call(this, err)
      }
    })
  }

  function parseFieldsMap (fieldsMap) {
    var that = this

    if (fieldsMap === null) {
      return null
    } else if (Array.isArray(fieldsMap)) {
      return fieldsMap.map(parseFieldsMap.bind(that))
    }

    switch (typeof fieldsMap) {
      case 'function':
        return fieldsMap.call(that)
      case 'string':
        return parseFieldString.call(that, fieldsMap)
      case 'object':
        return Object.keys(fieldsMap).reduce(function (obj, key) {
          return Object.assign(obj, {
            [key]: parseFieldsMap.call(that, fieldsMap[key])
          })
        }, {})
      default:
        return fieldsMap
    }
  }

  function parseFieldString (fieldsMap) {
    var $that = $(this)
    var $fields = null
    var regexStringVariables = /(\{[^}]+\})/g

    try {
      $fields = $that.find(fieldsMap)
    } catch (e) {}

    if ($fields !== null && $fields.length > 0) {
      return $fields.length === 1 ?
        $fields.val() :
        $fields.map(function (i, el) {
          return $(el).val()
        }).get()
    } else if (regexStringVariables.test(fieldsMap)) {
      var auxFieldsMap = fieldsMap
        .match(regexStringVariables)
        .reduce(function (obj, variable) {
          return Object.assign(obj, {
            [variable]: variable.replace(/[\{\}]/g, '')
          })
        }, {})

      auxFieldsMap = parseFieldsMap.call(this, auxFieldsMap)

      Object.keys(auxFieldsMap).forEach(function (field) {
        fieldsMap = fieldsMap.replace(field, auxFieldsMap[field])
      })

      return fieldsMap
    }

    return fieldsMap
  }

  function parseFields (fields, action) {
    switch (action.method) {
      case 'GET':
        return $.params(fields)
      case 'POST':
      case 'PUT':
      case 'PATCH':
      case 'DELETE':
        action.contentType = 'application/json; charset=UTF-8'
        return JSON.stringify(fields)
      default:
        return fields
    }
  }

  function noop () {}
})(jQuery);
