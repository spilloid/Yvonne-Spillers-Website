define([
	'jquery',
	'underscore',
	'internal/common/tooltip'],
function nodeDataTooltip($, _){

	var existingTooltips = [],
		tooltipCount = 0,
		$tt,
		defaults = {
			content:"",
			style:"mini"
		};

	// Merge the default options with any data-tooltip-* options specified in markup
	var parseOptions = function($el){
		var data = _.reduce($el.parents(), function(el, par){
			return _.extend(el, $(par).data());
		}, $el.data());
		var opts = _.reduce(_.keys(data), function(opts, key){
			if(key.substr(0, 7) === "tooltip") {
				opts[key.substr(7).toLowerCase()] = data[key];
			}
			return opts;
		}, _.clone(defaults));

		if(opts.offsetleft || opts.offsettop) {
			opts.offset = [parseInt(opts.offsetleft, 10), parseInt(opts.offsettop, 10)];
		}

		if(opts.anchor === "this") {
			opts.anchor = $el;
		}

		return opts;
	};

	var showTooltipFor = function($el){
		hideTooltip();
		if (!$el.data('existingTooltip')) {
			$tt = $.tooltip(parseOptions($el));
			$tt.prepend($el.data("tooltip"));
			if ($el.data('tooltip-title')) {
				$tt.prepend("<h6>" + $el.data('tooltip-title') + "</h6>");
			}
			existingTooltips[tooltipCount] = $tt;
			$el.data({existingTooltip: tooltipCount});
			tooltipCount++;
		}
		else {
			$tt = existingTooltips[$el.data('existingTooltip')];
		}
		$tt.addClass("active");
		$tt.show();
	};

	var hideTooltip = function(){
		if($tt) {
			$tt.removeClass("active");
			$tt.hide();
		}
	};

	$(function(){
		$("body").on("mouseover", "*[data-tooltip]", function(){
			showTooltipFor($(this));
		}).on("mouseout", "*[data-tooltip]", function(){
			hideTooltip();
		});

		$("body").on("showTooltip", function(e, el){
			showTooltipFor($(el));
		}).on("hideTooltip", function(e, el){
			hideTooltip();
		});
	});

});
