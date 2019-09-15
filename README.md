#这是封装的一些兼容鼠标事件和浏览器中元素position的一些方法
function addEvent(elem, type, fn) {
	if (elem.addEventListener) {
		elem.addEventListener(type, fn, false)
	} else if (elem.attachEvent) {
		elem.attachEvent('on' + type, function() {
			fn.call(elem)
		});
	} else {
		elem['on' + type] = fn;
	}
}

function removeEvent(elem, type, fn) {
	if (elem.addEventListener) {
		elem.removeEventListener(type, fn, false)
	} else if (elem.attachEvent) {
		elem.detachEvent('on' + type, function() {
			fn.call(elem)
		});
	} else {
		elem['on' + type] = fn;
	}
}

function cancelBubble(e) {
	var e = e || window.event;
	if (e.stopPropogation) {
		e.stopPropagation();
	} else {
		e.cancelBubble = true;
	}
}

function preventDefaultEvent(e) {
	var e = e || window.event;
	if (e.preventDefault) {
		e.preventDefault();
	} else {
		e.returnValue = false;
	}
}

function getElemPosition(el) {
	var parent = el.offsetParent,
		offsetLeft = el.offsetLeft,
		offsetTop = el.offsetTop;
	while (parent) {
		offsetLeft += parent.offsetLeft;
		offsetTop += parent.offsetTop;
		parent = parent.offsetParent;
	}
	return {
		left: offsetLeft,
		top: offsetTop
	}
}

function getPagePos(e) {
	var sLeft = getScrollOffset().left,
		sTop = getScrollOffset().top,
		cLeft = document.documentElement.clientLeft || 0,
		cTop = document.documentElement.clientTop || 0;
	return {
		X: e.clientX + sLeft - cLeft,
		Y: e.clientY + sTop - cTop,
	}
}

function getScrollSize() {
	if (document.body.scrollWidth) {
		return {
			width: document.body.scrollWidth,
			height: document.body.scrollHeight
		}
	} else {
		return {
			width: document.documentElement.scrollWidth,
			height: document.documentElement.scrollHeight
		}
	}
}
// 滚动的距离
function getScrollOffset() {
	if (window.pageXOffset) {
		// 所有浏览器 除IE9以及以下
		return {
			left: window.pageXOffset,
			top: window.pageYOffset,
		}
	} else {
		// IE9以及以下 兼容
		return {
			left: document.body.scrollLeft + document.documentElement.scrollLeft, // IE 中两个不可以同时存在,有一个为0
			top: document.body.scrollTop + document.documentElement.scrollTop,
		}
	}
}

function getStyles(elem, prop) {
	if (window.getComputedStyle) {
		if (prop) {
			return parseInt(window.getComputedStyle(elem, null)[prop]);
		} else {
			return window.getComputedStyle(elem, null);
		}
	} else {
		if (prop) {
			return parseInt(elem.currentStyle[prop]);
		} else {
			return elem.currentStyle;
		}
	}
}

function getViewportSize() {
	if (window.innerWidth) {
		return {
			width: window.innerWidth,
			height: window.innerHeight
		}
	} else {
		if (document.compatMode === 'BackCompat') {
			return {
				width: document.body.clientWidth,
				height: document.body.clientHeight
			}
		} else {
			return {
				width: document.documentElement.clientWidth,
				height: document.documentElement.clientHeight
			}
		}
	}
}

function elemDrag(elem) {
	var x, y;
	addEvent(elem, 'mousedown', function(e){
		var e = e || window.event;
		x = getPagePos(e).X - getStyles(elem, 'left'); 
		y = getPagePos(e).Y - getStyles(elem, 'top'); 
		addEvent(document, 'mouseover', mouseMove);
		addEvent(document, 'mouseup', mouseUp);
		cancelBubble(e);
		preventDefaultEvent(e);
		function mouseMove(e){
			var e = e || window.event,
				elemStyle = elem.style;
			elemStyle.left = getPagePos(e).X - x +'px';
			elemStyle.top = getPagePos(e).Y - y +'px';
		}
		function mouseUp(e){
			var e = e || window.event;
			removeEvent(document, 'mouseover', mouseMove);
			removeEvent(document, 'mouseup', mouseUp);
		}
	})
}
