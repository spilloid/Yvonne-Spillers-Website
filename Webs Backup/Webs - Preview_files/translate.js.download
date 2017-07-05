/*
 * RequireJS plugin to load Webs translation resources
 *
 * (c) Webs, 2011, written by Adam Solove.
 *
 * Specs:
 *   - If translate is defined by require in the parent window, that instance will be returned instead of a new one.
 *   - If translate is not available from the parent window, the following documentation applies:
 *     - If you want non-English, the page needs a webs.locale
 *     - If you want to load resources from a separate domain (e.g. dynamic.websimages.com)
 *       webs.props.dynamicAssetServer must be defined.
 *     - If you want certain namespaces to be preloaded, define prefetchTranslationNamespaces as an array of
 *       resource namespaces to fetch. This can be useful if the order of generic namespaces and specific
 *       subnamespaces cannot be guaranteed (e.g. webs.bldr.chrome.settings and webs.bldr). It is important that this
 *       require-managed resource is available immediately, otherwise there is no guarantee that the namespaces
 *       will be fetched before requests for more specific subnamespaces (which would be skipped when the more
 *       generic namespace has already been preloaded).
 */
/* global define:false, require:false, webs:false, window:false*/
define(['localize', 'locale', 'jquery'], function(localize, getLocale, $){
	"use strict";
	// First, try to grab translate from the parent frame, so that we can share translation resources
	// across the entire app, instead of re-loading translations in each frame.
	if (typeof(window) !== "undefined" && window.parent && window.parent !== window) {
		// We're going to try to reach into the parent frame, which won't work if it's on a different domain
		// (iframe apps in the VP dashboard), so we need to wrap this in a try to prevent cross-domain scripting
		// protections from killing us.
		try {
			return window.parent.require("translate");
		} catch(e) {
			// Oops, fall through and define it the normal way.
		}
	}

	var loadedNamespaces = {},
		localeName = typeof(webs) !== 'undefined' && webs.locale || "en-US",
		resourceUrl,
		locale = getLocale(localeName),
		NOOP_NAMESPACE = "none"; // To allow "translate!none" to just grab the translate function

	resourceUrl = "/s/resources/" + localeName + "/";
	if(typeof(webs) !== 'undefined' && webs.props && webs.props.dynamicAssetServer) {
		resourceUrl = webs.props.dynamicAssetServer + resourceUrl;
	}

	// If namespace or its parent is already loading/loaded,
	// returns a deferred representing when that is done.
	// Otherwise returns false.
	var namespaceLoading = function(toTest){
		var parts = toTest.split(".");
		for(var i=0; i<parts.length; i++){
			var ns = loadedNamespaces[parts.slice(0, i+1).join(".")];
			if(ns) {
				return ns;
			}
		}
		return false;
	};

	// Returns a deferred representing that the namespace has loaded.
	var loadNamespace = function(namespace){
		var loading = namespaceLoading(namespace);
		if(loading){
			return loading;
		} else {
			var url = resourceUrl + namespace + "/",
				deferred = $.Deferred(),
				promise = deferred.promise();

			require([url + '?callback=define'], function(resources){
				add(resources);
				deferred.resolve(resources);
			});

			loadedNamespaces[namespace] = promise;
			return promise;
		}
	};

	var add = function(data){
		for(var k in data){
			if(data.hasOwnProperty(k)){
				locale[k] = data[k];
			}
		}
	};

	function extend(o, extensions){
		function F() {}
		F.prototype = o;
		var obj = new F();

		for(var k in extensions){
			obj[k] = extensions[k];
		}
		return obj;
	}
	var translate = function(key, vals){
		var resources = extend(locale, vals);
		return localize.localize(key, resources);
	};

	try {
		// We need to upgrade to the newer require so that we can just ask require.specified("...")
		if (require.s.contexts._.specified.prefetchTranslationNamespaces) {
			require(["prefetchTranslationNamespaces"],function(namespaces){
				$.each(namespaces, function(i, namespace){
					loadNamespace(namespace);
				});
			});
		}
	} catch(e) {
		// failure is totally an option
	}

	return {
		load: function(name, req, load, config){
			// Make sure that name exists as a string in the current JS context (so that if an iframe
			// is blown away, the name strings it provided continue to behave properly).
			// See: http://msdn.microsoft.com/en-us/library/gg622929%28v=VS.85%29.aspx?ppud=4
			name = String(name).toString();
			if(!config.isBuild){
				if (name === NOOP_NAMESPACE) {
					load(translate);
				} else {
					loadNamespace(name).then(function(){ load(translate); });
				}
			} else {
				// We currently don't include these in builds so they can be
				// updated dynamically. But we could include them and require
				// a rebuild to get updated translations.
				load();
			}
		},

		translate: translate,
		add: add

	};
});
