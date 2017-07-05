/*
 * Locale system
 *
 */
/*
 * Locale
 *
 * A locale is a language/culture group that has a standard grammar for
 * pluralizing words, and standard ways to format numbers and dates.
 *
 * TODO: use the globalize jquery library to do things other than plurals
 *   Register plurals as another piece of data for globalize
 *   https://github.com/jquery/globalize/blob/master/lib/globalize.js
 */

define(["localize"], function(localize){
	function extend(o, extensions){
		function F() {}
		F.prototype = o;
		var obj = new F();

		for(var k in extensions){
			obj[k] = extensions[k];
		}
		return obj;
	}

	var baseLocale = {
		name: 'base',
		pluralize: function(items, locale){
			var number = localize.localize(items[2], locale);
			var pluralSuffix = locale.pluralTypes[locale.pluralType](number);
			var key = items[1] + "." + pluralSuffix;
			return localize.localize(key, locale);
		},
		pluralTypes: {
			// 1, 0 and many
			"1": function(n){
				return n == "1" ? "1" : "other";
			},
			// 0 and 1, many
			"2": function(n){
				return n == "0" || n == "1" ? "1" : "other";
			}
		},
		formatDate: function(items, locale){
			var date = locale[items[1]];

			date = new Date(date);

			if(!(date instanceof Date)) return "";
			var day = date.getDate();
			var month = date.getMonth() + 1;
			var year = date.getFullYear();

			return month + "/" + day + "/" + year;
		}
	},
	locales = {};

	// Simple Locale factory
	function addLocale(name, pluralType) {
		// default to pluralType 1
		if(typeof(pluralType) === 'undefined') pluralType = 1;

		locales[name] = extend(baseLocale, {
			pluralType: ''+pluralType,
			name: name
		});
	}

	addLocale('da-DK', 1);
	addLocale('de-DE', 1);
	addLocale('en-GB', 1);
	addLocale('en-AU', 1);
	addLocale('en-US', 1);
	addLocale('es-ES', 1);
	addLocale('es-MX', 1);
	addLocale('es-US', 1);
	addLocale('fr-CA', 2);
	addLocale('fr-FR', 2);
	addLocale('it-IT', 1);
	addLocale('nb-NO', 1);
	addLocale('nl-NL', 1);
	addLocale('sv-SE', 1);
	addLocale('zz-ZZ', 1);

	function logMissingLocale(name) {
		if(typeof(console) !== 'undefined' && console.warn) console.warn("No such locale", name);
		if(typeof(mpq) !== 'undefined' && mpq.push) mpq.push(['track', 'localeMissing', {
			locale: name
		}]);
	}

	return function(name){
		var locale = locales[name];
		if(locale) return locale;

		logMissingLocale(name);

		return extend(baseLocale, {});
	};
});
