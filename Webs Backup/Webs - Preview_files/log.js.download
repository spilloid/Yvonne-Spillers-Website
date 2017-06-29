if(!window.webs) { window.webs = {}; }
//jscs:disable requireCamelCaseOrUpperCaseIdentifiers
window.webs.log = (function create_webs_log() {
	var
		methods = ['log', 'debug', 'dir', 'info', 'warn', 'error', 'group', 'groupEnd'],
		log = {},
		i, method,

		inArray = function log_inArray(elem, array) {
			if(!array) { return -1; }
			if(typeof array.indexOf === 'function') {
				return array.indexOf(elem);
			}
			for(var i = 0, length = array.length; i < length; i++) {
				if(array[i] === elem) {
					return i;
				}
			}
			return -1;
		},
		getURLParam = function log_getURLParam(name) {
			name = name.replace(/[\[]/,'\\[').replace(/[\]]/,'\\]');
			var
				regexS = '[\\?&]' + name + '=([^&#]*)',
				regex = new RegExp(regexS),
				results = regex.exec(window.location.href);
			return results === null ? '' : results[1];
		};

	// enable logging of certain category
	log.enabled = getURLParam('log') || [];
	log.enable = function log_enable(type) {
		if(inArray(type, log.enabled) === -1) {
			log.enabled.push(type);
		}
	};

	log.trigger = function log_trigger(category, type) {
		var params = Array.prototype.slice.call(arguments, 2);
		params.splice(0,0, '[LOGGING "' + category + '"]');

		if(log.enabled.length === 0 || inArray(category, log.enabled) !== -1) {
			if(typeof(log[type]) === 'function') {
				log[type].apply(log, params);
			}
		}
	};

	/* jshint ignore:start */
	for(i = 0; i < methods.length; i++) {
		method = methods[i];

		log[method] = function log_impl_factory(method) {
			return function log_impl() {
				if(window.console) {
					if(typeof(console[method]) === 'function') {
						console[method].apply(console, Array.prototype.slice.call(arguments));
					} else if(typeof(console.log) === 'function') {
						// IE8 doesn't support debug :(
						console.log.apply(console, Array.prototype.slice.call(arguments));
					}
				}
				if(listeners[method] instanceof Array) {
					for(var i = 0; i < listeners[method].length; i++) {
						listeners[method][i].apply(undefined, Array.prototype.slice.call(arguments));
					}
				}

			};
		}(method);
	}
	/* jshint ignore:end */

	var listeners = {};
	log.bind = function(type, callback) {
		if(!(listeners[type] instanceof Array)) {
			listeners[type] = [];
		}
		listeners[type].push(callback);
	};

	return log;
})();

if(typeof(define) !== 'undefined' && define.amd) {
	define([], function define_log() { return webs.log; });
}
