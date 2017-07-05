if(!window.webs) { window.webs = {}; }

define([
	'jquery',
	'translate!webs.creativecommons.attribution'
], function($, translateCC) {
	/**
	 * Creative Commons Attribution
	 * Modules simply need to call webs.page.addCCImage,
	 * passing in a creative commons image object consisting of:
	 *   url - URL where the source image can be found
	 *   artist - Name of the original artist
	 */
	webs.page = webs.page || {};
	webs.page.addCCImage = function(ccImage, selector, toggleEvent) {
		var ccImages = webs.page.ccImages;
		if(!ccImages) {
			ccImages = webs.page.ccImages = [];

			$(function() {
				var createAttributionLink = function(img) {
					return '<a target="_blank" href="' + img.url + '">' + img.artist + '</a>';
				};

				var cc = $('<div/>')
					.attr('id', 'webs-cc')
					.append($('<div/>', {'id': 'webs-cc-mark'}))
					.appendTo(selector || 'body');

				cc.on(toggleEvent || 'mouseenter', function() {
					if($('#webs-cc-full').length === 0) {
						var attrList = $.map(ccImages, createAttributionLink).join(' ');

						$('<div/>')
							.attr('id', 'webs-cc-full')
							.append($('<p/>').html(translateCC('webs.creativecommons.attribution.message')))
							.append($('<p/>').html(translateCC('webs.creativecommons.attribution.photosby') + ' ' + attrList))
							.appendTo(cc);
						cc.addClass('over');

					}
				});
			});
		}

		ccImages.push(ccImage);
	};
});
