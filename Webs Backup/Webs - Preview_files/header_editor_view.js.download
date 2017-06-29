define(['jquery', 'internal/sitebuilder/common/ModuleClassLoader', 'translate!webs.module.header_editor'],
function($, ModuleClassLoader, translate) {
	var module = {}, extend = {};

	// SubModules

	// Module Styles
	extend.styles = {"default":{"global":{"css":"view.less"},"instance":{"css":"view.each.less"},"slug":"default"}};
	if (!extend.styles['default']['global']) {
		extend.styles['default']['global'] = {};
	}
	extend.styles['default']['global']['js'] = null;
	extend.defaultStyle = extend.styles['default'];

	// View JS
	var headerEditor = {
	oneLoaded: function() {}
};

// On mobile, we want to replace any links with spans
var mobileHeaderEditor = {
	oneLoaded: function() {
		var textLayers = $('.w-header-layer-text'),
			anchors = textLayers.find('a[href]');

		anchors.each(function() {
			$(this).replaceWith('<span>'+this.innerHTML+'</span>');
		});
	}
}

$.extend(module, headerEditor);

// Extend for mobile if on mobile theme.
if (webs && webs.theme && webs.theme.mobile) {
	$.extend(module, mobileHeaderEditor);
}


	return ModuleClassLoader.register('header_editor', module, extend);
});