define([
	'jquery',
	'moduleVersions',
	'internal/sitebuilder/common/log',
	'spine',
	'internal/sitebuilder/common/webs.modules',
	'internal/sitebuilder/common/creativeCommons'
], function($, moduleVersions, log, Spine, websModules) {
	var MODULE_LOAD_TIMEOUT = 30000;
	var RESOLVED_PROMISE = $.Deferred().resolve().promise();

	var ModuleClassLoader = Spine.Class.create({
		init: function(options) {
			this.MODULES_URL = options.MODULES_URL;
			this.MODULES_VERSION = options.MODULES_VERSION;

			/**
			* Map of module slugs to classes
			*/
			this.classes = {};

			/**
			* Map from module slugs to a promises for when it loads
			*/
			this.modulePromises = {};

			/**
			 * Map from module slugs to promises for when the JS file loads
			 */
			this.moduleJSPromises = {};
		},

		getClass: function(moduleSlug) {
			return this.classes[moduleSlug];
		},

		/**
		* Called from the module definition file. Each module type is only registered once.
		*/
		register: function(moduleSlug, include, extend) {
			var superClass;
			extend.slug = moduleSlug;

			if(extend.iframe) {
				superClass = websModules.IframeModule;
			} else if(extend.isWidget) {
				superClass = websModules.WidgetModule;
			} else if(extend.submodules) {
				superClass = websModules.CompositeModule;
			} else {
				superClass = websModules.CustomModule;
			}

			this.classes[moduleSlug] = superClass.create(include, extend);
			if(!this.moduleJSPromises[moduleSlug]) {
				this.moduleJSPromises[moduleSlug] = new $.Deferred();
			}
			this.moduleJSPromises[moduleSlug].resolve();
		},

		/**
		 * This create method is used when we KNOW that we have already loaded the module
		 *
		 * @param {ModuleClass} moduleClass
		 * @param {Object} options
		 * @param {$.Deferred} deferred
		 * @returns {*}
		 * @private
		 */
		_create: function(moduleClass, options, deferred) {
			var
				moduleSlug = moduleClass.slug,
				proto = $.extend({}, moduleClass),
				data = $.extend(true, {}, moduleClass.defaultData, options.data),
				style = moduleClass.styles[data._style] || moduleClass.defaultStyle;

			// TODO: HACK for buckets. jQuery deep extend also does arrays :(
			if(data.bucketContents && options.data && options.data.bucketContents) {
				data.bucketContents = options.data.bucketContents;
				// Hack for SITEBUILDER-2017 and SITEBUILDER-2025
				// Sometimes the image values were stored as NaN which caused image to come back as null
				// So we need to default it
				for(var i = 0, len = data.cols.length; i < len; i++) {
					if(!data.cols[i].image) {
						log.error('An image within buckets was null');
						data.cols[i].image = moduleClass.defaultData.cols[0].image;
					}
				}
			}
			if(moduleSlug == 'contact_form' && options.data && options.data.fields) {
				data.fields = options.data.fields;
			}

			var obj = this.getClassForStyle(moduleClass, style).init({
				el: options.container || options.el,
				data: data
			});

			deferred.resolve(obj);

			this.trigger('created', obj);

			return obj;
		},

		/**
		 * Essentially the same as create, but for appModules.
		 * Sidebar modules are different. We don't call 'load' on them.
		 */
		createAppModule: function(moduleSlug, options) {
			var
				moduleClass = this.getClass(moduleSlug),
				deferred = $.Deferred();

			if(!moduleClass) {
				// If module is not registered, register a fake one and log an error
				moduleClass = this.classes[moduleSlug] = websModules.AppModule.create({id: moduleSlug.substring(12)});
				log.error('Unable to load appmodule', moduleSlug);
			}

			this._create(moduleClass, options, deferred);
			return deferred.promise();
		},

		create: function(moduleSlug, options) {
			if(moduleSlug.indexOf('app-sidebar-') === 0) {
				return this.createAppModule(moduleSlug, options);
			}

			var
				self = this,
				moduleClass = this.getClass(moduleSlug),
				deferred = $.Deferred();

			self.load(moduleSlug).done(function() {
				self._create(self.getClass(moduleSlug), options, deferred);
			}).fail(deferred.reject);

			return deferred.promise();
		},

		/**
		* Given a module class and a style, return the proper class to init
		* Styles have their own subclasses of the moduleClass, because they can add methods
		* TODO: This should probably move to Module.getClassForStyle
		*/
		getClassForStyle: function(moduleClass, style) {
			if(style) {
				/* jshint ignore:start */
				// Collect all styles (because styles can inherit from other styles)
				var styles = [];
				do {
					styles.push(style);
				} while(style = moduleClass.styles[style.inherit]);
				/* jshint ignore:end */

				// Iterate through the styles in reverse, so that the base style is first, and the specified style is last
				var
					superClass = moduleClass,
					subClass,
					s;
				for(var i = styles.length - 1; i >= 0; i--) {
					s = styles[i];
					subClass = s.klass;
					if(!subClass) {
						delete s.defaultStyle;

						if(s.global.js) {
							// Extend the super class
							var include = {}, extend = {};
							s.global.js(include, extend);
							s.klass = subClass = superClass.create(include, extend);
						} else {
							// This happens when a style doesn't change the JS, but does change the CSS
							// No JS to add. Just use the super class
							s.klass = subClass = superClass;
						}
					}
					superClass = subClass;
				}

				return superClass;
			} else {
				// How!?!?
				// TODO: In the future, not all modules will need a style
				if(log) {
					log.trigger('Modules', 'info', 'No style found for ' + moduleClass.slug + ' module!', moduleClass);
				}
				return moduleClass;
			}
		},

		getModuleSlugFromUrl: function(url) {
			var urlParts = url.split('/');
			var moduleSlug;
			if(urlParts[2] == 'modules') {
				moduleSlug = urlParts[3];
			} else {
				moduleSlug = urlParts[3] + '_' + urlParts[urlParts.length - 1].replace('.less', '');
			}
			return moduleSlug;
		},

		getModuleVersion: function(slug) {
			var version;
			if(moduleVersions && slug in moduleVersions) {
				version = 'v' + moduleVersions[slug];
			} else {
				log.warn('WARNING: Retrieving unversioned asset for module ' + slug);
				version = this.MODULES_VERSION;
			}
			return version;
		},

		getModuleAssetURL: function(slug, path) {
			return this.MODULES_URL + slug + '/' + this.getModuleVersion(slug) + '/' + path;
		},

		cssPath: function(slug) {
			return this.getModuleAssetURL(slug, 'view.packaged.less');
		},

		/**
		* Load a module class from the backend
		*/
		load: function(slug) {
			if (this.modulePromises[slug]) {
				return this.modulePromises[slug];
			}

			var
				self = this,
				cssLoadedPromise = this.loadCss(this.cssPath(slug)),
				dependenciesLoadedPromise = $.Deferred();
			this.modulePromises[slug] = $.Deferred();

			if (!this.moduleJSPromises[slug]) {
				this.moduleJSPromises[slug] = new $.Deferred();
			}

			this.loadJs(slug);

			this.moduleJSPromises[slug].done(function() {
				self.loadModuleDependencies(slug).done(dependenciesLoadedPromise.resolve).fail(dependenciesLoadedPromise.reject);
			}).fail(dependenciesLoadedPromise.reject);

			// Module is loaded when css and js are all loaded.
			$.when(cssLoadedPromise, dependenciesLoadedPromise)
				.done(self.modulePromises[slug].resolve)
				.fail(self.modulePromises[slug].reject);

			// Trigger error callbacks if not laoded quickly.
			setTimeout(this.modulePromises[slug].reject, MODULE_LOAD_TIMEOUT);

			return this.modulePromises[slug];
		},

		loadAll: function(types) {
			var
				deferredBulkLoad = $.Deferred(),
				typePromises = [];
			for(var i = 0, len = types.length; i < len; i++) {
				typePromises.push(this.load(types[i]));
			}
			$.when.apply($, typePromises).done(deferredBulkLoad.resolve).fail(deferredBulkLoad.reject);
			return deferredBulkLoad.promise();
		},

		loadCss: function(url) {
			// In view mode, all the CSS is already loaded
			return RESOLVED_PROMISE;
		},

		loadJs: function(moduleSlug) {
			var deferred = $.Deferred();
			require([this.getModuleAssetURL(moduleSlug, moduleSlug + '_view.js')], null);
			return deferred;
		},

		/* Loads theme style css and returns a list of theme style js files */
		themeStyleFileList: function(moduleSlug) {
			// Per-Theme Module Styles
			var
				self = this,
				moduleClass = this.getClass(moduleSlug),
				theme = webs.theme,
				files = [],
				cssPromises = [];
			if(theme.moduleStyles && theme.moduleStyles[moduleSlug]) {
				$.each(theme.moduleStyles[moduleSlug], function(styleSlug, def) {
					if(log) {
						log.trigger('Modules', 'debug', 'Loading theme style ' + styleSlug + ' for ' + moduleSlug);
					}
					moduleClass.styles[styleSlug] = def;
					def.slug = styleSlug;
					if(def.global.js) {
						files.push(theme.url + '/' + def.global.js);
					}
					if(def.global.css) {
						cssPromises.push(self.loadCss(theme.url + '/' + def.global.css));
					}

					// Set this as the default style
					//jscs:disable requireDotNotation
					if(def['default']) {
						moduleClass.defaultStyle = def;
					}
					//jscs:enable requireDotNotation
				});
			}
			return [files, $.when.apply($,cssPromises)];
		},

		/* A list of module names the given module depends on */
		moduleDependencyList: function(moduleSlug) {
			var
				submodules = [],
				moduleClass = this.getClass(moduleSlug);

			// Module js dependencies
			if(moduleClass.submodules) {
				$.each(moduleClass.submodules, function(slug, sm) {
					submodules.push(slug);
				});
			}

			return submodules;
		},

		shouldConsolidateAssets: function(moduleSlug) {
			return moduleSlug.indexOf('zumba') === -1 && moduleSlug.indexOf('app-sidebar') === -1;
		},

		/* Loads both theme styles and module dependencies for the given module. */
		loadModuleDependencies: function(moduleSlug) {
			var
				self = this,
				dependencies = this.themeStyleFileList(moduleSlug),
				jsFiles = dependencies[0],
				cssPromise = dependencies[1],
				modules = this.moduleDependencyList(moduleSlug),
				total = jsFiles.length + modules.length,
				loaded = 0;

			if(total === 0) {
				return cssPromise;
			} else {
				var
					deferred = $.Deferred(),
					allRequiredPromises = [cssPromise];
				$.each(modules, function(index, moduleSlug) {
					allRequiredPromises.push(self.load(moduleSlug));
				});
				$.each(jsFiles, function(index, file) {
					var jsPromise = $.Deferred();
					allRequiredPromises.push(jsPromise);
					require([script(file)], jsPromise.resolve);
				});
				$.when.apply($, allRequiredPromises).done(deferred.resolve).fail(deferred.reject);
				return deferred.promise();
			}
		}

	});
	ModuleClassLoader.include(Spine.Events);

	var mcl = ModuleClassLoader.init({
		MODULES_URL: webs.props.dynamicAssetServer + '/s/modules/',
		MODULES_VERSION: webs.props.modulesVersion
	});

	var deferredModules = $.Deferred();
	mcl.renderedModulesPromise = deferredModules.promise();
	$(function() {
		$.when.apply($, webs.renderedModulesPromises || []).done(deferredModules.resolve).fail(deferredModules.reject);
	});

	websModules.ModuleClassLoader = mcl;

	return mcl;
});
