
// locale loader -> create locales, adds strings to them
// locale -> contains strings and formatting functions
// localize -> localize, execute, interpolate


define(function(){
	function trim(string){
		return string.replace(/^\s+|\s+$/g, "");
	}

	function execute_command(command, locale){
		var commandItems = trim(command).split(/\s+/);
		var commandName = commandItems[0];

		if(typeof locale[commandName] === "function")
			return locale[commandName].call(locale, commandItems, locale);
	}

	function lookup(key, locale){
		return locale[key];
	}

	function interpolate(string, locale){
		return string.replace(/{([^}]*)}/g, function(match, content){
			return ret.localize(trim(content), locale);
		});
	}

	var ret = {
		localize: function localize(key, locale){
			if(typeof(location) !== 'undefined' && location.search && location.search.indexOf('notranslate=1') > 0) {
				// In case you want to see the keys... put notranslate=1 in the queryString
				return key;
			}

			var translation = lookup(key, locale);
			if(typeof(translation) !== "undefined")
				return interpolate(String(translation), locale);

			translation = execute_command(key, locale);
			if(translation)
				return interpolate(translation, locale);

			// Didn't find key!
			this.logMissingLocalization(key, locale);
			translation = key;

			return translation;
		},

		logMissingLocalization: function logMissingLocalization(key, locale){
			if(typeof(console) !== 'undefined' && console.warn) console.warn("No localized string for", key, locale.name);
			if(typeof(mpq) !== 'undefined' && mpq.push) mpq.push(['track', 'localizationMissing', {
				key: key,
				locale: locale.name
			}]);
		}
	};

	return ret;

});
