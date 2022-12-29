//
// vivio.js
//
// vivio runtime
//
// Copyright (C) 2015 - 2021 jones@scss.tcd.ie
//
// 25/06/15	initial prototype
// 08/06/16	multiple players in a web page + namespace
// 30/06/16 multiple layers
// 07/07/16 move execute state and code to generated code module
// 11/07/16 added destroy, moveToFront, moveToBack, moveAfter and moveBefore
// 12/09/16 modified callEventHandler execute wait and non-wait functions
// 01/11/16 avoid using default values for parameters as not handled by IE
// 01/11/16 temporary work around for IE as it doesn't implement Path2D
// 03/01/17 contextMenu
// 11/07/17 GRAB
// 12/07/17	Bezier and Spline
// 14/07/17 clipping
// 20/07/17 args and cookies
// 24/07/17 line caps
// 08/08/17 Arc, Bezier, Pie and Spline
// 09/08/17 TxtOffTracker, RoundedTracker
// 05/09/17 brushes
// 07/09/17 pens
// 14/09/17 fonts
// 26/10/17 global event handlers
// 30/01/18 Menu
// 02/09/20 removed interval parameter from animation functions
// 03/09/20 no longer stops at checkpoints during tests when $testFlag == 1
// 03/09/20 fixed eventTick event handler
// 04/09/20 added zIndex parameter to Layer constructor
// 04/09/20 fixed problems with layer.setOpacity
// 05/10/20 args now use sessionStorage and localStorage
// 07/10/20 added isInClip to test if point inside clipping region (part of hit detection)
// 07/10/20 added eventMM handler and implemented contains()
// 12/10/20 full set of GETTER functions
// 14/10/20 gotoTick now working
// 15/10/20 save and restore Peen, Brush and Font state by iterating over properties
// 16/10/20 save and restore GObj state by iterating over properties
// 17/10/20 used "if (this instanceof Pie)" instead of a PIE flag
// 21/10/20 always save state of objects derived from a GObj (EXTENDEDGOBJ)
// 21/10/20 Layer.setVisible() now working
// 21/10/20 subMenus
// 02/11/20 spline functionality (setNPt, getNPt, getPtX, getPtY, setTension, translatePt...)
// 17/11/20 saving and restoring Layer state correctly
// 20/11/20 added JavaScript constants MIN_SAFE_INTEGER and MAX_SAFE_INTEGER
// 08/10/21 added getUIState and getLayers so VivioIDE can get UI and layer state from browser
// 11/10/21 used default parameters (eg x = 1)
//

//
// tested with Firefox, Chrome, Opera, Edge and Safari
//

"use strict";

//
// VPlayer
//
function VPlayer(canvasID, VCode, args) {

	// exported Vivio constants

	const VERSION				= "VivioJS 20.10";	// year.month

	const RT					= 0x80000000;		// {joj 06/07/17}

	// colour constants

	const BLACK					= 0xff000000;
	const BLUE					= 0xff0000ff;
	const CYAN					= 0xff00ffff;
	const GRAY32				= 0xff202020;
	const GRAY64				= 0xff404040;
	const GRAY96				= 0xff606060;
	const GRAY128				= 0xff808080;
	const GRAY160				= 0xffa0a0a0;
	const GRAY192				= 0xffc0c0c0;
	const GRAY224				= 0xffe0e0e0;
	const GREEN					= 0xff00ff00;
	const MAGENTA				= 0xffff00ff;
	const RED					= 0xffff0000;
	const WHITE					= 0xffffffff;
	const YELLOW				= 0xffffff00;

	// GObj options constants

	const RELATIVE				= 0x00000000;		// relative to previous point
	const ABSOLUTE				= 0x00000001;		// absolute
	const CLOSED				= 0x00000020;		// closed Arc, Bezier or Spline
	const NOATTACH				= 0x00000040;		// NOATTACH

	const HCENTRE				= 0x00000000;
	const HLEFT					= 0x00000100;
	const HRIGHT				= 0x00000200;
	const HMASK					= 0x00000300;
	const VCENTRE				= 0x00000000;
	const VTOP					= 0x00000400;
	const VBOTTOM				= 0x00000800;
	const VMASK					= 0x00000c00;

	const HITINVISIBLE			= 0x00001000;		// {joj 13/07/17}
	const HITWINDOW				= 0x00002000;		// {joj 15/07/17}

	const EXTENDEDGOBJ			= 0x80000000;		// {joj 18/10/20} internal

	// pen constants

	const NULLPEN				= 0;				// type
	const SOLIDPEN				= 1;
	const IMAGEPEN				= 2;

	const SOLID                 = 0;				// style
	const DASH                  = 1;
	const DOT                   = 2;
	const DASH_DOT              = 3;
	const DASH_DOT_DOT          = 4;
	const MAX_STYLE				= 4;				// {joj 07/09/17}

	const BUTT_START			= 0;				// {joj 24/07/17}
	const ROUND_START			= 1;
	const SQUARE_START			= 2;
	const ARROW40_START			= 3;
	const ARROW60_START			= 4;
	const ARROW90_START			= 5;
	const CIRCLE_START			= 6;

	const BUTT_END				= 0 << 16;			// {joj 12/09/17}
	const ROUND_END				= 1 << 16;
	const SQUARE_END			= 2 << 16;
	const ARROW40_END			= 3 << 16;
	const ARROW60_END			= 4 << 16;
	const ARROW90_END			= 5 << 16;
	const CIRCLE_END			= 6 << 16;
	const MAX_CAP				= 6;

	const BEVEL_JOIN			= 0 << 8;			// {joj 12/09/17
	const ROUND_JOIN			= 1 << 8
	const MITRE_JOIN			= 2 << 8;
	const MAX_JOIN				= 2;

	// brush constants

	const NULLBRUSH				= 0;				// {joj 05/09/17}
	const SOLIDBRUSH			= 1;				// {joj 05/09/17}
	const IMAGEBRUSH			= 2;				// {joj 05/09/17}
	const GRADIENTBRUSH			= 3;				// {joj 05/09/17}
	const RADIALBRUSH			= 4;				// {joj 05/09/17}

	// font constants

	const BOLD          		= 1;
	const ITALIC        		= 2;
	const UNDERLINE     		= 4;
	const STRIKETHROUGH 		= 8;
	const SMALLCAPS     		= 16;

	// event constants

	const MM					= 0;				// {joj 11/07/17}
	const MB      				= 1;				// {joj 11/07/17}
	const KEY    				= 2;				// {joj 11/07/17}

	const MB_LEFT				= 0x0001;
	const MB_RIGHT      		= 0x0002;
	const MB_MIDDLE    			= 0x0004;
	const MB_SHIFT      		= 0x0010;
	const MB_CTRL       		= 0x0020;
	const MB_ALT				= 0x0040;

	// menu constants

	const MENU_ITEM				= 0;				// {joj 02/10/20}
	const MENU_SEPARATOR		= 1;				// {joj 02/10/20}
	const MENU_SUBMENU			= 2;				// {joj 02/10/20}

	const MENU_ITEM_CHECKBOX	= 0x0010;			// {joj 28/09/20
	const MENU_ITEM_DISABLED	= 0x0040;			// {joj 28/09/20}

	const MENU_NOHIDE			= 0x0100;			// {joj 22/10/20}

	// internal constants

	const INFOFONTH 			= 14;				// info font height
	const NMBB					= 4;				// max number of mbbs
	const SST					= 1024;				// default save state ticks (1024)

	const TRACKER				= 0;				// tracker event
	const RESUMETHREAD			= 1;				// resume thread event
	const ASYNC					= 2;				// async event

	const PLAY					= 0;				// PLAY mode
	const SINGLESTEP			= 1;				// SINGLESTEP mode
	const SNAPTOCHKPT			= 2;				// SNAPTOCHKPT mode

	const UPDATE				= 1;				// update
	const UPDATEMEMBERS			= 2;				// update group members
	const UPDATEGOBJSLEN		= 4;				// update gobjs length
	const UPDATEGOBJS			= 8;				// update gobjs (gobj deleted or order changed)
	const UPDATEALL				= 16;				// redraw all gobjs or all gobjs on a layer {joj 16/07/17}
	const FIREMOVED				= 1 << 8;			// fire event moved handler

	const REMEMBER				= 1;				// remember event flag
	const PROPAGATE				= 2;				// propagate event flag

	const E$PATH				= 0;				// E$
	const R$PATH				= 1;				// R$

	const MENU_ITEM_TYPE_MASK	= 3;				// menu item type mask
	const MENU_ITEM_BORDER		= 3;				// menu border around items
	const MENU_SEPARATOR_H		= 5;				// menu separator height

	var layer = [];									// layers
	var canvas;										// canvas
	var ctx;										// canvas context
	var inCanvas = 0;								// {joj 16/07/17}

	var overlayLayer = 0;							// overlay canvas for displaying mbbs and menus

	var tick = 0;									// animation tick
	var asyncPhaseUpdate = 0;						// async phase update
	var startTick = 0;								// start tick
	var startTimeStamp;								// start timeStamp
	var tps = 50;									// ticks per second
	var timer = 0;									// animation timer
	var dir = 1;									// -1:backwards 1:forwards
	var infoTipTimer = 0;							//
	var eventQ = 0;									// eventQ
	var asyncEventQ = [];							// asyncEventQ
	var saveEventQ = 0;								// saved event Q
	var playZero = 0;								// play Zero
	var playMode = PLAY;							// play mode
	var lastAsyncEvent = 0;							// last ASYNC event
	var gotoTickExecuted = 0;						// {joj 14/10/20}

	var vx;											// viewport x
	var vy;											// viewport y
	var vw;											// viewport width
	var vh;											// viewport height
	var keepAspectRatio;							// viewport keep aspect ratio

	var sx = 1;										// viewport transform
	var sy = 1;										//
	var tx = 0;										//
	var ty = 0;										//
	var t2D = new T2D();							//

	var hit = 0;									//

	var mouseX;										//
	var mouseY;										//
	var contextMenu = 0;							// context menu
	var grab = 0;									//

	var bgBrush = 0;								// background brush
	var bgPen = 0;									// background pen

	var $g = [];									// global variables
	var pens = [];									// pens
	var brushes = [];								// brushes
	var fonts = [];									// fonts
	var vobjs = [];									// vobjs
	var aobjs = [];									// aobjs
	var threads = [];								// threads
	var arg = {};									// arg (obj used as an associative array)
	var handler = [];								// global event handlers
	var interrupt = [];								// interrupts

	var sst = SST;									// save state every sst ticks (only when going backwards)
	var checkPt = [];								// checkPoints
	var savedState = 0;								// saved state (linked listed)
	var atCheckPoint = 0;							// at a checkPoint

	var showStats = 0;								// show runtime stats
	var showMbbs = 0;								// show update mbbs

	var etStart = 0;								// elapsed time start
	var etf = 0;									// elapsed time forward
	var etb = 0;									// elapsed time backward
	var rtLast = 0;									// run time last frame
	var dtLast = 0;									// draw time last frame
	var nff = 0;									// number of frames forward
	var nfb = 0;									// number of frames backward
	var rtStart = 0;								// run time start
	var rtf = 0;									// run time forward
	var rtb = 0;									// run time backward
	var dtf = 0;									// draw time forward
	var dtb = 0;									// draw time backward

	var isChrome = 0;								// is Chrome
	var isFF = 0;									// is FireFox

	if (args)										// {joj 20/10/16}
		getArgs();									// get args here {joj 20/10/16}

	//
	// LayerState constructor
	//
	function LayerState(tick, next) {
		this.tick = tick;	// asyncPhaseUpdate will be 0
		this.next = next;	// linked list
	}

	//
	// Layer constructor
	//
	// creates a new layer (canvas) with specified zIndex
	//
	// zIndex must be in the range 0 to 999
	// overlay canvas given zIndex = 1000
	//
	// overlay used for displaying menus, tooltips and mbbs
	// overlay created on demand
	//
	// if animation is 100% (fills web page), canvas effectively appended as a child of document body
	// if animation not 100% (part of web page), canvas should be within a <div> element and canvas...
	// appended as a child of the <div> element
	//
	// example html code to create suitable canvas

	// <div style="position:relative;">
	//   <!-- tabindex needed for keyboard input -->
	//	 <canvas id="canvas" tabindex = "1" style="width:95%; height:500; overflow:hidden; display:block;">
	//	   No canvas support
	//   </canvas>
	// </div>
	//
	function Layer(zIndex, overlay) {				// {joj 02/10/20}
		if (zIndex < 0) {
			console.log("Layer zIndex < 0 (" + zIndex + ") set to 0");
			zIndex = 0;
		} else if (zIndex > 999) {
			console.log("Layer zIndex > 999 (" + zIndex + ") set to 999");	// {joj 21/10/20}
			zIndex = 999;

		}
		overlay = overlay || 0;						// {joj 21/10/20}
		if (overlay)								// create ovelay layer canvas if overlay set {joj 21/10/20}
			zIndex = 1000;

		Object.defineProperty(this, "oldMbbs", {value:new Mbbs(), writable:1});						// {joj 21/10/20}
		Object.defineProperty(this, "mbbs", {value:new Mbbs(), writable:1});						// {joj 21/10/20}
		Object.defineProperty(this, "savedState", {value:0, writable:1});							// {joj 21/10/20}
		Object.defineProperty(this, "savedStateGObjs", {value:0, writable:1});						// {joj 17/11/20}
		Object.defineProperty(this, "created", {value:(2*tick + asyncPhaseUpdate), writable:1});	// {joj 21/10/20}
		Object.defineProperty(this, "visible", {value:1, writable:1});								// {joj 21/10/20}
		Object.defineProperty(this, "updFlags", {value:0, writable:1});								// {joj 21/10/20}
		this.flags = 0;

		this.gobjs = [];							// gobjs
		this.lastUpdated = 0;
		this.lastReorder = 0;
		this.opacity = 1;							// {joj 02/10/16}

		if (layer.length == 0) {					// canvas inherited from web page
			this.canvas = canvas;
			this.ctx = ctx;
		} else {
			this.canvas = document.createElement("canvas");		// create new canvas
			//this.canvas.style.width = canvas.style.width;		// keeps IE happy
			//this.canvas.style.height = canvas.style.height;
			this.canvas.style.position = "absolute";
			this.canvas.style.zIndex = zIndex;					// {joj 21/10/20}
			this.canvas.style.pointerEvents = "none";
			this.canvas.style.overflow = "hidden";
			this.canvas.style.display = "block";
			this.canvas.style.left = canvas.offsetLeft + "px";
			this.canvas.style.top = canvas.offsetTop + "px";
			this.canvas.width = canvas.width;
			this.canvas.height = canvas.height;
			canvas.parentNode.appendChild(this.canvas); 		// expects to append to <body> or <div> element
			this.ctx = this.canvas.getContext("2d");
		}

		// layer[] ordered my zIndex
		let i = 0, nlayer = layer.length;
		for (; i != nlayer; i++) {
			if (layer[i].canvas.style.zIndex > zIndex) {
				for (let j = nlayer; j > i; j--)				// make room
					layer[j] = layer[j-1];
				break;
			}
		}
		layer[i] = this;
		//console.log("zIndex=" + layer[nlayer - 1].canvas.style.zIndex);
	}

	//
	// Layer.add
	//
	Layer.prototype.add = function(gobj) {
		this.firstUpdate();
		this.gobjs.push(gobj);
		this.lastUpdate = 2*tick + asyncPhaseUpdate;
	}

	//
	// Layer.firstUpdate
	//
	Layer.prototype.firstUpdate = function() {
		if (playZero == 0 && this.savedState == 0 && this.created == 0)
			this.save(0);
	}

	//
	// Layer.firstUpdateGObjs {joj 17/11/20}
	//
	Layer.prototype.firstUpdateGObjs = function() {
		if (playZero == 0 && this.savedStateGObjs == 0 && this.created == 0)
			this.saveGObjs(0);
	}

	//
	// Layer.moveToBack
	//
	Layer.prototype.moveToBack = function(gobj) {
		//console.log("moveToBack");
		this.firstUpdateGObjs();						// {joj 17/11/20}
		this.gobjs.splice(this.gobjs.indexOf(gobj), 1);	// remove gobj
		this.gobjs.unshift(gobj);						// add to front
		this.lastReorder = 2*tick + asyncPhaseUpdate;	// remember when gobjs last reordered
	}

	//
	// Layer.moveToFront
	//
	Layer.prototype.moveToFront = function(gobj) {
		//console.log("moveToFront");
		this.firstUpdateGObjs();						// {joj 17/11/20}
		this.gobjs.splice(this.gobjs.indexOf(gobj), 1);	// remove gobj
		this.gobjs.push(gobj);							// add to end
		this.lastReorder = 2*tick + asyncPhaseUpdate;	// remember when gobjs last reordered
	}

	//
	// Layer.save
	//
	// save Layer.opacity and Layer.gobjs.length
	//
	// gobjs.length saved so that additional GObjs added to gobjs after the save can be removed whn going back
	//
	Layer.prototype.save = function(saveTick) {
		if ((saveTick == 0) || (this.lastUpdate > 2*this.savedState.tick)) {		// {joj 17/11/20}
			this.savedState = new LayerState(saveTick, this.savedState);
			this.savedState.opacity = this.opacity;
			this.savedState.length = this.gobjs.length;
			this.savedState.lastUpdate = this.lastUpdate;
		}
	}

	//
	// Layer.saveGObjs
	//
	// Layer.gobjs must be saved if reordered (eg moveToFront)
	//
	Layer.prototype.saveGObjs = function(saveTick) {								// {joj 17/11/20}
		if ((saveTick == 0) || (this.lastReorder > 2*this.savedStateGObjs.tick)) {	// {joj 17/11/20}
			this.savedStateGObjs = new LayerState(saveTick, this.savedStateGObjs);	// {joj 17/11/20}
			this.savedStateGObjs.gobjs = this.gobjs.slice();						// {joj 17/11/20} shallow copy
			this.savedStateGObjs.lastReorder = this.lastReorder;					// {joj 17/11/20}
		}
	}

	//
	// Layer.restore
	//
	// restore gobjs
	// restore opacity and gobj.length
	//
	Layer.prototype.restore = function(restoreTick) {

		// restore gobjs
		while (this.savedStateGObjs) {												// {joj 17/11/20}
			if (this.savedStateGObjs.tick <= restoreTick) {							// {joj 17/11/20}
				this.gobjs = this.savedStateGObjs.gobjs.slice();					// {joj 17/11/20} shallow copy
				this.lastReorder = this.savedStateGObjs.lastReorder;				// {joj 17/11/20}
				break;																// {joj 17/11/20}
			}
			this.savedStateGObjs = this.savedStateGObjs.next;						// {joj 17/11/20}
		}

		// restore state
		while (this.savedState) {													// {joj 17/11/20}
			if (this.savedState.tick <= restoreTick) {								// {joj 17/11/20}
				this.opacity = this.savedState.opacity;								// {joj 17/11/20}
				this.gobjs.length = this.savedState.length;							// {joj 17/11/20}
				this.lastUpdate = this.savedState.lastUpdate;						// {joj 17/11/20}
				break;																// {joj 17/11/20}
			}
			this.savedState = this.savedState.next;									// {joj 17/11/20}
		}
	}

	//
	// Layer.setOpacity
	//
	Layer.prototype.setOpacity = function(opacity, nsteps = 0, wait = 0) {
		//console.log("Layer.setOpacity(" + opacity + ")");
		if ((this.opacity == opacity) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.opacity = opacity;
			this.updFlags |= UPDATEALL | UPDATE;
			return 0;
		}
		new LayerOpacityTracker(this, opacity, nsteps, wait);
		return wait;
	}

	//
	// Layer.setVisible {joj 21/10/20}
	//
	// alternative approach of setting canvas.style.visibility = "hidden" means that...
	// contextMenu events no longer triggered
	//
	Layer.prototype.setVisible = function(visible) {
		//console.log("Layer.setVisible(" + visible + ")");
		this.visible = visible;
		this.updFlags |= UPDATEALL | UPDATE;
	}

	//
	// setLayerVisibility {joj 08/10/21}
	//
	function setLayerVisibility(index, visible) {
		layer[index].setVisible(visible)
		drawChanges();
	}

	//
	// TrackerEvent constructor
	//
	function TrackerEvent(tick, tracker) {
		//console.log("TrackerEvent tick=" + tick);
		this.tick = tick | 0;		// make sure tick is an integer
		this.typ = 0;				// tracker:0 thread:1: async:2
		this.tracker = tracker;
		this.next = 0;
	}

	//
	// Event constructor
	//
	function Event(tick, thread) {
		this.tick = tick | 0;		// make sure tick is an integer
		this.typ = 1;				// tracker:0 thread:1: async:2
		this.thread = thread;
		this.next = 0;
	}

	//
	// AsyncEvent constructor
	//
	function AsyncEvent(tick, handler) {
		//console.log("new AsyncEvent tick=" + tick + " ", handler);
		this.tick = tick | 0;		// make sure tick is an integer
		this.typ = 2;				// tracker:0 thread:1: async:2
		this.handler = handler;
	}

	//
	// addToEventQ
	//
	function addToEventQ(ev) {
		let e = eventQ;
		let ee = 0;
		while (e) {
			if (e.tick > ev.tick || ((e.tick == ev.tick) && (e.typ > ev.typ)))
				break;
			ee = e;
			e = e.next;
		}
		if (ee) {
			ev.next = e;
			ee.next = ev;
		} else {
			ev.next = eventQ;
			eventQ = ev;
		}
	}

	//
	// removeFutureAsyncEvents
	//
	// remove future ASYNC events from eventQ and asyncEventQ
	//
	function removeFutureAsyncEvents() {
		//console.log("removeFutureAsyncEvents");
		let e = eventQ;
		let ee = 0;
		while (e) {
			if ((e.typ == ASYNC) && (e.tick >= tick)) {
				if (ee) {
					ee.next = e.next;
				} else {
					eventQ = e.next;
				}
			} else {
				ee = e;
			}
			e = e.next;
		}
		//console.log("BEFORE asyncEventQ length=" + asyncEventQ.length);
		asyncEventQ.splice(asyncEventQ.indexOf(lastAsyncEvent) + 1);
		//console.log("AFTER asyncEventQ length=" + asyncEventQ.length);
	}

	//
	// addToAsyncEventQ
	//
	function addToAsyncEventQ(e) {
		asyncEventQ.push(e);
		lastAsyncEvent = e;
	}

	//
	// Interrupt constructor {joj 15/12/20}
	//
	function Interrupt() {
		this.eventQ = eventQ;
		this.tick = tick;
		this.wasRunning = 0;
		if (isRunning()) {
			this.wasRunning = 1;
			stop();
		}
	}

	//
	// startInterrupt {joj 15/12/20}
	//
	function startInterrupt() {
		//console.log("startInterrupt");
		interrupt.push(new Interrupt());
		eventQ = 0;
		start();
	}

	//
	// endInterrupt {joj 15/12/20}
	//
	function endInterrupt() {
		//console.log("endInterrupt");
		let int = interrupt.pop();
		eventQ = int.eventQ;
		tick = int.tick;
		if (int.wasRunning)
			start();
	}

	//
	// ThreadState constructor
	//
	// a main thread is created initially
	// new threads are created by fork
	// the global variable points to the thread that is currently running
	// the global variable threads contains the list of threads
	// when a thread is executed, its members pc, fp etc are copied to the variables of the same name
	// on thread switch the state is saved in the thread and loaded from the next thread
	// each thread has its own stack
	//
	function ThreadState(next) {
		this.tick = tick;	// asyncPhaseUpdate will be 0
		this.next = next;	// linked list
	}

	//
	// Thread constructor
	//
	function Thread(pc, obj = 0) {
		//console.log("new Thread");
		this.pc = pc;
		this.fp = -1;
		this.sp = -1;
		this.acc = obj;
		this.obj = this.acc;
		this.stack = [];
		this.savedState = 0;
		this.created = 2*tick + asyncPhaseUpdate;
		this.flags = PROPAGATE;		// set by event handlers
		threads.push(this);
	}

	//
	// Thread.save
	//
	Thread.prototype.save = function() {
		this.savedState = new ThreadState(this.savedState);
		this.savedState.pc = this.pc;
		this.savedState.fp = this.fp;
		this.savedState.sp = this.sp;
		this.savedState.acc = this.acc;
		this.savedState.obj = this.obj;
		this.savedState.stack = this.stack.slice(0);			// copy stack
	}

	//
	// Thread.restore
	//
	Thread.prototype.restore = function(toTick) {
		while (this.savedState) {
			if (this.savedState.tick <= toTick) {
				this.pc = this.savedState.pc;
				this.fp = this.savedState.fp;
				this.sp = this.savedState.sp;
				this.acc = this.savedState.acc;
				this.obj = this.savedState.obj;
				this.stack.length = 0;
				for (let i = 0; i < this.savedState.stack.length; i++)	// reuse...
					this.stack[i] = this.savedState.stack[i];			// this.stack
				break;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// $terminateThread
	//
	function $terminateThread(thread) {
		if (savedState && thread.created > 2*savedState.tick)
			threads.splice(threads.indexOf(thread), 1);				// remove thread
	}

	//
	// executeRT
	//
	function executeRT(thread) {									// {joj 06/07/17}
		$execute(thread);
		drawChanges();
	}

	//
	// $addWaitToEventQ
	//
	function $addWaitToEventQ(ticks, thread) {
		if (ticks & RT) {
			setTimeout(executeRT, ticks & ~RT, thread);
		} else {
			addToEventQ(new Event(tick + ticks, thread));
		}
	}

	//
	// callEventHandler(pc, [obj, [args]])
	//
	// pc can be a JavaScript function or the pc of a vivio function in execute
	//
	function callEventHandler(pc, obj = 0) {
		//console.log("callEventHandler pc=", pc, " obj=" + obj);
		let r = 0;
		if (typeof pc == "function") { 							// {joj 12/09/16}
			let args = [];
			for (let i = 2; i < arguments.length; i++)
				args.push(arguments[i]);
			r = pc.apply(obj, args);
		} else {
			let ehThread = new Thread(pc, obj);					// create new event handler thread
			let l = arguments.length - 1;						// number of arguments
			for (let i = l; i >= 2; i--)						// push parameters...
				ehThread.stack[++ehThread.sp] = arguments[i];	// onto stack
			ehThread.stack[++ehThread.sp] = -1;					// dummy return address used to terminate thread
			$execute(ehThread);									// execute thread
			r = ehThread.flags;									// return flags
		}
		//playAddedEvents();										// play events added by handler (eg fork) {joj 22/11/17}
		return r;
	}

	//
	// fork(pc, [obj = 0, [args]])
	//
	// pc can be a function or the pc of a wait function
	// why fork a non wait function?
	// forked function executed immediately
	//
	function fork(pc, obj = 0) {
		//console.log("fork pc=" + pc);
		if (typeof pc == "function") { 							// {joj 11/09/20}
			let args = [];										// not a WAITF...
			for (let i = 2; i < arguments.length; i++)			// so function...
				args.push(arguments[i]);						// is...
			pc.apply(obj, args);								// executed immediately
		} else {
			let fThread = new Thread(pc, obj);					// create new thread
			let l = arguments.length - 1;						// get number of arguments
			for (let i = l; i >= 2; i--)						// push parameters...
				fThread.stack[++fThread.sp] = arguments[i];		// onto stack right to left
			fThread.stack[++fThread.sp] = -1;					// dummy return address used to terminate thread
			let th = $getCurrentThread();						// {joj 02/01/21}
			$execute(fThread);									// {joj 02/01/21}
			$switchToThread(th);								// {joj 02/01/21}
		}
	}

	//
	// Mbb constructor
	//
	// keep normalised
	//	x0 <= x1 and y0 <= y1
	// 	x0, x1, y0, y1 rounded to nearest integer
	//
	function Mbb() {
		if (arguments.length == 1) {			// mbb(mbb1)
			this.x0 = arguments[0].x0;
			this.y0 = arguments[0].y0;
			this.x1 = arguments[0].x1;
			this.y1 = arguments[0].y1;
		} else if (arguments.length == 4) {		// mbb(x0, y0, x1, y1)
			this.x0 = Math.round(arguments[0] <= arguments[2] ? arguments[0] : arguments[2]);
			this.y0 = Math.round(arguments[1] <= arguments[3] ? arguments[1] : arguments[3]);
			this.x1 = Math.round(arguments[0] <= arguments[2] ? arguments[2] : arguments[0]);
			this.y1 = Math.round(arguments[1] <= arguments[3] ? arguments[3] : arguments[1]);
		} else {
			this.x0 = 0;						// mbb()
			this.y0 = 0;
			this.x1 = 0;
			this.y1 = 0;
		}
	}

	//
	// Mbb.add
	//
	Mbb.prototype.add = function() {
		let x0, y0, x1, y1;
		if (arguments.length == 1) {			// mbb.add(mbb1)
			x0 = arguments[0].x0;
			y0 = arguments[0].y0;
			x1 = arguments[0].x1;
			y1 = arguments[0].y1;
		} else if (arguments.length == 4) {		// mbb.add(x0, y0, x1, y1)
			x0 = Math.round(arguments[0] <= arguments[2] ? arguments[0] : arguments[2]);
			y0 = Math.round(arguments[1] <= arguments[3] ? arguments[1] : arguments[3]);
			x1 = Math.round(arguments[0] <= arguments[2] ? arguments[2] : arguments[0]);
			y1 = Math.round(arguments[1] <= arguments[3] ? arguments[3] : arguments[1]);
		}
		if (x0 < this.x0)
			this.x0 = x0;
		if (y0 < this.y0)
			this.y0 = y0;
		if (x1 > this.x1)
			this.x1 = x1;
		if (y1 > this.y1)
			this.y1 = y1;
	}

	//
	// Mbb.addPt
	//
	Mbb.prototype.addPt = function(x, y) {
		if (x < this.x0)
			this.x0 = Math.round(x);
		if (y < this.y0)
			this.y0 = Math.round(y);
		if (x > this.x1)
			this.x1 = Math.round(x);
		if (y > this.y1)
			this.y1 = Math.round(y);
	}

	//
	// Mbb.area
	//
	Mbb.prototype.area = function() {
		return (this.x1 - this.x0) * (this.y1 - this.y0);
	}

	//
	// Mbb.height
	//
	Mbb.prototype.height = function() {
		return this.y1 - this.y0;
	}

	//
	// Mbb.inflate
	//
	// NB: deflate if dx/dy negative
	//
	Mbb.prototype.inflate = function(dx, dy = dx) {
		this.x0 -= dx;
		this.y0 -= dy;
		this.x1 += dx;
		this.y1 += dy;
		return this;
	}

	//
	// Mbb.isEmpty
	//
	Mbb.prototype.isEmpty = function() {
		return this.x0 == 0 && this.y0 == 0 && this.x1 == 0 && this.y1 == 0;
	}

	//
	// Mbb.set
	//
	Mbb.prototype.set = function() {
		Mbb.apply(this, arguments);
	}

	//
	// Mbb.toString
	//
	Mbb.prototype.toString = function() {
		return "mbb x0=" + this.x0 + " y0=" + this.y0 + " x1=" + this.x1 + " y1=" + this.y1;
	}

	//
	// Mbb.width
	//
	Mbb.prototype.width = function() {
		return this.x1 - this.x0;
	}

	//
	// Mbb.getX {joj 12/10/20}
	//
	Mbb.prototype.getX = function() {
		return this.x0;
	}

	//
	// Mbb.getY {joj 12/10/20}
	//
	Mbb.prototype.getY = function() {
		return this.y0;
	}

	//
	// Mbb.getW {joj 12/10/20}
	//
	Mbb.prototype.getW = function() {
		return this.width();
	}

	//
	// Mbb.getH {joj 12/10/20}
	//
	Mbb.prototype.getH = function() {
		return this.height();
	}

	//
	// Mbbs constructor
	//
	// keeps track of upto NMBB areas of screen to be updated
	// mbbs clipped to canvas
	//
	function Mbbs() {
		this.nmbb = 0;
		this.mbb = [];
		for (let i = 0; i < NMBB + 1; i++)		// pre-allocate (for speed?)
			this.mbb[i] = new Mbb();			// extra mbb used by add
		this.mbbu = new Mbb();					// pre-allocate (for speed?)
	}

	//
	// Mbbs.add
	//
	Mbbs.prototype.add = function(mbb) {
		if (mbb.isEmpty())						// quick return if mbb empty
			return;
		this.mbb[this.nmbb].set(mbb);			// copy mbb
		if (this.mbb[this.nmbb].x0 < 0)			// clip mbb to canvas
			this.mbb[this.nmbb].x0 = 0;
		if (this.mbb[this.nmbb].y0 < 0)
			this.mbb[this.nmbb].y0 = 0;
		if (this.mbb[this.nmbb].x1 > canvas.width)
			this.mbb[this.nmbb].x1 = canvas.width;
		let h = (showStats) ? canvas.height - INFOFONTH - 4 : canvas.height;
		if (this.mbb[this.nmbb].y1 > h)
			this.mbb[this.nmbb].y1 = h;
		if (++this.nmbb == 1)					// return if only one mbb
			return;
		let da = Number.MAX_VALUE;
		let index1, index2
		for (let i = 0; i < this.nmbb; i++) {
			for (let j = i + 1; j < this.nmbb; j++) {
				this.mbbu.set(this.mbb[i]);
				this.mbbu.add(this.mbb[j]);
				let da1 = this.mbbu.area() - this.mbb[i].area() - this.mbb[j].area();
				if (da1 < da) {
					da = da1;
					index1 = i;
					index2 = j;
				}
			}
		}
		if (da > 0 && this.nmbb <= NMBB)
			return;
		this.nmbb--;
		this.mbb[index1].add(this.mbb[index2]);
		this.mbb[index2].set(this.mbb[this.nmbb]);
	}

	//
	// Mbbs.draw
	//
	// draw:0 clear draw:1 draw
	// pixel offset of 0.5 so line is drawn anti-aliased (a pixel wide)
	//
	Mbbs.prototype.draw = function(draw) {
		let ctx = overlayLayer.ctx;
		ctx.save();
		if (draw) {
			ctx.strokeStyle = "#808080";
			for (let i = 0; i < this.nmbb; i++)
				ctx.strokeRect(this.mbb[i].x0 + 0.5, this.mbb[i].y0 + 0.5, this.mbb[i].width(), this.mbb[i].height());
		} else {
			for (let i = 0; i < this.nmbb; i++)
				ctx.clearRect(this.mbb[i].x0, this.mbb[i].y0, this.mbb[i].width() + 1, this.mbb[i].height() + 1);

		}
		ctx.restore();
	}

	//
	// Mbbs.overlap
	//
	Mbbs.prototype.overlap = function(mbb) {
		for (let i = 0; i < this.nmbb; i++) {
			if (this.mbb[i].x0 < mbb.x1 && this.mbb[i].x1 > mbb.x0 && this.mbb[i].y0 < mbb.y1 && this.mbb[i].y1 > mbb.y0)
				return 1;
		}
		return 0;
	}

	//
	// Mbbs.toString
	//
	Mbbs.prototype.toString = function() {
		let str = "";
		for (let i = 0; i < this.nmbb; i++)
			str += "Mbbs:" + i + " " + this.mbb[i].toString() + "\n";
		return str;
	}

	//
	// clearMbbs
	//
	function clearMbbs() {
		if (showMbbs) {
			for (let i = 0; i < layer.length; i++)
				layer[i].oldMbbs.draw(0);
		}
	}

	//
	// drawMbbs
	//
	function drawMbbs() {
		if (showMbbs) {
			for (let i = 0; i < layer.length; i++)
				layer[i].oldMbbs.draw(1);
		}
	}

	//
	// swapMbbs
	//
	function swapMbbs(draw) {
		for (let i = 0; i < layer.length; i++) {
			let tmp = layer[i].oldMbbs;
			layer[i].oldMbbs = layer[i].mbbs;
			layer[i].mbbs = tmp;
			layer[i].mbbs.nmbb = 0;
		}
	}

	//
	// createOverlayLayer
	//
	// see Layer comments
	//
	function createOverlayLayer() {
		if (overlayLayer == 0)
			overlayLayer = new Layer(0, 1);
	}

	//
	// VObjState constructor
	//
	function VObjState(next) {
		this.tick = tick;	// asyncPhaseUpdate will be 0
		this.next = next;	// linked list
	}

	//
	// VObj constructor
	//
	// savedState property NOT enumerable so it will NOT be saved by VObj.save()
	//
	function VObj() {
		this.created = 2*tick + asyncPhaseUpdate;
		//this.savedState = 0;
		Object.defineProperty(this, "savedState", {value:0, writable:1});	// non enumerable
		vobjs.push(this);
	}

	//
	// VObj.save
	//
	// ptx and pty stored explictily
	//
	VObj.prototype.save = function() {
		this.savedState = new VObjState(this.savedState);
		//console.log("VObj.save " + this.txt);
		for (let prop in this) {
			if (this.hasOwnProperty(prop)) {
				//console.log("VObj.save saving prop=" + prop);
				this.savedState[prop] = this[prop];
			}
		}
	}

	//
	// VObj.restore
	//
	VObj.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				for (let prop in this) {
					if (this.hasOwnProperty(prop)) {
						//console.log("VObj.restore restoring prop=" + prop);
						this[prop] = this.savedState[prop];
					}
				}
				break;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// ArrayState constructor
	//
	// decided not to derive from ArrayObj
	// pragmatic approach initially
	//
	function ArrayState(next) {
		this.tick = tick;	// asyncPhaseUpdate will be 0
		this.next = next;	// linked list
	}

	//
	// newArrayHelper
	//
	function newArrayHelper(a, index, indices) {
		for (let i = 0; i < indices[index]; i++)
			a[i] = new Array(indices[index + 1]);
		if (index == indices.length - 1)
			return;
		for (let i = 0; i < indices[index]; i++)
			newArrayHelper(a[i], index + 1, indices);
	}

	//
	// newArray
	//
	// handles 1 and 2d arrays directly
	// otherwise created recursively using newArrayHelper
	// savedState not enumerable
	//
	function newArray() {
		//console.log("newArray");
		let a = new Array(arguments[0]);
		if (arguments.length > 1) {
			for (let i = 0; i < arguments[0]; i++)
				a[i] = new Array(arguments[1]);
		}
		if (arguments.length > 2)
			for (let i = 0; i < arguments[0]; i++)
				newArrayHelper(a[i], 1, arguments);
		Object.defineProperty(a, "savedState", {value:0, writable:1});	// {joj 05/08/17}
		aobjs.push(a);
		return a;
	}

	//
	// saveArray
	//
	function saveArray(a) {
		//console.log("saveArray() tick=", tick);
		a.savedState = new ArrayState(a.savedState);
		a.savedState.a = a.slice(0);									// copy a
	}

	//
	// restoreArray
	//
	function restoreArray(a, toTick) {
		//console.log("restoreArray toTick=" + toTick);
		while (a.savedState) {
			if (a.savedState.tick <= toTick) {							//
				a.length = 0;											// resuse...
				for (let i = 0; i < a.savedState.a.length; i++)			// a
					a[i] = a.savedState.a[i];							//
				break;
			}
			a.savedState = a.savedState.next;
		}
	}

	//
	// SavedState constructor
	//
	function SavedState() {
		this.tick = tick;												// asyncPhaseUpdate will be 0
		this.next = savedState;											// linked list
		this.sst = sst;
	}

	//
	// saveState
	//
	function saveState() {
		//console.log("saveState() tick=" + tick);
		savedState = new SavedState();
		savedState.g = $g.slice(0);										// copy globals
		if (playZero == 0) {
			for (let i = 0; i < pens.length; i++)						// save pens
				pens[i].save(tick);
			for (let i = 0; i < brushes.length; i++)					// save brushes
				brushes[i].save(tick);
			for (let i = 0; i < fonts.length; i++)						// save fonts
				fonts[i].save(tick);
		}

		//
		// save GObjs
		//
		// if extended GObj must save (could keep track of changes to non GObj properties)
		// if simple GOBj use firstUpdate() + update() mechanism which means that constant GObjs are never saved
		//
		for (let i = 0; i < layer.length; i++) {						// save layers
			let gobjs = layer[i].gobjs;
			for (let j = 0; j < gobjs.length; j++)						// save gobjs
				if (gobjs[j].options & EXTENDEDGOBJ || playZero == 0)	// {joj 18/10/20}
					gobjs[j].save(tick);
			if (playZero == 0) {										// {joj 17/11/20}
				layer[i].saveGObjs(tick);								// save layer gobjs {joj 17/11/20}
				layer[i].save(tick);									// save layer state
			}
		}
		for (let i = 0; i < vobjs.length; i++)							// save vobjs
			vobjs[i].save();
		for (let i = 0; i < aobjs.length; i++)							// save aobjs
			saveArray(aobjs[i]);
		savedState.thread = $suspendThread();							// save thread and thread state
		for (let i = 0; i < threads.length; i++)						// save threads
			threads[i].save();
		savedState.eventQ = 0;
		let e = eventQ;
		let nee = 0;
		while (e) {
			let ne;
			if (e.typ == TRACKER)	{									// tracker
				ne = new TrackerEvent(e.tick, e.tracker);				// copy and...
				e.tracker.save();										// save TRACKER event
			} else if (e.typ == RESUMETHREAD) {							// copy RESUMETHREAD event
				ne = new Event(e.tick, e.thread);						// CHECK e.thread or thread
			}															// skip async events
			if (nee == 0) {
				savedState.eventQ = ne;
			} else {
				nee.next = ne;
			}
			e = e.next;
			nee = ne;
		}
		savedState.lastAsyncEvent = lastAsyncEvent;
	}

	//
	// restoreState
	//
	function restoreState(restoreTick) {
		while (savedState) {
			if (savedState.tick <= restoreTick)
				break;
			savedState = savedState.next;
		}
		//console.log("restoreState(" + restoreTick + ") from savedState.tick=" + savedState.tick);
		tick = savedState.tick;
		sst = savedState.sst;
		for (let i = 0; i < checkPt.length; i++) {
			if (checkPt[i] > tick) {
				checkPt.splice(i);
				break;
			}
		}
		$g.length = 0;
		for (let i = 0; i < savedState.g.length; i++)			// restore globals
			$g[i] = savedState.g[i];
		for (let i = 0; i < pens.length; i++)					// restore pens
			pens[i].restore(tick);
		for (let i = 0; i < brushes.length; i++)				// restore brushes
			brushes[i].restore(tick);
		for (let i = 0; i < fonts.length; i++)					// restore fonts
			fonts[i].restore(tick);
		for (let i = 0; i < layer.length; i++) {				// restore layers
			let gobjs = layer[i].gobjs;
			for (let j = 0; j < gobjs.length; j++) {			// restore gobjs
				if (gobjs[j].created > 2*tick) {				// if created after tick...
					layer[i].mbbs.add(gobjs[j].mbb);			// add mbb to layer mbbs...
				} else {										// so area of screen cleared...
					gobjs[j].restore();							// otherwise restore gobj
				}
			}
			layer[i].restore(tick);								// restore layer (gobjs and state)
		}
		for (let i = 0; i < vobjs.length ; i++) {				// restore vobjs
			if (vobjs[i].created > 2*tick) {					// remove vobjs created after tick
				vobjs.splice(i);
				break;
			} else {
				vobjs[i].restore(tick);							// restore vobjs if created on or before tick
			}
		}
		for (let i = 0; i < aobjs.length; i++)					// restore array objs
			restoreArray(aobjs[i], tick);
		for (let i = threads.length - 1; i >= 0; i--) {			// restore threads
			if (threads[i].created > 2*tick) {					// remove thread created after tick
				threads.splice(i);
				continue;
			}
			threads[i].restore(tick);							// restore threads created on or before tick
		}
		//console.log("restoreB threads.length=" + threads.length);
		$resumeThread(savedState.thread);						// resume thread
		eventQ = 0;												// restore eventQ
		let e = savedState.eventQ;
		let nee = 0;
		while (e) {
			let ne = 0;
			if (e.typ == TRACKER) {								// tracker
				//console.log("restore TRACKER event tick=" + e.tick);
				ne = new TrackerEvent(e.tick, e.tracker);
				e.tracker.restore(tick);						// restore tracker event
			} else if (e.typ == RESUMETHREAD) {					// restore RESUMETHREAD event
				//console.log("restore RESUMETHREAD event tick=" + e.tick);
				ne = new Event(e.tick, e.thread);
			}													// skip async events
			if (nee == 0) {
				eventQ = ne;
			} else {
				nee.next = ne;
			}
			nee = ne;
			e = e.next;
		}
		for (let i = 0; i < asyncEventQ.length; i++) {			// process asyncEventQ
			if (asyncEventQ[i].tick >= tick) {					// transfer while >= tick
				addToEventQ(asyncEventQ[i]);					// transfer event from asyncEventQ to eventQ
			}
		}
		lastAsyncEvent = savedState.lastAsyncEvent;
	}

	//
	// TrackerState constructor
	//
	function TrackerState(next) {
		this.tick = tick;	// asyncPhaseUpdate will be 0
		this.next = next;	// linked list
	}

	//
	// ArcRotateTracker constructor
	//
	// NB: v used to select startAngle (v == 0) or spanAngle (v == 1)
	// NB: nsteps and wait will be defined
	// NB: ddTheta reduces the accumulation of errors
	//
	function ArcRotateTracker(g, v, theta, nsteps, wait) {
		this.g = g;
		this.v = v;
		this.dTheta = theta;
		this.ddTheta = 0;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// ArcRotateTracker.save
	//
	ArcRotateTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddTheta = this.ddTheta;
	}

	//
	// ArcRotateTracker.restore
	//
	ArcRotateTracker.prototype.restore = function() {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddTheta = this.savedState.ddTheta;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// ArcRotateTracker.action
	//
	ArcRotateTracker.prototype.action = function() {
		this.step++;
		let nddTheta = this.step * this.dTheta / this.nsteps;
		if (nddTheta != this.ddTheta) {
			this.g.firstUpdate();
			if (this.v) {
				this.g.spanAngle += nddTheta - this.ddTheta;
			} else {
				this.g.startAngle += nddTheta - this.ddTheta;
			}
			this.g.update();
			this.ddTheta = nddTheta;
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// RGBATracker constructor
	//
	// NB: nsteps and wait will be valid
	// NB: ddR0 to reduce the accumulation of errors
	//
	function RGBATracker(brush, rgba0, rgba1, nsteps, wait) {
		this.brush = brush;
		this.b = (rgba1 === undefined) ? 0 : 1;
		this.dR0 = (rgba0 >> 16 & 0xff) - (brush.rgba0 >> 16 & 0xff);		// NB: integer
		this.dG0 = (rgba0 >> 8 & 0xff) - (brush.rgba0 >> 8 & 0xff);
		this.dB0 = (rgba0 & 0xff) - (brush.rgba0 & 0xff);
		this.dA0 = (rgba0 >> 24 & 0xff) - (brush.rgba0 >> 24 & 0xff);
		this.ddA0 = 0;
		this.ddR0 = 0;
		this.ddG0 = 0;
		this.ddB0 = 0;
		if (this.b) {
			this.dR1 = (rgba1 >> 16 & 0xff) - (brush.rgba1 >> 16 & 0xff);	// NB: integer
			this.dG1 = (rgba1 >> 8 & 0xff) - (brush.rgba1 >> 8 & 0xff);
			this.dB1 = (rgba1 & 0xff) - (brush.rgba1 & 0xff);
			this.dA1 = (rgba1 >> 24 & 0xff) - (brush.rgba1 >> 24 & 0xff);
			this.ddR1 = 0;
			this.ddG1 = 0;
			this.ddB1 = 0;
			this.ddA1 = 0;
		}
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// RGBATracker.save
	//
	RGBATracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddR0 = this.ddR0;
		this.savedState.ddG0 = this.ddG0;
		this.savedState.ddB0 = this.ddB0;
		this.savedState.ddA0 = this.ddA0;
		if (this.b) {
			this.savedState.ddR1 = this.ddR1;
			this.savedState.ddG1 = this.ddG1;
			this.savedState.ddB1 = this.ddB1;
			this.savedState.ddA1 = this.ddA1;
		}
	}

	//
	// RGBATracker.restore
	//
	RGBATracker.prototype.restore = function() {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddR0 = this.savedState.ddR0;
				this.ddG0 = this.savedState.ddG0;
				this.ddB0 = this.savedState.ddB0;
				this.ddA0 = this.savedState.ddA0;
				if (this.b) {
					this.ddR1 = this.savedState.ddR1;
					this.ddG1 = this.savedState.ddG1;
					this.ddB1 = this.savedState.ddB1;
					this.ddA1 = this.savedState.ddA1;
				}
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// RGBATracker.action
	//
	RGBATracker.prototype.action = function() {
		this.step++;
		let nddR0 = this.step*this.dR0/this.nsteps | 0;		// NB: integer
		let nddG0 = this.step*this.dG0/this.nsteps | 0;
		let nddB0 = this.step*this.dB0/this.nsteps | 0;
		let nddA0 = this.step*this.dA0/this.nsteps | 0;
		if (this.b) {
			var nddR1 = this.step*this.dR1/this.nsteps | 0;	// NB: var for function scope
			var nddG1 = this.step*this.dG1/this.nsteps | 0;
			var nddB1 = this.step*this.dB1/this.nsteps | 0;
			var nddA1 = this.step*this.dA1/this.nsteps | 0;
		}
		let b0 = (nddR0 != this.ddR0) || (nddG0 != this.ddG0) || (nddB0 != this.ddB0) || (nddA0 != this.ddA0);
		let b1 = this.b && ((nddR1 != this.ddR1) || (nddG1 != this.ddG1) || (nddB1 != this.ddB1) || (nddA1 != this.ddA1));
		if (b0 || b1) {
			this.brush.firstUpdate();
			if (b0) {
				let r = ((this.brush.rgba0 >> 16) & 0xff) + (nddR0 - this.ddR0);
				let g = ((this.brush.rgba0 >> 8) & 0xff) + (nddG0 - this.ddG0);
				let b = (this.brush.rgba0 & 0xff) + (nddB0 - this.ddB0);
				let a = ((this.brush.rgba0 >> 24) & 0xff) + (nddA0 - this.ddA0);
				this.brush.rgba0 = (a << 24) + (r << 16) + (g << 8) + b;
				this.ddR0 = nddR0;
				this.ddG0 = nddG0;
				this.ddB0 = nddB0;
				this.ddA0 = nddA0;
			}
			if (b1) {
				let r = ((this.brush.rgba1 >> 16) & 0xff) + (nddR1 - this.ddR1);
				let g = ((this.brush.rgba1 >> 8) & 0xff) + (nddG1 - this.ddG1);
				let b = (this.brush.rgba1 & 0xff) + (nddB1 - this.ddB1);
				let a = ((this.brush.rgba1 >> 24) & 0xff) + (nddA1 - this.ddA1);
				this.brush.rgba1 = (a << 24) + (r << 16) + (g << 8) + b;
				this.ddR1 = nddR1;
				this.ddG1 = nddG1;
				this.ddB1 = nddB1;
				this.ddA1 = nddA1;
			}
			this.brush.setFillStyle();
			this.brush.update();
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// ToggleTracker constructor
	//
	// NB: n, ticks and wait will be defined
	// NB: added textPen1 {j0j 20/11/20}
	//
	function ToggleTracker(g, brush1, pen1, txtPen1, n, ticks, wait) {
		//console.log("new ToggleTracker tick=" + tick);
		this.g = g;
		this.brush0 = g.brush;
		this.brush1 = brush1;
		this.pen0 = g.pen;
		this.pen1 = pen1;
		this.txtPen0 = g.txtPen;	// {joj 20/11/20}
		this.txtPen1 = txtPen1;		// {joj 20/11/20}
		this.n = n;
		this.ticks = ticks;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + this.ticks, this));
	}

	//
	// ToggleTracker.save
	//
	ToggleTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
	}

	//
	// ToggleTracker.restore
	//
	ToggleTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				//console.log("ToggleTracker restore step=" + this.step);
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// ToggleTracker.action
	//
	ToggleTracker.prototype.action = function() {
		this.step++;
		this.g.firstUpdate();
		this.g.setBrush((this.step & 1) ? this.brush1 : this.brush0);		// g.update() called
		this.g.setPen((this.step & 1) ? this.pen1 : this.pen0);				// g.update() called
		this.g.setTxtPen((this.step & 1) ? this.txtPen1 : this.txtPen0);	// {joj 20/11/20}
		if (this.step < 2*this.n) {
			addToEventQ(new TrackerEvent(tick + this.ticks, this));
		} else if (this.wait) {
			//console.log("ToggleTracker complete tick=" + tick);
			$execute(this.thread);
		}
	}

	//
	// FontSzTracker constructor
	//
	// NB: nsteps and wait will be defined
	// NB: ddSz reduces the accumulation of errors
	//
	function FontSzTracker(font, dsz, nsteps, wait) {
		this.font = font;
		this.dSz = dsz;
		this.ddSz = 0;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// FontSzTracker.save
	//
	FontSzTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddSz = this.ddSz;
	}

	//
	// FontSzTracker.restore
	//
	FontSzTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddSz = this.savedState.ddSz;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// FontSzTracker.action
	//
	FontSzTracker.prototype.action = function() {
		this.step++;
		let nddSz = this.step*this.dSz/this.nsteps;
		if (nddSz != this.ddSz) {
			this.font.firstUpdate();
			this.font.sz += (nddSz - this.ddSz);
			this.font.setCtxFont();
			this.font.update();
			this.ddSz = nddSz;
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// LayerOpacityTracker constructor
	//
	// NB: nsteps and wait will be defined
	// NB: ddOpacity reduces the accumulation of errors
	//
	function LayerOpacityTracker(g, opacity, nsteps, wait) {
		//console.log("new LayerOpacityTracker opacity=" + opacity + " nsteps=" + nsteps + " wait=" + wait + " [tick=" + tick + "]");
		this.g = g;
		this.dOpacity = opacity - g.opacity;	// relative
		this.ddOpacity = 0;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// LayerOpacityTracker.save
	//
	LayerOpacityTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddOpacity = this.ddOpacity
	}

	//
	// LayerOpacityTracker.restore
	//
	LayerOpacityTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddOpacity = this.savedState.ddOpacity;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// LayerOpacityTracker.action
	//
	LayerOpacityTracker.prototype.action = function() {
		this.step++;
		let nddOpacity = this.step * this.dOpacity / this.nsteps;
		if (nddOpacity != this.ddOpacity) {
			this.g.firstUpdate();
			this.g.opacity += nddOpacity - this.ddOpacity;
			if (this.g.opacity < 0) {
				this.g.opacity = 0;
			} else if (this.g.opacity > 1) {
				this.g.opacity = 1;
			}
			//console.log("layer opacity=" + this.g.opacity);
			this.g.flags |= UPDATEALL;
			this.ddOpacity = nddOpacity;
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			//console.log("OpacityTracker complete tick=" + tick);
			$execute(this.thread);	// execute suspended thread
		}
	}

	//
	// OpacityTracker constructor
	//
	// NB: nsteps and wait will be defined
	// NB: ddOpacity reduces the accumulation of errors
	//
	function OpacityTracker(g, opacity, nsteps, wait) {
		//console.log("new OpacityTracker opacity=" + opacity + " nsteps=" + nsteps + " wait=" + wait + " [tick=" + tick + "]");
		this.g = g;
		this.dOpacity = opacity - g.opacity;	// relative
		this.ddOpacity = 0;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// OpacityTracker.save
	//
	OpacityTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddOpacity = this.ddOpacity
	}

	//
	// OpacityTracker.restore
	//
	OpacityTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddOpacity = this.savedState.ddOpacity;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// OpacityTracker.action
	//
	OpacityTracker.prototype.action = function() {
		this.step++;
		let nddOpacity = this.step * this.dOpacity / this.nsteps;
		//console.log("opacity=" + (this.g.opacity + nddOpacity - this.ddOpacity));
		if (nddOpacity != this.ddOpacity) {
			this.g.firstUpdate();
			this.g.opacity += nddOpacity - this.ddOpacity;
			if (this.g.opacity < 0) {
				this.g.opacity = 0;
			} else if (this.g.opacity > 1) {
				this.g.opacity = 1;
			}
			this.g.update(UPDATEMEMBERS);
			this.ddOpacity = nddOpacity;
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			//console.log("OpacityTracker complete tick=" + tick);
			$execute(this.thread);	// execute suspended thread
		}
	}

	//
	// PenRGBATracker
	//
	// NB: nsteps and wait will be defined
	// NB: ddR, ddG and ddB reduce the accumulation of errors
	//
	function PenRGBATracker(pen, rgba, nsteps, wait) {
		this.pen = pen;
		this.dA = (rgba >> 24 & 0xff) - (pen.rgba >> 24 & 0xff);	// NB: integer
		this.dR = (rgba >> 16 & 0xff) - (pen.rgba >> 16 & 0xff);
		this.dG = (rgba >> 8 & 0xff) - (pen.rgba >> 8 & 0xff);
		this.dB = (rgba & 0xff) - (pen.rgba & 0xff);
		this.ddA = 0;
		this.ddR = 0;
		this.ddG = 0;
		this.ddB = 0;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// PenRGBATracker.save
	//
	PenRGBATracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddR = this.ddR;
		this.savedState.ddG = this.ddG;
		this.savedState.ddB = this.ddB;
		this.savedState.ddA = this.ddA;
	}

	//
	// PenRGBATracker.restore
	//
	PenRGBATracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddR = this.savedState.ddR;
				this.ddG = this.savedState.ddG;
				this.ddB = this.savedState.ddB;
				this.ddA = this.savedState.ddA;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// PenRGBATracker.action
	//
	PenRGBATracker.prototype.action = function() {
		this.step++;
		let nddR = this.step*this.dR/this.nsteps | 0;	// NB: integer
		let nddG = this.step*this.dG/this.nsteps | 0;
		let nddB = this.step*this.dB/this.nsteps | 0;
		let nddA = this.step*this.dA/this.nsteps | 0;
		if ((nddR != this.ddR) || (nddG != this.ddG) || (nddB != this.ddB) || (nddA != this.ddA)) {
			this.pen.firstUpdate();
			let r = ((this.pen.rgba >> 16) & 0xff) + (nddR - this.ddR);
			let g = ((this.pen.rgba >> 8) & 0xff) + (nddG - this.ddG);
			let b = (this.pen.rgba & 0xff) + (nddB - this.ddB);
			let a = ((this.pen.rgba >> 24) & 0xff) + (nddA - this.ddA);
			this.pen.rgba = (a << 24) + (r << 16) + (g << 8) + b;
			this.pen.strokeStyle = toRGBA(this.pen.rgba);
			this.ddR = nddR;
			this.ddG = nddG;
			this.ddB = nddB
			this.ddA = nddA;
			this.pen.update();
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// PenWidthTracker
	//
	// NB: nsteps and wait will be defined
	// NB: ddWidth reduces the accumulation of errors
	//
	function PenWidthTracker(pen, width, nsteps, wait) {
		this.pen = pen;
		this.dWidth = width - pen.width;	// relative
		this.ddWidth = 0;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// PenWidthTracker.save
	//
	PenWidthTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddWith = this.ddWidth;
	}

	//
	// PenWidthTracker.restore
	//
	PenWidthTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddWIdth = this.savedState.ddWidth;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// PenWidthTracker.action
	//
	PenWidthTracker.prototype.action = function() {
		this.step++;
		let w = this.dWidth * this.step / this.nsteps;
		if (w != this.ddWidth) {
			this.pen.firstUpdate();
			this.pen.width += (w - this.ddWidth);
			this.pen.update();
			this.ddWidth = w;
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// PtTracker
	//
	// NB: nsteps and wait will be defined
	// NB: uses ddx and ddy to reduce the accumulation of errors
	//
	function PtTracker(g, n, dx, dy, nsteps, wait) {
		this.g = g;
		this.n = n;
		this.dx = dx;
		this.dy = dy;
		this.ddx = 0;
		this.ddy = 0;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// PtTracker.save
	//
	PtTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddx = this.ddx;
		this.savedState.ddy = this.ddy;
	}

	//
	// PtTracker.restore
	//
	PtTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddx = this.savedState.ddx;
				this.ddy = this.savedState.ddy;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// PtTracker.action
	//
	PtTracker.prototype.action = function() {
		this.step++;
		let nddx = this.step * this.dx / this.nsteps;
		let nddy = this.step * this.dy / this.nsteps;
		if (nddx || nddy) {
			this.g.firstUpdate();
			this.g.ptx[this.n] += nddx - this.ddx;
			this.g.pty[this.n] += nddy - this.ddy;
			this.g.calculateMbb0();
			this.g.update();
			this.ddx = nddx;
			this.ddy = nddy;
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// RectPtTracker {joj 24/09/20}
	//
	// n will be 0 or 1
	// steps and wait will be defined
	// ddx and ddy reduce the accumulation of errors
	//
	function RectPtTracker(g, n, dx, dy, nsteps, wait) {
		this.g = g;
		this.n = n;
		this.dx = dx;
		this.dy = dy;
		this.ddx = 0;
		this.ddy = 0;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// RectPtTracker.save
	//
	RectPtTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddx = this.ddx;
		this.savedState.ddy = this.ddy;
	}

	//
	// RectPtTracker.restore
	//
	RectPtTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddx = this.savedState.ddx;
				this.ddy = this.savedState.ddy;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// RectPtTracker.action {joj 24/09/20}
	//
	// this.n will be 0 or 1
	//
	RectPtTracker.prototype.action = function() {
		this.step++;
		let nddx = this.step * this.dx / this.nsteps;
		let nddy = this.step * this.dy / this.nsteps;
		if (nddx || nddy) {
			this.g.firstUpdate();
			this.g.ptx[this.n] += nddx - this.ddx;
			this.g.pty[this.n] += nddy - this.ddy;
			this.g.calculateMbb0();
			this.g.update();
			this.ddx = nddx;
			this.ddy = nddy;
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// SplinePtTracker {joj 02/11/20}
	//
	// NB: nsteps and wait will be defined
	// NB: uses ddx and ddy to reduce the accumulation of errors
	//
	function SplinePtTracker(g, n, dx, dy, nsteps, wait) {
		this.g = g;
		this.n = n;
		this.dx = dx;
		this.dy = dy;
		this.ddx = 0;
		this.ddy = 0;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// SplinePtTracker.save {joj 02/11/20}
	//
	SplinePtTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddx = this.ddx;
		this.savedState.ddy = this.ddy;
	}

	//
	// SplinePtTracker.restore {joj 02/11/20}
	//
	SplinePtTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddx = this.savedState.ddx;
				this.ddy = this.savedState.ddy;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// SplinePtTracker.action {joj 02/11/20}
	//
	SplinePtTracker.prototype.action = function() {
		this.step++;
		let nddx = this.step * this.dx / this.nsteps;
		let nddy = this.step * this.dy / this.nsteps;
		if (nddx || nddy) {
			this.g.firstUpdate();
			this.g.sptx[this.n] += nddx - this.ddx;
			this.g.spty[this.n] += nddy - this.ddy;
			this.g.convertToBezier;	// expensive
			this.g.update();
			this.ddx = nddx;
			this.ddy = nddy;
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// SplineTensionTracker {joj 03/11/20}
	//
	// NB: nsteps and wait will be defined
	// NB: uses ddTension to reduce the accumulation of errors
	//
	function SplineTensionTracker(g, dTension, nsteps, wait) {
		this.g = g;
		this.dTension = dTension;
		this.ddTension = 0;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// SplineTensionTracker.save {joj 03/11/20}
	//
	SplineTensionTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddTension = this.ddTension;
	}

	//
	// SplineTensionTracker.restore {joj 03/11/20}
	//
	SplineTensionTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddTension = this.savedState.ddTension;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// SplineTensionTracker.action {joj 03/11/20}
	//
	SplineTensionTracker.prototype.action = function() {
		this.step++;
		let nddTension = this.step * this.dTension / this.nsteps;
		if (nddTension) {
			this.g.firstUpdate();
			this.g.tension += nddTension - this.ddTension;
			this.g.convertToBezier;	// expensive
			this.g.update();
			this.ddTension = nddTension;
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// RotateTracker
	//
	// NB: nsteps and wait will be defined
	// NB: ddTheta reduces the accumulation of errors
	//
	function RotateTracker(g, dTheta, nsteps, wait) {
		this.g = g;
		this.dTheta = dTheta;
		this.ddTheta = 0;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// RotateTracker.save
	//
	RotateTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddTheta = this.ddTheta;
	}

	//
	// RotateTracker.restore
	//
	RotateTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddTheta = this.savedState.ddTheta;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// RotateTracker.action
	//
	RotateTracker.prototype.action = function() {
		this.step++;
		let nddTheta = this.step * this.dTheta / this.nsteps;
		if (nddTheta != this.ddTheta) {
			this.g.firstUpdate();
			this.g.angle += nddTheta - this.ddTheta;
			this.g.setTransform2D();
			this.g.update(UPDATEMEMBERS);
			this.ddTheta = nddTheta;
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// RoundedTracker
	//
	// NB: nsteps and wait will be defined
	//
	function RoundedTracker(g, rx, ry, nsteps, wait) {
		this.g = g;
		this.dRx = (rx - g.rx) / nsteps;
		this.dRy = (ry - g.ry) / nsteps;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// RoundedTracker.save
	//
	RoundedTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.dRx = this.dRx;
		this.savedState.dRy = this.dRy;
		this.savedState.step = this.step;
	}

	//
	// RoundedTracker.restore
	//
	RoundedTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.dRx = this.savedState.dRx;
				this.dRy = this.savedState.dRy;
				this.step = this.savedState.step;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// RoundedTracker.action
	//
	RoundedTracker.prototype.action = function() {
		this.step++;
		this.g.firstUpdate();
		this.g.rx += this.dRx;
		this.g.ry += this.dRy;
		this.g.update();
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// ScaleTracker
	//
	// NB: nsteps and wait will be defined
	// TODO: reduce accumulation of errors
	//
	function ScaleTracker(g, sx, sy, nsteps, wait) {
		this.g = g;
		this.sx = sx;
		this.sy = sy;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// ScaleTracker.save
	//
	ScaleTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
	}

	//
	// ScaleTracker.restore
	//
	ScaleTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// ScaleTracker.action
	//
	ScaleTracker.prototype.action = function() {
		this.step++;
		this.g.firstUpdate();
		this.g.sx *= Math.pow(this.sx, 1/this.nsteps);
		this.g.sy *= Math.pow(this.sy, 1/this.nsteps);
		this.g.setTransform2D();
		this.g.update(UPDATEMEMBERS);
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// TxtOffTracker
	//
	// NB: nsteps and wait will be defined
	//
	function TxtOffTracker(g, txtOffX, txtOffY, nsteps, wait) {
		this.g = g;
		this.dTxtOffX = (txtOffX - g.txtOffX) / nsteps;
		this.dTxtOffY = (txtOffY - g.txtOffY) / nsteps;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// TxtOffTracker.save
	//
	TxtOffTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.dTxtOffX = this.dTxtOffX;
		this.savedState.dTxtOffY = this.dTxtOffY;
		this.savedState.step = this.step;
	}

	//
	// TxtOffTracker.restore
	//
	TxtOffTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.dTxtOffX = this.savedState.dTxtOffX;
				this.dTxtOffY = this.savedState.dTxtOffY;
				this.step = this.savedState.step;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// TxtOffTracker.action
	//
	TxtOffTracker.prototype.action = function() {
		this.step++;
		this.g.firstUpdate();
		this.g.txtOffX += this.dTxtOffX;
		this.g.txtOffY += this.dTxtOffY;
		this.g.update();
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// TranslateTracker
	//
	// nticks and wait will be defined
	// ddx and ddy used to reduce the accumulation of errors
	// added RT tracking
	//
	// this.thread = $getCurrentThread gets a pointer to the current thread
	// the thread state is NOT up to date as the current state is in $pc, $sp, $fp etc.
	// when execute(toThread) is called, it checks if toThread is the current thread
	// if toThread is the current thread, nothing need to be done - the current values of $pc, $fp, $sp etc. are used
	// if toThread != thread, the current thread state is saved by $suspendThread and
	// toThread resumed by callin $resumeThread
	//
	function TranslateTracker(g, dx, dy, ticks, wait) {
		//console.log("new TranslateTracker tick=" + tick);
		this.g = g;
		this.dx = dx;
		this.dy = dy;
		this.ddx = 0;
		this.ddy = 0;
		this.ticks = ticks & ~RT;								// {joj 25/11/20}
		this.rt = ticks & RT;									// {joj 25/11/20}
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.tick = 0;
		this.savedState = 0;
		if (this.rt) {											// {joj 25/11/2}
			setTimeout(this.action.bind(this), 1000 / tps);		// {joj 25/11/20}
		} else {
			addToEventQ(new TrackerEvent(tick + 1, this));
		}
	}

	//
	// TranslateTracker.save
	//
	TranslateTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.tick = this.tick;
		this.savedState.ddx = this.ddx;
		this.savedState.ddy = this.ddy;
	}

	//
	// TranslateTracker.restore
	//
	TranslateTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.tick = this.savedState.tick;
				this.ddx = this.savedState.ddx;
				this.ddy = this.savedState.ddy;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// TranslateTracker.action
	//
	TranslateTracker.prototype.action = function() {
		this.tick++;
		let nddx = this.tick * this.dx / this.ticks;
		let nddy = this.tick * this.dy / this.ticks;
		if (nddx || nddy) {
			this.g.firstUpdate();
			this.g.tx += nddx - this.ddx;	// difference
			this.g.ty += nddy - this.ddy;	// difference
			this.g.setTransform2D();
			//debug("TranslateTracker.action txt=%s tick=%d ticks=%d tx=%d ty=%d", this.g.txt, this.tick, this.ticks, this.g.tx, this.g.ty);
			this.g.update(FIREMOVED | UPDATEMEMBERS);
			this.ddx = nddx;
			this.ddy = nddy;
			if (this.rt) {				// {joj 25/11/20}
				asyncPhaseUpdate = 1;	// {joj 25/11/20}
				drawChanges();			// {joj 25/11/20}
			}
		}
		if (this.tick < this.ticks) {
			//console.log("TranslateTracker.action addToEventQ txt=" + this.g.txt);
			if (this.rt) {										// {joj 25/11/20}
				setTimeout(this.action.bind(this), 1000 / tps);	// {joj 25/11/20}
			} else {
				addToEventQ(new TrackerEvent(tick + 1, this));
			}
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// TranslatePinTracker
	//
	// NB: nsteps and wait will be defined
	// NB: ddx and ddy reduce the accumulation of errors
	//
	function TranslatePinTracker(g, dx, dy, nsteps, wait) {
		//console.log("new TranslatePinTracker tick=" + tick);
		this.g = g;
		this.dx = dx;
		this.dy = dy;
		this.ddx = 0;
		this.ddy = 0;
		this.nsteps = nsteps;
		this.wait = wait;
		if (wait)
			this.thread = $getCurrentThread();
		this.step = 0;
		this.savedState = 0;
		addToEventQ(new TrackerEvent(tick + 1, this));
	}

	//
	// TranslatePinTracker.save
	//
	TranslatePinTracker.prototype.save = function() {
		this.savedState = new TrackerState(this.savedState);
		this.savedState.step = this.step;
		this.savedState.ddx = this.ddx;
		this.savedState.ddy = this.ddy;
	}

	//
	// TranslatePinTracker.restore
	//
	TranslatePinTracker.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.step = this.savedState.step;
				this.ddx = this.savedState.ddx;
				this.ddy = this.savedState.ddy;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// TranslatePinTracker.action
	//
	TranslatePinTracker.prototype.action = function() {
		this.step++;
		let nddx = this.step * this.dx / this.nsteps;
		let nddy = this.step * this.dy / this.nsteps;
		if (nddx || nddy) {
			this.g.firstUpdate();
			this.g.pinx += nddx - this.ddx;
			this.g.piny += nddy - this.ddy;
			this.g.setTransform2D();
			this.g.update(FIREMOVED | UPDATEMEMBERS);
			this.ddx = nddx;
			this.ddy = nddy;
		}
		if (this.step < this.nsteps) {
			addToEventQ(new TrackerEvent(tick + 1, this));
		} else if (this.wait) {
			$execute(this.thread);
		}
	}

	//
	// PBFState constructor
	//
	// used by Pen, Brush and Font
	//
	function PBFState (tick, next) {
		//this.tick = tick;	// asyncPhaseUpdate will be 0
		//this.next = next;	// linked list
		Object.defineProperty(this, "tick", {value:tick, writable:1});	// non enumerable {joj 16/10/20}
		Object.defineProperty(this, "next", {value:next, writable:1});	// non enumerable {joj 16/10/20}
	}

	//
	// PBF constructor {joj 15/10/20}
	//
	// Pen, Brush and Font are "derived from" PBF
	//
	function PBF() {
		Object.defineProperty(this, "savedState", {value:0, writable:1});							// non enumerable {joj 16/10/20}
		Object.defineProperty(this, "created", {value:2*tick + asyncPhaseUpdate, writable:1});		// non enumerable {joj 16/10/20}
		Object.defineProperty(this, "lastUpdate", {value:2*tick + asyncPhaseUpdate, writable:1});	// non enumerable {joj 16/10/20}
		Object.defineProperty(this, "gobjs", {value:0, writable:1});								// non enumerable {joj 16/10/20}
	}

	//
	// PBF.add (internal)
	//
	PBF.prototype.add = function(gobj) {
		if (this.gobjs)
			this.gobjs.push(gobj);
	}

	//
	// PBF.remove (internal)
	//
	PBF.prototype.remove = function(gobj) {
		if (this.gobjs)
			this.gobjs.splice(this.gobjs.indexOf(gobj), 1);		// gobjs updated in place
	}

	//
	// PBF.save (internal)
	//
	// save 2*tick + asyncPhas properties {joj 15/10/20}
	//
	PBF.prototype.save = function(tick) {
		if (this.savedState == 0 || this.lastUpdate > this.savedState.tick) {
			this.savedState = new PBFState(tick, this.savedState);
			for (let prop in this) {
				if (this.hasOwnProperty(prop)) {
					this.savedState[prop] = this[prop];
				}
			}
		}
	}

	//
	// PBF.restore (internal)
	//
	// call own restoreState function
	//
	PBF.prototype.restore = function(tick) {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				this.restoreState();
				this.gobjs = 0;
				return;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// PBF.firstUpdate (internal)
	//
	PBF.prototype.firstUpdate = function() {
		if (playZero == 0 && this.savedState == 0 && this.created == 0)
			this.save(0);
	}

	//
	// PBF.updateGObjs (internal)
	//
	PBF.prototype.updateGObjs = function(gobj) {
		if (this.match(gobj)) {
			gobj.firstUpdate();	// {joj 14/10/20}
			gobj.update();
			this.gobjs.push(gobj);
		}
		if (gobj.gobjs) {
			for (let i = 0; i < gobj.gobjs.length; i++)
				this.updateGObjs(gobj.gobjs[i]);
		}
	}

	//
	// PBF.update (internal)
	//
	PBF.prototype.update = function() {
		this.lastUpdate = 2*tick + asyncPhaseUpdate;
		if (this.gobjs) {
			for (let i = 0; i < this.gobjs.length; i++)
				this.gobjs[i].update();
		} else {
			this.gobjs = [];
			this.updateGObjs($g[0]);
		}
	}

	//
	// Pen constructor
	//
	// this description also applies to Brush and Font
	//
	// Pen, Brush and Font are derived from PBF.
	//
	// A Pen has a number of properties (eg colour and width). If a Pen property
	// is changed, all GObjs drawn with the Pen must be marked as updated
	// so they will be redrawn. The implementation is optimised on the
	// assumption that the Pen properties normally remain constant. Initially,
	// a Pen doesn't keep a list of gobjs that are drawn using it, but if a Pen
	// property is changed, this list of gobjs is generated dynamically by
	// scanning all GObjs. The gobjs list can then be used thereafter
	// to update the GObjs drawn with the Pen more efficiently. When
	// gobj.setPen is called, the gobj is removed from the the old Pen gobjs
	// list and added to the new Pen gobjs list if they exist.
	//
	// Pen changed is recorded in lastUpdate, so that it can be saved or otherwise
	// Pen is state only saved when first updated
	//
	// Pen() not called from Vivio
	//
	function Pen() {
		PBF.call(this);
		Object.defineProperty(this, "ctxStrokeStyle", {value:"", writable:1});	// non enumerable {joj 15/10/20}
		pens.push(this);
	}
	Pen.prototype = Object.create(PBF.prototype);

	//
	// Pen.setStrokeStyle {joj 14/10/20}
	//
	Pen.prototype.setStrokeStyle = function() {
		switch (this.type) {
		case NULLPEN:
			this.ctxStrokeStyle = "";
			break;
		case SOLIDPEN:
			this.ctxStrokeStyle = toRGBA(this.rgba);
			break;
		case IMAGEPEN:
			this.image = new window.Image();
			this.image.pen = this;
			this.image.onload = penPatternLoaded;
			this.image.src = this.url;
			break;
		}
	}

	//
	// NullPen constructor
	//
	function NullPen() {
		Pen.call(this);
		this.type = NULLPEN;
	}
	NullPen.prototype = Object.create(Pen.prototype);

	//
	// SolidPen constructor
	//
	function SolidPen(style = 0, width = 1, rgba = BLACK, caps = 0, scs = 2, ecs = 2) {
		Pen.call(this);
		this.type = SOLIDPEN;
		this.style = style > MAX_STYLE ? 0 : style;
		this.width = width;
		this.rgba = rgba;
		this.caps = caps;
		this.scs = scs;
		this.ecs = ecs;
		this.setStrokeStyle();
	}
	SolidPen.prototype = Object.create(Pen.prototype);

	//
	// penPatternLoaded
	//
	// handles the asychronous loading of brush patterns
	//
	function penPatternLoaded() {
		//console.log("penPatternLoaded loaded");
		if (this.pen.type == IMAGEPEN) {	//  make sure brush type hasn't changed before image loaded
			this.pen.strokeStyle = ctx.createPattern(this, "repeat");
			this.pen.update();		// update objects drawn with brush
			asyncPhaseUpdate = 1;
			drawChanges();			// clears asyncPhaseUpdate {joj 28/11/17}
		}
	}

	//
	// ImagePen constructor
	//
	function ImagePen(options, style, width, url, caps = 0, scs = 2, ecs = 2) {
		Pen.apply(this);
		this.type = IMAGEPEN;
		this.options = options;
		this.style = style;
		this.width = width;
		this.rgba = 0;
		this.url = url;
		this.caps = caps;
		this.scs = scs; 	// startCapScale
		this.ecs = ecs;		// endCapScale
		this.setStrokeStyle();
	}
	ImagePen.prototype = Object.create(Pen.prototype);

	//
	// Pen.setNull
	//
	Pen.prototype.setNull = function() {
		if (this.type != NULLPEN) {
			this.firstUpdate();
			this.type = NULLPEN;
			delete this.options;	// delete {joj 15/10/20}
			delete this.style;		// delete {joj 15/10/20}
			delete this.width;		// delete {joj 15/10/20}
			delete this.rgba;		// delete {joj 15/10/20}
			delete this.url;		// delete {joj 15/10/20}
			delete this.caps;		// delete {joj 15/10/20}
			delete this.scs;		// delete {joj 15/10/20}
			delete this.ecs;		// delete {joj 15/10/20}
			this.setStrokeStyle();
			this.update();
		}
		return this;
	}

	//
	// Pen.setSolid
	//
	Pen.prototype.setSolid = function(style = 0, width = 1, rgba = 0, caps = 0, scs = 2, ecs = 2) {
		style = style > MAX_STYLE ? 0 : style;
		if ((this.type != SOLIDPEN) || (this.style != style) || (this.width != width) || (this.rgba != rgba) || (this.caps != caps) || (this.scs != scs) || (this.ecs != ecs)) {
			this.firstUpdate();
			this.type = SOLIDPEN;
			delete this.options;	// delete {joj 15/10/20}
			this.style = style;
			this.width = width;
			this.rgba = rgba;
			delete this.url;		// delete {joj 15/10/20}
			this.caps = caps;
			this.scs = scs;
			this.ecs = ecs;
			this.setStrokeStyle();
			this.update();
		}
		return this;
	}

	//
	// Pen.setImage
	//
	Pen.prototype.setImage = function(options, style, width, url, caps = 0, scs = 2, ecs = 2) {
		if ((this.type != IMAGEPEN) || (this.options != options) || (this.style != style) || (this.width != width) || (this.url != url) || (this.caps != caps) || (this.scs != scs) || (this.ecs != ecs)) {
			this.firstUpdate();
			this.type = IMAGEPEN;
			this.options = options;
			this.style = style;
			this.width = width;
			this.rgba = 0;
			this.url = url;
			this.caps = caps;
			this.scs = scs;
			this.ecs = ecs;
			this.setStrokeStyle();
			this.update();
		}
		return this;
	}

	//
	// Pen.restoreState (internal)
	//
	Pen.prototype.restoreState = function() {
		for (let prop in this.savedState)		// {joj 15/10/20}
			this[prop] = this.savedState[prop];	// {joj 15/10/20}
		this.setStrokeStyle();
	}

	//
	// Pen.getType
	//
	Pen.prototype.getType = function() { return this.type }

	//
	// Pen.getOptions
	//
	Pen.prototype.getOptions = function() { return this.options || 0}

	//
	// Pen.getStyle
	//
	Pen.prototype.getStyle = function() { return this.style || 0}

	//
	// Pen.getWidth
	//
	Pen.prototype.getWidth = function() { return this.width || 0}

	//
	// Pen.getRGBA
	//
	Pen.prototype.getRGBA = function() { return this.rgba || 0}

	//
	// Pen.getURL
	//
	Pen.prototype.getURL = function() { return this.url || ""}

	//
	// Pen.getCaps
	//
	Pen.prototype.getCaps = function() { return this.caps || 0}

	//
	// Pen.getStartCapScale
	//
	Pen.prototype.getStartCapScale = function() { return this.scs || 0}

	//
	// Pen.getEndCapScale
	//
	Pen.prototype.getEndCapScale = function() { return this.ecs || 0}

	//
	// Pen.match helper (internal)
	//
	Pen.prototype.match = function(gobj) { return gobj.pen == this }

	//
	// Pen.setWidth
	//
	Pen.prototype.setWidth = function(width, nsteps = 0, wait = 0) {
		if ((this.width == width) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.width = width;
			this.update();
			return 0;
		}
		new PenWidthTracker(this, width, nsteps, wait);
		return wait;
	}

	//
	// Pen.setRGBA
	//
	Pen.prototype.setRGBA = function(rgba, nsteps = 0, wait = 0) {
		if (this.type != SOLIDPEN)
			return 0;
		if ((this.rgba == rgba) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.rgba = rgba;
			this.setStrokeStyle();
			this.update();
			return 0;
		}
		new PenRGBATracker(this, rgba, nsteps, wait);
		return wait;
	}

	var defaultPen = new SolidPen();

	//
	// Brush constructor
	//
	// never called by vivio
	//
	function Brush() {
		PBF.call(this);
		Object.defineProperty(this, "ctxFillStyle", {value:"", writable:1});	// non enumerable {joj 15/10/20}
		brushes.push(this);
	}
	Brush.prototype = Object.create(PBF.prototype);

	//
	// Brush.setFillStyle {joj 13/10/20}
	//
	Brush.prototype.setFillStyle = function() {
		switch (this.type) {
		case NULLBRUSH:
			this.ctxFillStyle = "";
			break;
		case SOLIDBRUSH:
			this.ctxFillStyle = toRGBA(this.rgba0);
			break;
		case GRADIENTBRUSH:
			this.ctxFillStyle = ctx.createLinearGradient(this.x0, this.y0, this.x1, this.y1);
			this.ctxFillStyle.addColorStop(0, toRGBA(this.rgba0));
			this.ctxFillStyle.addColorStop(1, toRGBA(this.rgba1));
			break;
		case RADIALBRUSH:
			this.ctxFillStyle = ctx.createRadialGradient(this.x0, this.y0, this.r0, this.x1, this.y1, this.r1);
			this.ctxFillStyle.addColorStop(0, toRGBA(this.rgba0));
			this.ctxFillStyle.addColorStop(1, toRGBA(this.rgba1));
			break;
		case IMAGEBRUSH:
			this.image = new window.Image();
			this.image.patternBrush = this;
			this.image.onload = brushPatternLoaded;
			this.image.src = this.url;
			break;
		}
	}

	//
	// NullBrush constructor
	//
	function NullBrush() {
		Brush.call(this);
		this.type = NULLBRUSH;
	}
	NullBrush.prototype = Object.create(Brush.prototype);

	//
	// SolidBrush constructor
	//
	function SolidBrush(rgba) {
		Brush.call(this);
		this.type = SOLIDBRUSH;
		this.rgba0 = rgba || 0;
		this.setFillStyle();
	}
	SolidBrush.prototype = Object.create(Brush.prototype);

	//
	// ImageBrush constructor
	//
	function ImageBrush(options, url) {
		Brush.call(this);
		this.type = IMAGEBRUSH;
		this.options = options;
		this.url = url;
		this.setFillStyle();
	}
	ImageBrush.prototype = Object.create(Brush.prototype);

	//
	// GradientBrush constructor
	//
	function GradientBrush(options, x0, y0, x1, y1, rgba0, rgba1) {
		Brush.call(this);
		this.type = GRADIENTBRUSH;
		this.options = options;
		this.x0 = x0;
		this.y0 = y0;
		this.x1 = x1;
		this.y1 = y1;
		this.rgba0 = rgba0;
		this.rgba1 = rgba1;
		this.setFillStyle();
	}
	GradientBrush.prototype = Object.create(Brush.prototype);

	//
	// RadialBrush constructor
	//
	function RadialBrush(options, x0, y0, r0, x1, y1, r1, rgba0, rgba1) {
		Brush.call(this);
		this.type = RADIALBRUSH;
		this.options = options;
		this.x0 = x0;
		this.y0 = y0;
		this.r0 = r0;
		this.x1 = x1;
		this.y1 = y1;
		this.r1 = r1;
		this.rgba0 = rgba0;
		this.rgba1 = rgba1;
		this.setFillStyle();
	}
	RadialBrush.prototype = Object.create(Brush.prototype);

	//
	// Brush.setNull
	//
	Brush.prototype.setNull = function() {
		if (this.type != NULLBRUSH) {
			this.firstUpdate();
			this.type = NULLBRUSH;
			delete this.options;	// delete {joj 15/10/20}
			delete this.rgba0;		// delete {joj 15/10/20}
			delete this.rgba1;		// delete {joj 15/10/20}
			delete this.url;		// delete {joj 15/10/20}
			delete this.x0;			// delete {joj 15/10/20}
			delete this.y0;			// delete {joj 15/10/20}
			delete this.r0;			// delete {joj 15/10/20}
			delete this.x1;			// delete {joj 15/10/20}
			delete this.y1;			// delete {joj 15/10/20}
			delete this.r1;			// delete {joj 15/10/20}
			this.setFillStyle();
			this.update();
		}
		return this
	}

	//
	// Brush.setSolid
	//
	Brush.prototype.setSolid = function(rgba) {
		if ((this.type != SOLIDBRUSH) || (this.rgba0 != rgba)) {
			this.firstUpdate();
			this.type = SOLIDBRUSH;
			delete this.options;	// delete {joj 15/10/20}
			this.rgba0 = rgba;
			delete this.rgba1;		// delete {joj 15/10/20}
			delete this.url;		// delete {joj 15/10/20}
			delete this.x0;			// delete {joj 15/10/20}
			delete this.y0;			// delete {joj 15/10/20}
			delete this.r0;			// delete {joj 15/10/20}
			delete this.x1;			// delete {joj 15/10/20}
			delete this.y1;			// delete {joj 15/10/20}
			delete this.r1;			// delete {joj 15/10/20}
			this.setFillStyle();
			this.update();
		}
		return this
	}

	//
	// Brush.setImage
	//
	Brush.prototype.setImage = function(options, url) {
		if ((this.type != IMAGEBRUSH) || (this.options != options) || (this.url != url)) {
			this.firstUpdate();
			this.type = IMAGEBRUSH;
			this.options = options;
			delete this.rgba0;		// delete {joj 15/10/20}
			delete this.rgba1;		// delete {joj 15/10/20}
			this.url = url;
			delete this.x0;			// delete {joj 15/10/20}
			delete this.y0;			// delete {joj 15/10/20}
			delete this.r0;			// delete {joj 15/10/20}
			delete this.x1;			// delete {joj 15/10/20}
			delete this.y1;			// delete {joj 15/10/20}
			delete this.r1;			// delete {joj 15/10/20}
			this.setFillStyle();
		}
		return this
	}

	//
	// Brush.setGradient
	//
	Brush.prototype.setGradient = function(options, x0, y0, x1, y1, rgba0, rgba1) {
		if ((this.type != GRADIENTBRUSH) || (this.options != options) || (this.x0 != x0) || (this.y0 != y0) || (this.x1 != x1) || (this.y1 != y1) || (this.rgba0 != rgba0) || (this.rgba1 != rgba1)) {
			this.firstUpdate();
			this.type = GRADIENTBRUSH;
			this.options = options;
			delete this.url;	// delete {joj 15/10/20}
			this.x0 = x0;
			this.y0 = y0;
			delete this.r0;		// delete {joj 15/10/20}
			this.x1 = x1;
			this.y1 = y1;
			delete this.r1;		// delete {joj 15/10/20}
			this.rgba0 = rgba0;
			this.rgba1 = rgba1;
			this.setFillStyle();
			this.update();
		}
		return this
	}

	//
	// Brush.setRadial
	//
	Brush.prototype.setRadial = function(options, x0, y0, r0, x1, y1, r1, rgba0, rgba1) {
		if ((this.type != GRADIENTBRUSH) || (this.options != options) || (this.x0 != x0) || (this.y0 != y0) || (this.r0 != r0) || (this.x1 != x1)|| (this.y1 != y1) || (this.r1 != r1) || (this.rgba0 != rgba0) || (this.rgba1 != rgba1)) {
			this.firstUpdate();
			this.type = RADIALBRUSH;
			this.options = options;
			delete this.url;	// delete {joj 15/10/20}
			this.x0 = x0;
			this.y0 = y0;
			this.r0 = r0;
			this.x1 = x1;
			this.y1 = y1;
			this.r1 = r1;
			this.rgba0 = rgba0;
			this.rgba1 = rgba1;
			this.setFillStyle();
			this.update();
		}
		return this
	}

	//
	// Brush.getType
	//
	Brush.prototype.getType = function() { return this.type }

	//
	// Brush.getRGBA
	//
	Brush.prototype.getRGBA = function() { return this.rgba0 || 0}

	//
	// Brush.getOptions
	//
	Brush.prototype.getOptions = function() { return this.options || 0}

	//
	// Brush.getURL
	//
	Brush.prototype.getURL = function() { return this.url || "" }

	//
	// Brush.getX0
	//
	Brush.prototype.getX0 = function() { return this.x0 || 0}

	//
	// Brush.getY0
	//
	Brush.prototype.getY0 = function() { return this.y0 || 0}

	//
	// Brush.getR0
	//
	Brush.prototype.getR0 = function() { return this.r0 || 0}

	//
	// Brush.getX1
	//
	Brush.prototype.getX1 = function() { return this.x1 || 0}

	//
	// Brush.getY1
	//
	Brush.prototype.getY1 = function() { return this.y1 || 0}

	//
	// Brush.getR1
	//
	Brush.prototype.getR1 = function() { return this.r1 || 0}

	//
	// Brush.getRGBA0
	//
	Brush.prototype.getRGBA0 = function() { return this.rgba0 || 0}

	//
	// Brush.getRGBA1
	//
	Brush.prototype.getRGBA1 = function() { return this.rgba1 || 0}

	//
	// Brush.match helper (internal)
	//
	Brush.prototype.match = function(gobj) { return gobj.brush === this }

	//
	// Brush.restoreState (internal)
	//
	Brush.prototype.restoreState = function() {
		for (let prop in this.savedState) 		// {joj 15/10/20}
			this[prop] = this.savedState[prop];	// {joj 15/10/20}
		this.setFillStyle();
	}

	//
	// Brush.setRGBA
	//
	Brush.prototype.setRGBA = function(rgba, nsteps, wait) {
		return this.setRGBA2(rgba, undefined, nsteps, wait);
	}

	//
	// Brush.setRGBA2
	//
	Brush.prototype.setRGBA2 = function(rgba0, rgba1, nsteps = 0, wait = 0) {
		if ((this.type != SOLIDBRUSH) && (this.type != GRADIENTBRUSH) && (this.type != RADIALBRUSH))
			return 0;
		if ((this.type == SOLIDBRUSH) && (this.rgba0 == rgba))
			return 0;
		if (((this.type == GRADIENTBRUSH) || (this.type == RADIALBRUSH)) && (this.rgba0 == rgba0) && (this.rgba1 == rgba1))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.rgba0 = rgba0;
			this.rgba1 = rgba1
			this.setFillStyle();
			this.update();
			return 0;
		}
		new RGBATracker(this, rgba0, rgba1, nsteps, wait);
		return wait;
	}

	//
	// brushPatternLoaded
	//
	// handles the asychronous loading of brush patterns
	// set asyncPhaseUpdate so GObjs using brush are redrawn asynchronously
	//
	function brushPatternLoaded() {
		//console.log("brushPatternLoaded loaded");
		if (this.patternBrush.type == IMAGEBRUSH) {	//  make sure brush type hasn't changed before image loaded
			this.patternBrush.fillStyle = ctx.createPattern(this, "repeat");
			this.patternBrush.update();	// update objects drawn with brush
			asyncPhaseUpdate = 1;
			drawChanges();				// clears asyncPhaseUpdate {joj 28/11/17}
		}
	}

	//
	// Font constructor
	//
	function Font(face, sz, flags) {
		PBF.call(this);
		this.face = face;
		this.sz = sz;
		this.flags = flags || 0;
		Object.defineProperty(this, "ctxFont", {writable:1});	// non enumerable {joj 15/10/20}
		this.setCtxFont();
		fonts.push(this);
	}
	Font.prototype = Object.create(PBF.prototype);

	//
	// Font.setCtxFont (internal)
	//
	// canvas fonts do not support UNDERLINE and STRIKETHROUGH (handled in drawTxt)
	//
	Font.prototype.setCtxFont = function() {
		this.ctxFont = "";
		if (this.flags & BOLD)
			this.ctxFont += "bold";
		if (this.flags & ITALIC)
			this.ctxFont += this.ctxFont.length ? " italic" : "italic";
		if (this.flags & SMALLCAPS)
			this.ctxFont += this.ctxFont.length ? " small-caps" : "small-caps";
		this.ctxFont += (this.ctxFont.length ? " " : "") + this.sz + "px " + this.face;
		//console.log("font.setCtxFont() returning " + this.ctxFont);
	}

	//
	// Font.getFace
	//
	Font.prototype.getFace = function() { return this.face }

	//
	// Font.getSz
	//
	Font.prototype.getSz = function() { return this.sz }

	//
	// Font.getFlags
	//
	Font.prototype.getFlags = function() { return this.flags }

	//
	// Font.match (internal)
	//
	Font.prototype.match = function(gobj) {
		return gobj.font == this;
	}

	//
	// Font.restoreState (internal)
	//
	Font.prototype.restoreState = function() {
		for (let prop in this.savedState)		// {joj 15/10/20}
			this[prop] = this.savedState[prop];	// {joj 15/10/20}
		this.setCtxFont();
	}

	//
	// Font.setFont
	//
	Font.prototype.setFont = function(face, sz, flags) {
		flags = flags || 0;
		if ((this.face != face) || (this.sz != sz) ||(this.flags != flags)) {
			this.firstUpdate();
			this.face = face;
			this.sz = sz;
			this.flags = flags;
			this.setCtxFont();
			this.update();
		}
		return this;
	}

	//
	// Font.setSz
	//
	Font.prototype.setSz = function(sz, nsteps = 0, wait = 0) {
		if ((this.sz == sz) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.sz = sz;
			this.setCtxFont();
			this.update();
			return 0;
		}
		new FontSzTracker(this, sz - this.sz, nsteps, wait);
		return wait;
	}

	var defaultFont = new Font("sans-serif", 24);		// default font {joj 09/05/21}
	var defaultMenuFont = new Font("sans-serif", 18);	// default menu font {joj 09/05/21}

	//
	// GObjState constructor
	//
	function GObjState(tick, next) {
		Object.defineProperty(this, "tick", {value:tick, writable:1});	// non enumerable {joj 16/10/20}
		Object.defineProperty(this, "next", {value:next, writable:1});	// non enumerable {joj 16/10/20}
	}

	//
	// GObj constructor {joj 16/10/20}
	//
	function GObj(grp, _layer, options, x, y) {
		this.grp = grp;
		Object.defineProperty(this, "layer", {value:(_layer ? _layer : layer[0]), writable:1});
		this.options = options;
		Object.defineProperty(this, "transform2D", {value: (new T2D()), writable:1});
		Object.defineProperty(this, "mbb0", {value: (new Mbb()), writable:1});
		Object.defineProperty(this, "savedState", {value: 0, writable:1});
		Object.defineProperty(this, "clipped", {value: 0, writable:1});
		Object.defineProperty(this, "enter", {value: 0, writable:1});
		Object.defineProperty(this, "created", {value: 2*tick + asyncPhaseUpdate, writable:1});;
		Object.defineProperty(this, "updFlags", {value: UPDATE, writable:1});
		Object.defineProperty(this, "lastUpdateDraw", {value: -1, writable:1});
		Object.defineProperty(this, "drawOpacity", {value: 1, writable:1});
		Object.defineProperty(this, "mbb", {value: (new Mbb()), writable:1});
		Object.defineProperty(this, "drawT2D", {value: (new T2D()), writable:1});
		this.tx = x;
		this.ty = y;
		this.sx = 1;
		this.sy = 1;
		this.pinx = 0;
		this.piny = 0;
		this.angle = 0;
		this.txtOffX = 0;
		this.txtOffY = 0;
		this.textureOffX = 0;
		this.textureOffY = 0;
		this.txtPen = defaultPen;
		this.font = defaultFont;
		this.txt = "";
		this.clipPath = 0;
		this.opacity = 1;
		this.handler = [];
		this.lastUpdate = 2*tick + asyncPhaseUpdate;
		this.setTransform2D();
		if (this.grp && ((options & NOATTACH) == 0))
			this.grp.add(this);	// add to group
		this.layer.add(this);	// add to layer
	}

	//
	// GObj.addEventHandler
	//
	GObj.prototype.addEventHandler = function(e, obj, pc) {
		//console.log("GObj.addEventHandler("  + e + ", " + obj + ", " + pc + ")");
		if (this.handler[e] == undefined)
			this.handler[e] = [];
		this.handler[e].push({"obj":obj, "pc":pc});
	}

	//
	// GObj.attachTo
	//
	GObj.prototype.attachTo = function(grp) {
		console.log("GObj.attachTo()");
		grp.add(this);	// TODO remove from group, if in one,  first
	}

	//
	// GObj.calculateMbb0 {joj 25/09/20}
	//
	GObj.prototype.calculateMbb0 = function() {
		let minx = Number.MAX_VALUE;
		let miny = Number.MAX_VALUE;
		let maxx = Number.MIN_VALUE;
		let maxy = Number.MIN_VALUE;
		for (let i = 0; i < this.ptx.length; i++) {
			if (this.ptx[i] < minx)
				minx = this.ptx[i];
			if (this.pty[i] < miny)
				miny = this.pty[i];
			if (this.ptx[i] > maxx)
				maxx = this.ptx[i];
			if (this.pty[i] > maxy)
				maxy = this.pty[i];
		}
		this.mbb0.set(minx, miny, maxx, maxy);
	}

	//
	// GObj.contains {joj 07/10/20}
	//
	GObj.prototype.contains = function(x, y) {
		//console.log("GObj.contains x=" + x + " y=" + y + " hit=" + this.hit(x, y));
		return this.hit(x, y);
	}

	//
	// GObj.drawTxt helper
	//
	// code accounts for the different vertical positioning of text by different browsers
	// this.drawOpacity > 0
	// handle UNDERLINE and STRIKETHROUGH as not supported by canvas fonts
	//
	GObj.prototype.drawTxt = function(ctx) {
		if (this.txt.length && this.txtPen && this.font) {
			let maxw = 0, x, y;
			let hoptions = this.options & HMASK;
			let voptions = this.options & VMASK;
			let txt = this.txt.split("\n");
			let len = Array();
			ctx.font = this.font.ctxFont;
			//console.log("ctx.font=" + ctx.font + " this.font.ctxFont=" + this.font.ctxFont);
			for (let i = 0; i < txt.length; i++) {
				len[i] = ctx.measureText(txt[i]).width + 1;
				maxw = Math.max(len[i], maxw);
			}
			let h = this.font.sz * txt.length;
			ctx.fillStyle = toRGBA(this.txtPen.rgba);
			if (hoptions == HLEFT) {
				ctx.textAlign = "left";
				x = this.mbb0.x0;
			} else if (hoptions == HRIGHT) {
				ctx.textAlign = "right";
				x = this.mbb0.x1;
			} else {
				ctx.textAlign = "center"
				x = (this.mbb0.x0 + this.mbb0.x1)/2;
			}
			ctx.textBaseline = "middle";	// browers agree on middle
			if (voptions == VTOP) {
				y = this.mbb0.y0;
			} else if (voptions == VBOTTOM) {
				y = this.mbb0.y1 - h;
			} else {
				y = (this.mbb0.y0 + this.mbb0.y1)/2 - h/2;
			}
			if (this.txtOffX)
				x += this.txtOffX;
			if (this.txtOffY)
				y += this.txtOffY;
			for (let i = 0; i < txt.length; i++, y += this.font.sz) {
				ctx.fillText(txt[i], x, y + this.font.sz/2);
				if (this.font.flags & (UNDERLINE | STRIKETHROUGH)) {
					let xx = x;
					if (hoptions == HCENTRE) {
						xx -= len[i] / 2;
					} else if (hoptions == HRIGHT) {
						xx -= len[i];
					}
					ctx.save();
					ctx.lineWidth = 0;
					ctx.beginPath();
					if (this.font.flags & UNDERLINE) {
						let yy = y + this.font.sz;
						ctx.moveTo(xx, yy);
						ctx.lineTo(xx + len[i], yy);
					}
					if (this.font.flags & STRIKETHROUGH) {
						let yy = y + this.font.sz/2;
						ctx.moveTo(xx, yy);
						ctx.lineTo(xx + len[i], yy);
					}
					ctx.stroke();
					ctx.restore();
				}
			}
		}
	}

	//
	// GObj.firstUpdate
	//
	GObj.prototype.firstUpdate = function() {
		if (playZero == 0 && this.savedState == 0 && this.created == 0)
			this.save(0);
	}

	//
	// GObj.toggleBrushAndPens
	//
	// cahnged name and added txtPen1 parameter {joj 20/11/20}
	//
	GObj.prototype.toggleBrushAndPens = function(brush1, pen1, txtPen1, n, ticks, wait = 0) {
		if (n <= 0 || ticks <= 0)
			return 0;
		new ToggleTracker(this, brush1, pen1, txtPen1, n, ticks, wait);
		return wait;
	}

	//
	// GObj.getBrush
	//
	GObj.prototype.getBrush = function() { return this.brush }

	//
	// GObj.getClipPath
	//
	GObj.prototype.getClipPath = function() { return this.clipPath }

	//
	// GObj.getFont
	//
	GObj.prototype.getFont = function() { return this.font }

	//
	// GObj.getLayer
	//
	GObj.prototype.getLayer = function() { return this.layer }

	//
	// GObj.getMbbHelper
	//
	GObj.prototype.getMbbHelper = function(level) {
		if (level < 0)
			level = 0;
		//this.calculateMbb0();
		let mbbx = new Mbb(this.mbb0.x0, this.mbb0.y0, this.mbb0.x1, this.mbb0.y1);
		if (level == 0)
			return mbbx
		let gObj = this;
		for (let l = 0; l < level; l++) {
			gObj.setTransform2D();
			gObj.transform2D.transformMbb(mbbx);
			gObj = gObj.grp;
		}
		//console.log("getMbbHelper(" + level + ") x0=" + mbbx.x0 + " y0=" + mbbx.y0 + " x1=" + mbbx.x1 + " y1=" + mbbx.y1);
		return mbbx;
	}

	//
	// GObj.getMbb {joj 12/10/20}
	//
	GObj.prototype.getMbb = function(level) {
		return this.getMbbHelper(level);
	}

	//
	// GObj.getOpacity {joj 12/10/20}
	//
	GObj.prototype.getOpacity = function() { return this.opacity }

	//
	// GObj.getOptions
	//
	GObj.prototype.getOptions = function() { return this.options }

	//
	// GObj.getPen {joj 12/10/20}
	//
	GObj.prototype.getPen = function() { return this.pen }

	//
	// GObj.getPinX
	//
	GObj.prototype.getPinX = function() { return this.pinx }

	//
	// GObj.getPinY
	//
	GObj.prototype.getPinY = function() { return this.piny }

	//
	// GObj.getPosX
	//
	GObj.prototype.getPosX = function(level) {
		if (level < 0)
			level = 0;
		if (level == 0)
			return 0
		let x = 0, y = 0;
		let gObj = this;
		for (let l = 0; l < level; l++) {
			gObj.setTransform2D();
			x = x*gObj.transform2D.a + y*gObj.transform2D.c + gObj.transform2D.e;
			y = x*gObj.transform2D.b + y*gObj.transform2D.d + gObj.transform2D.f;
			gObj = gObj.grp;
		}
		return x;
	}

	//
	// GObj.getPosY
	//
	GObj.prototype.getPosY = function(level) {
		if (level < 0)
			level = 0;
		if (level == 0)
			return 0
		let x = 0, y = 0;
		let gObj = this;
		for (let l = 0; l < level; l++) {
			gObj.setTransform2D();
			x = x*gObj.transform2D.a + y*gObj.transform2D.c + gObj.transform2D.e;
			y = x*gObj.transform2D.b + y*gObj.transform2D.d + gObj.transform2D.f;
			gObj = gObj.grp;
		}
		return y;
	}

	//
	// GObj.getAngle
	//
	GObj.prototype.getAngle = function() { return this.angle }

	//
	// GObj.getTextureOffX
	//
	GObj.prototype.getTextureOffX = function() {
		return this.textureOffX
	}

	//
	// GObj.getTextureOffY
	//
	GObj.prototype.getTextureOffY = function() { return this.textureOffY }

	//
	// GObj.getTxt
	//
	GObj.prototype.getTxt = function() { return this.txt }

	//
	// GObj.getTxtOffX
	//
	GObj.prototype.getTxtOffX = function() { return this.txtOffX }

	//
	// GObj.getTxtOffY
	//
	GObj.prototype.getTxtOffY = function() { return this.txtOffY }

	//
	// GObj.getTxtPen
	//
	GObj.prototype.getTxtPen = function() { return this.txtPen }

	//
	// GObj.grab
	//
	GObj.prototype.grab = function() {
		//console.log("GRAB");
		grab = this;
	}

	//
	// GObj.moveAfter
	//
	GObj.prototype.moveAfter = function(gobj) {
		console.log("moveAfter");
	}

	//
	// GObj.moveToBack
	//
	// reorder layer gobjs, NO need to reorder grp gobjs
	//
	GObj.prototype.moveToBack = function() {
		this.layer.moveToBack(this);			// reorder layer gobjs
		this.update();							// make sure gobj redrawn
	}

	//
	// GObj.moveBefore
	//
	GObj.prototype.moveBefore = function(gobj) {
		console.log("moveBefore");
	}

	//
	// GObj.moveToFront
	//
	// reorder layer gobjs, NO need to reorder grp gobjs
	//
	GObj.prototype.moveToFront = function() {
		this.layer.moveToFront(this);			// reorder layer gobjs
		this.update();							// make sure gobj redrawn
	}

	//
	// GObj.restore {joj 16/10/20}
	//
	// this.savedState.ptx and this.savedState.pty non enumerable - restore only if they exist
	// this.savedState.length non enumerable - restore only if exists
	//
	GObj.prototype.restore = function() {
		while (this.savedState) {
			if (this.savedState.tick <= tick) {
				for (let prop in this.savedState)
					this[prop] = this.savedState[prop];
				if (this.savedState.ptx) {
					this.ptx.length = this.pty.length = 0;
					for (let i = 0; i < this.savedState.ptx.length; i++) {
						this.ptx[i] = this.savedState.ptx[i]
						this.pty[i] = this.savedState.pty[i];
					}
					this.calculateMbb0();
				}
				this.setTransform2D();
				if (this.savedState.length) {						// {joj 16/11/20}
					this.gobjs.length = this.savedState.length;		// {joj 16/11/20} GroupObj only
				}
				this.updFlags = UPDATE;				// {joj 14/10/20}
				break;
			}
			this.savedState = this.savedState.next;
		}
	}

	//
	// GObj.remove
	//
	GObj.prototype.remove = function() {
		console.log("remove");
	}

	//
	// GObj.rotate
	//
	GObj.prototype.rotate = function(dtheta, nsteps = 0, wait = 0) {
		if ((dtheta == 0) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.angle += dtheta;
			this.setTransform2D();
			this.update(UPDATEMEMBERS);
			return 0;
		}
		new RotateTracker(this, dtheta, nsteps, wait);
		return wait;
	}

	//
	// GObj.save {joj 16/10/20}
	//
	// NOT be called when tick = 0
	// normally saveTick = tick unless on firstUpdate when saveTick = 0
	// this.ptx and this.pty non enumerable - save only if they exist
	// this.gobjs non enumerable - save this.gobjs.length only if this.gobjs exists
	//
	GObj.prototype.save = function(saveTick) {
		if (saveTick == 0 || this.options & EXTENDEDGOBJ || (this.lastUpdate > 0 && (this.savedState == 0 || this.lastUpdate > 2*this.savedState.tick))) {	// {joj 18/10/20}

			this.savedState = new GObjState(saveTick, this.savedState);
			for (let prop in this) {
				if (this.hasOwnProperty(prop))
					this.savedState[prop] = this[prop];
			}

			if (this.ptx) {
				Object.defineProperty(this.savedState, "ptx", {value:[], writable:1});	// non enumerable {joj 16/10/20}
				Object.defineProperty(this.savedState, "pty", {value:[], writable:1});	// non enumerable {joj 16/10/20}
				for (let i = 0; i < this.ptx.length; i++) {
					this.savedState.ptx[i] = this.ptx[i]
					this.savedState.pty[i] = this.pty[i];
				}
			}

			if (this.gobjs) {
				Object.defineProperty(this.savedState, "length", {value:this.gobjs.length, writable:1});	// non enumerable {joj 16/11/20}
			}

		}
	}

	//
	// GObj.scale
	//
	GObj.prototype.scale = function(sx, sy, nsteps = 0, wait = 0) {
		if ((sx == 0) && (sy == 0) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.sx *= sx;
			this.sy *= sy;
			this.setTransform2D();
			this.update(UPDATEMEMBERS);
			return 0;
		}
		new ScaleTracker(this, sx, sy, nsteps, wait);
		return wait;
	}

	//
	// GObj.setAngle
	//
	GObj.prototype.setAngle = function(theta, nsteps, wait) {
		return this.rotate(theta - this.angle, nsteps, wait);
	}

	//
	// GObj.setBrush
	//
	GObj.prototype.setBrush = function(brush) {
		let oldBrush = this.brush;
		if (this.brush != brush) {
			this.firstUpdate();
			if (this.brush)
				this.brush.remove(this);
			this.brush = brush;
			if (this.brush)
				this.brush.add(this);
			this.update();
		}
		return oldBrush
	}

	//
	// GObj.setClipPath
	//
	GObj.prototype.setClipPath = function(clipPath) {
		//console.log("setClipPath");
		let oldClipPath = this.clipPath;
		if (this.clipPath != clipPath) {
			this.firstUpdate();
			this.clipPath = clipPath;
			this.update(UPDATEMEMBERS);
		}
		return oldClipPath;
	}

	//
	// GObj.setFont
	//
	GObj.prototype.setFont = function(font) {
		let oldFont = this.font;
		if (font == 0)	// {joj 16/09/20}
			font = defaultFont;
		if (this.font != font) {
			this.firstUpdate();
			if (this.font)
				this.font.remove(this);
			this.font = font;
			if (this.font)
				this.font.add(this);
			this.update();
		}
		return oldFont;
	}

	//
	// GObj.setMapping {joj 20/11/20}
	//
	GObj.prototype.setMapping = function() {
		// TODO
	}

	//
	// GObj.setLineStyle helper
	//
	// NB: decided not to scale dash and dots with pen width {joj 08/09/17}
	//
	GObj.prototype.setLineStyle = function(ctx, pen) {
		ctx.lineWidth = pen.width;
		ctx.lineCap = "butt";
		let caps = pen.caps & 0x00ff00;
		ctx.lineJoin = (caps == BEVEL_JOIN) ? "bevel" : (caps == ROUND_JOIN) ? "round" : "miter";
		ctx.fillStyle = pen.ctxStrokeStyle;		// used to fill caps
		ctx.strokeStyle = pen.ctxStrokeStyle;
		switch (pen.style) {
		case DASH:
			ctx.setLineDash([4, 4]);
			break;
		case DOT:
			ctx.setLineDash([2, 2]);
			break;
		case DASH_DOT:
			ctx.setLineDash([4, 2, 2, 2]);
			break;
		case DASH_DOT_DOT:
			ctx.setLineDash([4, 2, 2, 2, 2, 2]);
			break;
		}
	}

	//
	// GObj.getNPt {joj 18/09/20}
	//
	// not a valid function for all GObjs
	// compiler ensures function not called with an invalid GObj type
	//
	GObj.prototype.getNPt = function() {
		return this.ptx.length;
	}

	//
	// GObj.setNPt
	//
	// not a valid function for all GObj types
	// compiler ensures function not called with an invalid GObj type
	//
	GObj.prototype.setNPt = function(n) {
		let oldn = this.ptx.length;
		n = (n < 0) ? 0 : n;
		if (oldn != n) {
			this.firstUpdate();
			this.ptx.length = n;
			this.pty.length = n;
			this.update();
		}
		return oldn; // {joj 24/09/20}
	}

	//
	// GObj.getPtX {joj 18/09/20}
	//
	// not a valid function for all GObjs
	// compiler ensures function not called with an invalid GObj type
	//
	GObj.prototype.getPtX = function(n) {
		return this.ptx[n];
	}

	//
	// GObj.getPtY {joj 18/09/20}
	//
	// not a valid function for all GObjs
	// compiler ensures function not called with an invalid GObj type
	//
	GObj.prototype.getPtY = function(n) {
		return this.pty[n];
	}

	//
	// GObj.setOpacity
	//
	GObj.prototype.setOpacity = function(opacity, nsteps = 0, wait = 0) {
		if ((this.opacity == opacity) && (wait == 0))	// {joj 17/11/17}
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.opacity = opacity;
			this.update(UPDATEMEMBERS);
			return 0;
		}
		new OpacityTracker(this, opacity, nsteps, wait);
		return wait;
	}

	//
	// GObj.setOptions
	//
	GObj.prototype.setOptions = function(options) {
		//console.log("setOptions(" + options + ")");
		let oldOptions = this.options;
		if (this.options != options) {
			this.firstUpdate();
			this.options = options;
			this.update();
		}
		return oldOptions;
	}

	//
	// GObj.setPen
	//
	GObj.prototype.setPen = function(pen) {
		let oldPen = this.pen;
		if (this.pen != pen) {
			this.firstUpdate();
			if (this.pen)
				this.pen.remove(this);
			this.pen = pen;
			if (this.pen)
				this.pen.add(this);
			this.update();
		}
		return oldPen;
	}

	//
	// GObj.setPin
	//
	GObj.prototype.setPin = function(pinx, piny, nsteps, wait) {
		return this.translatePin(pinx - this.pinx, piny - this.piny, nsteps, wait);
	}

	//
	// GObj.setPos
	//
	GObj.prototype.setPos = function(x, y, nsteps, wait) {
		return this.translate(x - this.tx, y - this.ty, nsteps, wait);
	}

	//
	// GObj.setSz
	//
	GObj.prototype.setSz = function(w, h, nsteps = 0, wait = 0) {
		let gw = this.mbb0.x1 - this.mbb0.x0;
		let gh = this.mbb0.y1 - this.mbb0.y0;
		if ((w / gw == this.sx) && (h / gh == this.sy) && (wait == 0))	// {joj 2/11/20}
			return;
		let newsx = (gw == 0) ? 1 : w / gw;	// {joj 10/08/17}
		let newsy = (gh == 0) ? 1 : h / gh;	// {joj 10/08/17}
		if (nsteps <= 0) {
			this.firstUpdate();
			this.sx = newsx;
			this.sy = newsy;
			this.setTransform2D();
			this.update(UPDATEMEMBERS);
			return 0;
		}
		new ScaleTracker(this, newsx / this.sx, newsy / this.sy, nsteps, wait);
		return wait;
	}

	//
	// GObj.setTransform2D
	//
	GObj.prototype.setTransform2D = function() {
		this.transform2D.setIdentity();
		if ((this.sx != 1) || (this.sy != 1))
			this.transform2D.scale(this.sx, this.sy);
		if (this.angle != 0) {
			//this.transform2D.translate(-this.pinx, -this.piny);	// CHECK
			this.transform2D.rotate(this.angle);
			//this.transform2D.translate(this.pinx, this.piny);	// CHECK
		}
		this.transform2D.translate(this.tx, this.ty);
	}


	//
	// GObj.setTextureOff
	//
	GObj.prototype.setTextureOff = function(textureOffX, textureOffY, nsteps = 0, wait = 0) {
		if ((this.textureOffX == textureOffX) && (this.textureOffY == textureOffY) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.textureOffX = textureOffX;
			this.textureOffY = textureOffY;
			this.update();
			return 0;
		}
		//new TxtOffTracker(this, textureOffX, textureOffY, nsteps, wait);
		//return wait;
		return 0;
	}

	//
	// GObj.setTxt
	//
	GObj.prototype.setTxt = function(txt) {
		let oldTxt = this.txt;
		txt = sprintf.apply(txt || "", arguments);
		if (txt != this.txt) {
			this.firstUpdate();
			this.txt = txt;
			this.update();
		}
		return oldTxt;
	}

	//
	// GObj.setTxt3
	//
	GObj.prototype.setTxt3 = function(txtPen, font, txt) {
		txtPen = txtPen || defaultPen;
		font = font || defaultFont;
		txt = txt || "";
		if (arguments.length > 3) {
			let args = [this.txt];
			for (let i = 3; i < arguments.length; i++)
				args.push(arguments[i]);
			txt = sprintf.apply(this, args);
		}
		this.setTxtPen(txtPen);
		this.setFont(font);
		this.setTxt (txt);
	}

	//
	// GObj.setTxtOff
	//
	GObj.prototype.setTxtOff = function(txtOffX, txtOffY, nsteps = 0, wait = 0) {
		if ((this.txtOffX == txtOffX) && (this.txtOffY == txtOffY) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.txtOffX = txtOffX;
			this.txtOffY = txtOffY;
			this.update();
			return 0;
		}
		new TxtOffTracker(this, txtOffX, txtOffY, nsteps, wait);
		return wait;
	}

	//
	// GObj.setTxtPen
	//
	GObj.prototype.setTxtPen = function(txtPen) {
		let oldTxtPen = this.txtPen;
		if (txtPen == 0)			// {joj 16/09/20}
			txtPen = defaultPen;
		if (txtPen != this.txtPen) {
			this.firstUpdate();
			this.txtPen = txtPen;
			this.update();
		}
		return oldTxtPen;
	}

	//
	// GObj.translate
	//
	// if ticks > 0, tracker will be called even if dx and dy are zero
	//
	GObj.prototype.translate = function(dx, dy, ticks = 0, wait = 0) {
		if ((dx == 0) && (dy == 0) && (wait == 0))
			return 0;
		if ((ticks & ~RT) <= 0) {		// {joj 25/11/20}
			this.firstUpdate();
			this.tx += dx;
			this.ty += dy;
			this.setTransform2D();
			this.update(FIREMOVED | UPDATEMEMBERS);
			return 0;
		}
		new TranslateTracker(this, dx, dy, ticks, wait);
		return wait;
	}

	//
	// GObj.translatePin
	//
	GObj.prototype.translatePin = function(dx, dy, nsteps = 0, wait = 0) {
		if ((dx == 0) && (dy == 0) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.pinx += dx;
			this.piny += dy;
			this.setTransform2D();
			this.update(FIREMOVED | UPDATEMEMBERS);
			return 0;
		}
		new TranslatePinTracker(this, dx, dy, nsteps, wait);
		return wait;
	}

	//
	// GObj.translatePt
	//
	GObj.prototype.translatePt = function(n, dx, dy, nsteps = 0, wait = 0) {
		if (n > this.ptx.length)
			return 0;
		if ((dx == 0) && (dy == 0) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.ptx[n] += dx;
			this.pty[n] += dy;
			this.calculateMbb0();
			this.update();
			return 0;
		}
		new PtTracker(this, n, dx, dy, nsteps, wait);
		return wait;
	}

	//
	// GObj.ungrab
	//
	GObj.prototype.ungrab = function() {
		//console.log("UNGRAB");
		grab = 0;
	}

	//
	// GObj.update
	//
	GObj.prototype.update = function(updFlags = 0) {
		this.updFlags |= (updFlags | UPDATE);
		this.lastUpdate = 2*tick + asyncPhaseUpdate;
		//if ((updFlags & FIREMOVED) && this.handler["eventMOVED"]) {
		//	let currentThread = $getCurrentThread();
		//	for (let i = 0; i < this.handler["eventMOVED"].length; i++)
		//		callEventHandler(this.handler["eventMOVED"][i].pc, this.handler["eventMOVED"][i].obj);
		//	$resumeThread(currentThread);
		//}
		if ((updFlags & UPDATEMEMBERS) && this.gobjs) {
			for (let i = 0; i < this.gobjs.length; i++) {
				this.gobjs[i].firstUpdate();		// needed in case group members haven't been updated
				this.gobjs[i].update(updFlags);
			}
		}
		if (this.handler["eventUPDATED"]) {
			let handler = this.handler["eventUPDATED"];
			for (let k = 0; k < handler.length; k++) {
				let r = callEventHandler(handler[k].pc, handler[k].obj);
				if ((r & PROPAGATE) == 0)
					break;
			}
		}
	}

	//
	// GObj.updateTxtMbb
	//
	GObj.prototype.updateTxtMbb = function(bGObj, bTxt) {
		if (bTxt) {
			let h, hoptions, maxw = 0, txt, voptions, x0, y0;
			ctx.save();
			ctx.font = this.font.ctxFont;
			txt = this.txt.split("\n");
			for (let i = 0; i < txt.length; i++)
				maxw = Math.max(ctx.measureText(txt[i]).width + 1, maxw);
			h = this.font.sz * txt.length;
			hoptions = this.options & HMASK;
			voptions = this.options & VMASK;
			if (hoptions == HLEFT) {
				x0 = this.mbb0.x0;
			} else if (hoptions == HRIGHT) {
				x0 = this.mbb0.x1 - maxw;
			} else {
				x0 = (this.mbb0.x0 + this.mbb0.x1 - maxw) / 2;
			}
			x0 += this.txtOffX;	// {joj 12/07/17}
			if (voptions == VTOP) {
				y0 = this.mbb0.y0;
			} else if (voptions == VBOTTOM) {
				y0 = this.mbb0.y1 - h;
			} else {
				y0 = (this.mbb0.y0 + this.mbb0.y1 - h) / 2;
			}
			y0 += this.txtOffY;	// {joj 12/07/17}
			if (bGObj == 0 && bTxt) {	// txt ONLY
				this.mbb.set(x0, y0, x0 + maxw, y0 + h);	// set mbb
			} else {
				this.mbb.add(x0, y0, x0 + maxw, y0 + h);	// add to existing mbb
			}
			this.mbb.inflate(1, 1); // to be sure
			ctx.restore();
		}
	}

	//
	// GObj.setClip (internal)
	//
	// used by GObj.draw()
	// recursive so transform is calculated in "reverse" order
	//
	GObj.prototype.setClip = function(ctx) {
		if (this.grp)
			this.grp.setClip(ctx);
		ctx.transform(this.transform2D.a, this.transform2D.b, this.transform2D.c, this.transform2D.d, this.transform2D.e, this.transform2D.f);
		if (this.clipPath) {
			for (let i = 0; i < this.clipPath.path.length; i++) {
				ctx.beginPath();
				let path = this.clipPath.path[i];
				switch (path.type) {
				case E$PATH:
					ctx.ellipse(path.x + path.w / 2, path.y + path.h / 2, path.w / 2, path.h / 2, 0, 0, 2 * Math.PI);
					break;
				case R$PATH:
					ctx.rect(path.x, path.y, path.w, path.h);
					break;
				}
				ctx.clip();
			}
		}
	}

	//
	// GObj.isInClip {joj 07/10/20}
	//
	// needed because isPointInPath(x, y) ignores clip regions
	// calculates transform for Gobj.hit
	// calculates transform recursively so calculated in "reverse" order
	// need to check point is in all clip regions (their intersection)
	//
	GObj.prototype.isInClip = function(ctx, x, y) {
		if (this.grp && this.grp.isInClip(ctx, x, y) == 0)	// recursive call
			return 0;
		ctx.transform(this.transform2D.a, this.transform2D.b, this.transform2D.c, this.transform2D.d, this.transform2D.e, this.transform2D.f);
		if (this.clipPath) {
			for (let i = 0; i < this.clipPath.path.length; i++) {
				ctx.beginPath();
				switch (this.clipPath.path[i].type) {
				case E$PATH:
					ctx.ellipse(this.clipPath.path[i].x + this.clipPath.path[i].w / 2, this.clipPath.path[i].y + this.clipPath.path[i].h / 2, this.clipPath.path[i].w / 2, this.clipPath.path[i].h / 2, 0, 0, 2 * Math.PI);
					break;
				case R$PATH:
					ctx.rect(this.clipPath.path[i].x, this.clipPath.path[i].y, this.clipPath.path[i].w, this.clipPath.path[i].h);
					break;
				}
				if (ctx.isPointInPath(x, y) == 0)
					return 0;
			}
		}
		return 1;
	}

	//
	// Arc constructor
	//
	// NB: used by Pie
	//
	function Arc(grp, _layer, options, pen, brush, x, y, cx, cy, rx, ry, startAngle, spanAngle, txtPen, font, txt) {
		//console.log("new Arc()");
		GObj.call(this, grp, _layer, options, x, y);
		if (this.pen = pen)
			pen.add(this);
		if (this.brush = brush)
			brush.add(this);
		this.mbb0.set(cx - rx, cy - ry, cx + rx, cy + ry);	// CHECK {joj 25/09/20}
		this.rx = rx;
		this.ry = ry;
		this.startAngle = startAngle * Math.PI / 180;		// convert to radians
		this.spanAngle = spanAngle * Math.PI / 180;			// convert to radians
		this.txtPen = txtPen || defaultPen;
		this.font = font || defaultFont;
		this.txt = txt || "";
		if (arguments.length > 15) {
			let args = [this.txt];
			for (let i = 15; i < arguments.length; i++)
				args.push(arguments[i]);
			this.txt = sprintf.apply(this, args);			// this ignored
		}
	}
	Arc.prototype = Object.create(GObj.prototype);

	//
	// Arc.draw
	//
	Arc.prototype.draw = function(flags) {
		//console.log("Arc.draw()");
		if (this.drawOpacity && (flags || this.layer.mbbs.overlap(this.mbb))) {
			let ctx = this.layer.ctx;
			ctx.save();
			if (this.clipped) {
				ctx.setTransform(sx, 0, 0, sy, tx, ty);
				this.setClip(ctx);
			} else {
				ctx.transform(this.drawT2D.a, this.drawT2D.b, this.drawT2D.c, this.drawT2D.d, this.drawT2D.e, this.drawT2D.f);
			}
			if (this.brush || this.pen) {
				ctx.beginPath();
				let cx = this.mbb0.x0 + this.rx;
				let cy = this.mbb0.y0 + this.ry;
				let theta = this.startAngle + this.spanAngle;
				if (this instanceof Pie) {	// {joj 17/10/20}
					ctx.save();
					ctx.translate(cx, cy);
					ctx.scale(this.rx, this.ry);
					ctx.moveTo(0, 0);
					ctx.lineTo(Math.cos(this.startAngle), Math.sin(this.startAngle));
					ctx.arc(0, 0, 1, this.startAngle, theta, theta < 0);
					ctx.closePath();
					ctx.restore();
				} else {
					ctx.save();
					ctx.translate(cx, cy);
					ctx.scale(this.rx, this.ry);
					ctx.arc(0, 0, 1, this.startAngle, theta, theta < 0);
					if (this.closed)
						ctx.closePath();
					ctx.restore();
				}
				ctx.globalAlpha = this.layer.opacity * this.drawOpacity;	// {joj 04/09/20}
				if (brushVisible(this.brush)) {
					ctx.save();
					ctx.fillStyle = this.brush.ctxFillStyle;
					if (this.brush.options == 0) {
						ctx.setTransform(1, 0, 0, 1, 0, 0);
					} else {
						ctx.setTransform(1, this.drawT2D.b, this.drawT2D.c, 1, 0, 0);
					}
					ctx.fill();
					ctx.restore();
				}
				if (this.pen) {
					ctx.save();
					this.setLineStyle(ctx, this.pen);
					this.drawStartCap(ctx, 0);	// draw start cap
					this.drawEndCap(ctx, 0);	// draw end cap
					this.drawStartCap(ctx, 1);	// setup clip for drawing line
					this.drawEndCap(ctx, 1);	// setup clip for drawing line
					ctx.stroke();
					ctx.restore();
				}
			}
			this.drawTxt(ctx);
			ctx.restore();
		}
	}

	//
	// Arc.getStartAngle
	//
	Arc.prototype.getStartAngle = function() {
		return this.startAngle * 180 / Math.PI;	// convert to degrees
	}

	//
	// Arc.getSpanAngle
	//
	Arc.prototype.getSpanAngle = function() {
		return this.spanAngle * 180 / Math.PI;	// convert to degrees
	}

	//
	// Arc.hit
	//
	Arc.prototype.hit = function(x, y) {
		let r = 0;
		if (this.drawOpacity || (this.options & HITINVISIBLE)) {
			ctx.save();
			if (this.isInClip(ctx, x, y) == 0) {	// test if in clip region and computes ctx.transform
				ctx.restore();
				return 0;
			}
			ctx.beginPath();
			let cx = this.mbb0.x0 + this.rx;
			let cy = this.mbb0.y0 + this.ry;
			let theta = this.startAngle + this.spanAngle;
			if (this instanceof Pie) {	// {joj 17/10/20}
				ctx.save();
				ctx.translate(cx, cy);
				ctx.scale(this.rx, this.ry);
				ctx.moveTo(0, 0);
				ctx.lineTo(Math.cos(this.startAngle), Math.sin(this.startAngle));
				ctx.arc(0, 0, 1, this.startAngle, theta, theta < 0);
				ctx.closePath();
				ctx.restore();
			} else {
				ctx.save();
				ctx.translate(cx, cy);
				ctx.scale(this.rx, this.ry);
				ctx.arc(0, 0, 1, this.startAngle, theta, theta < 0);
				if (this.closed)
					ctx.closePath();
				ctx.restore();
			}
			if (this instanceof Pie)				// {joj 17/10/20}
				r = ctx.isPointInPath(x, y);		// hit fill (inside)
			if (r == 0 && this.pen) {				// try border
				ctx.lineWidth = this.pen.width;
				r = ctx.isPointInStroke(x, y);		// hit border drawn by pen
			}
			ctx.restore();
		}
		return r ? 1 : 0;	// NEEDED
	}

	//
	// Arc.rotateAngleHelper
	//
	// flag == 0	rotateStartAngle
	// flag == 1	rotateSpanAngle
	// flag == 2	setStartAngle
	// flag == 3	setSpanAngle
	//
	Arc.prototype.rotateAngleHelper = function(flag, theta, nsteps = 0, wait = 0) {
		//console.log("Arc.rotateAngleHelper(flag=" + flag + "theta=" + theta + ")");
		theta *= Math.PI / 180;						// convert to radians
		if (flag == 2) {							// {joj 23/03/18}
			theta -= this.startAngle;				// make relative
		} else if (flag == 3) {
			theta -= this.spanAngle;				// make relative
		}
		if (nsteps <= 0) {
			this.firstUpdate();
			if (flag & 1) {							// {joj 23/03/18}
				this.spanAngle += theta;
			} else {
				this.startAngle += theta;
			}
			this.update();
			return 0;
		}
		new ArcRotateTracker(this, flag & 1, theta, nsteps, wait);
		return wait;
	}

	//
	// Arc.rotateSpanAngle
	//
	Arc.prototype.rotateSpanAngle = function(theta, nsteps, wait) {
		//console.log("Arc.rotateSpanAngle(theta=" + theta + ")");
		return this.rotateAngleHelper(1, theta, nsteps, wait);
	}

	//
	// Arc.rotateStartAngle
	//
	Arc.prototype.rotateStartAngle = function(theta, nsteps, wait) {
		//console.log("Arc.rotateStartAngle(theta=" + theta + ")");
		return this.rotateAngleHelper(0, theta, nsteps, wait);
	}

	//
	// Arc.setSpanAngle {joj 23/03/18}
	//
	Arc.prototype.setSpanAngle = function(angle, nsteps, wait) {
		//console.log("Arc.setSpanAngle(angle=" + angle + ") this.spanAngle=" + this.spanAngle);
		return this.rotateAngleHelper(3, angle, nsteps, wait);
	}

	//
	// Arc.setStartAngle {joj 23/03/18}
	//
	Arc.prototype.setStartAngle = function(angle, nsteps, wait) {
		//console.log("Arc.setStartAngle(angle=" + angle + ")");
		return this.rotateAngleHelper(2, angle, nsteps, wait);
	}

	//
	// Arc.updateMbb
	//
	Arc.prototype.updateMbb = function(opacity, drawT2D, clipped, flags) {
		//console.log("Arc.updateMbb()");
		this.drawOpacity = opacity * this.opacity;
		if (flags || this.layer.updFlags || (this.updFlags && (this.lastUpdate != this.lastUpdateDraw || asyncPhaseUpdate))) {	// async needed as multiple asysnc events can be executed
			this.drawOpacity = opacity * this.opacity;
			this.clipped = (clipped || this.clipPath) ? 1 : 0;
			this.closed = this.options & CLOSED;
			this.layer.mbbs.add(this.mbb);
			let bGObj = brushVisible(this.brush) || this.pen;
			let bTxt = this.txt.length && this.txtPen && this.font;
			if (this.drawOpacity && (bGObj || bTxt)) {
				this.drawT2D.set(this.transform2D);
				this.drawT2D.multiplyBA(drawT2D);
				this.mbb.set(this.mbb0);
				this.mbb.inflate(1);	// {joj 12/07/17}
				if (this.pen)
					this.mbb.inflate(((this.pen.width == 0) ? 1 : (this.pen.width + 1) / 2));
				this.updateTxtMbb(bGObj, bTxt);
				this.drawT2D.transformMbb(this.mbb);
				this.layer.mbbs.add(this.mbb);
			} else {
				this.mbb.set();
			}
			this.lastUpdateDraw = this.lastUpdate;	// NB
		}
		this.updFlags = 0;
	}

	//
	// Bezier constructor
	//
	// a Bezier curve is a list of points with the point <0,0> positioned at <x,y>
	// the points specified as either ABSOLUTE or RELATIVE to previous point
	//
	function Bezier(grp, _layer, options, pen, brush, x, y) {
		//console.log("new Bezier()");
		GObj.call(this, grp, _layer, options, x, y);
		if (this.pen = pen)
			pen.add(this);
		if (this.brush = brush)
			brush.add(this);
		this.closed = 0;
		let npt = (arguments.length - 7) / 2;	// {joj 24/09/20}
		Object.defineProperty(this, "ptx", {value:[], writable:1});	// non enumerable {joj 16/10/20}
		Object.defineProperty(this, "pty", {value:[], writable:1});	// non enumerable {joj 16/10/20}
		for (let i = 0; i < npt; i++) {
			this.ptx[i] = arguments[7 + i*2];
			this.pty[i] = arguments[7 + i*2 + 1];
			if (((options & ABSOLUTE) == 0) && (i > 0)) {
				this.ptx[i] += this.ptx[i-1];
				this.pty[i] += this.pty[i-1]
			}
		}
		this.calculateMbb0();
	}
	Bezier.prototype = Object.create(GObj.prototype);

	//
	// Bezier,draw
	//
	Bezier.prototype.draw = function(flags) {
		//console.log("Bezier.draw()");
		if (this.drawOpacity && (flags || this.layer.mbbs.overlap(this.mbb))) {
			let ctx = this.layer.ctx;
			ctx.save();
			if (this.clipped) {
				ctx.setTransform(sx, 0, 0, sy, tx, ty);
				this.setClip(ctx);
			} else {
				ctx.transform(this.drawT2D.a, this.drawT2D.b, this.drawT2D.c, this.drawT2D.d, this.drawT2D.e, this.drawT2D.f);
			}
			ctx.globalAlpha = this.layer.opacity * this.drawOpacity;	// {joj 04/09/20}
			if (brushVisible(this.brush)) {
				ctx.save();
				ctx.beginPath();
				ctx.moveTo(this.ptx[0], this.pty[0]);
				for (let i = 1; i < this.ptx.length; i += 3)
					ctx.bezierCurveTo(this.ptx[i], this.pty[i], this.ptx[i+1], this.pty[i+1], this.ptx[i+2], this.pty[i+2]);
				if (this.closed)
					ctx.closePath();
				ctx.fillStyle = this.brush.ctxFillStyle;
				ctx.fill();
				ctx.restore();
			}
			if (this.pen) {
				ctx.save();
				this.setLineStyle(ctx, this.pen);
				this.drawStartCap(ctx, 0);
				this.drawEndCap(ctx, 0);
				this.drawStartCap(ctx, 1);
				this.drawEndCap(ctx, 1);
				ctx.beginPath();
				ctx.moveTo(this.ptx[0], this.pty[0]);
				for (let i = 1; i < this.ptx.length; i += 3)
					ctx.bezierCurveTo(this.ptx[i], this.pty[i], this.ptx[i+1], this.pty[i+1], this.ptx[i+2], this.pty[i+2]);
				if (this.closed)
					ctx.closePath();
				ctx.stroke();
				ctx.restore();
			}
			this.drawTxt(ctx);
			ctx.restore();
		}
	}

	//
	// Bezier.hit
	//
	Bezier.prototype.hit = function(x, y) {
		//console.log("Bezier.hit");
		let r = 0;
		if (this.drawOpacity || (this.options & HITINVISIBLE)) {
			ctx.save();
			if (this.isInClip(ctx, x, y) == 0) {	// test if in clip region and computes ctx.transform
				ctx.restore();
				return 0;
			}
			ctx.beginPath();
			ctx.moveTo(this.ptx[0], this.pty[0]);
			var n = this.ptx.length;
			for (let i = 1; i < n; i += 3)
				ctx.bezierCurveTo(this.ptx[i], this.pty[i], this.ptx[i+1], this.pty[i+1], this.ptx[i+2], this.pty[i+2]);
			if (this.closed) {
				ctx.closePath();
				r = ctx.isPointInPath(x, y);		// hit fill (inside)
			}
			if (r == 0 && this.pen) {				// try border
				ctx.lineWidth = this.pen.width;
				r = ctx.isPointInStroke(x, y);		// hit border drawn by pen
			}
			ctx.restore();
		}
		return r ? 1 : 0;	// NEEDED
	}

	//
	// Bezier.setPt
	//
	Bezier.prototype.setPt = function(n, x, y, nsteps = 0, wait = 0) {
		//console.log("Bezier.setPt()");
		if (nsteps <= 0) {
			this.firstUpdate();
			this.ptx[n] = x;
			this.pty[n] = y;
			//this.calculateMbb0();
			this.update();
			return 0;
		}
		new PtTracker(this, n, x - this.ptx[n], y - this.pty[n], nsteps, wait);
		return wait;
	}

	//
	// Bezier.updateMbb
	//
	Bezier.prototype.updateMbb = function(opacity, drawT2D, clipped, flags) {
		//console.log("Bezier.updateMbb()");
		this.drawOpacity = opacity * this.opacity;
		if (flags || this.layer.updFlags || (this.updFlags && (this.lastUpdate != this.lastUpdateDraw || asyncPhaseUpdate))) {	// async needed as multiple async events can be executed
			this.drawOpacity = opacity * this.opacity;
			this.clipped = (clipped || this.clipPath) ? 1 : 0;
			this.closed = this.options & CLOSED;
			this.layer.mbbs.add(this.mbb);
			let bGObj = brushVisible(this.brush) || this.pen;
			let bTxt = this.txt.length && this.txtPen && this.font;
			if (this.drawOpacity && (bGObj || bTxt)) {
				this.drawT2D.set(this.transform2D);
				this.drawT2D.multiplyBA(drawT2D);
				this.mbb.set(this.mbb0);
				this.mbb.inflate(1);
				if (this.pen)
					this.mbb.inflate(((this.pen.width == 0) ? 1 : (this.pen.width + 1) / 2));
				this.updateTxtMbb(bGObj, bTxt);
				this.drawT2D.transformMbb(this.mbb);
				this.layer.mbbs.add(this.mbb);
			} else {
				this.mbb.set();
			}
			this.lastUpdateDraw = this.lastUpdate;	// NB
		}
		this.updFlags = 0;
	}

	//
	// Ellipse constructor
	//
	function Ellipse(grp, _layer, options, pen, brush, x, y, ix, iy, iw, ih, txtPen, font, txt) {
		GObj.call(this, grp, _layer, options, x, y);
		if (this.pen = pen)
			pen.add(this);
		if (this.brush = brush)
			brush.add(this);
		Object.defineProperty(this, "ptx", {value:[], writable:1});	// non enumerable {joj 16/10/20}
		Object.defineProperty(this, "pty", {value:[], writable:1});	// non enumerable {joj 16/10/20}
		this.ptx[0] = ix;			// keep 4 points {joj 24/09/20}
		this.pty[0] = iy;
		this.ptx[1] = ix + iw;
		this.pty[1] = iy + ih;
		this.calculateMbb0();
		this.txtPen = txtPen || defaultPen;
		this.font = font || defaultFont;
		this.txt = txt || "";
		if (arguments.length > 14) {
			let args = [this.txt];
			for (let i = 14; i < arguments.length; i++)
				args.push(arguments[i]);
			this.txt = sprintf.apply(this, args);
		}
	}
	Ellipse.prototype = Object.create(GObj.prototype);

	//
	// Ellipse.draw
	//
	Ellipse.prototype.draw = function(flags) {
		//console.log("Ellipse.draw tick=" + tick);
		if (this.drawOpacity && (flags || this.layer.mbbs.overlap(this.mbb))) {
			let ctx = this.layer.ctx;
			ctx.save();
			if (this.clipped) {
				ctx.setTransform(sx, 0, 0, sy, tx, ty);
				this.setClip(ctx);
			} else {
				ctx.transform(this.drawT2D.a, this.drawT2D.b, this.drawT2D.c, this.drawT2D.d, this.drawT2D.e, this.drawT2D.f);
			}
			ctx.beginPath();
			let x0 = this.mbb0.x0;
			let y0 = this.mbb0.y0;
			let x1 = this.mbb0.x1;
			let y1 = this.mbb0.y1;
			if (ctx.ellipse) {
				ctx.ellipse((x0 + x1) / 2, (y0 + y1) / 2, Math.abs(x1 - x0) / 2, Math.abs(y1 - y0) / 2, 0, 0, 2 * Math.PI);
			} else {
				ctx.save();
				ctx.translate(x0, y0);
				ctx.scale((x1 - x0) / 2, (y1 - y0) / 2);
				ctx.arc(1, 1, 1, 0, Math.PI*2);
				ctx.restore();
			}
			ctx.globalAlpha = this.layer.opacity * this.drawOpacity;	// {joj 04/09/20}
			if (brushVisible(this.brush)) {
				ctx.save();
				ctx.fillStyle = this.brush.ctxFillStyle;
				if (this.brush.options == 0)
					ctx.setTransform(1, 0, 0, 1, 0, 0);
				ctx.fill();
				ctx.restore();
			}
			if (this.pen) {
				this.setLineStyle(ctx, this.pen);
				ctx.stroke();
			}
			this.drawTxt(ctx);
			ctx.restore();
		}
	}

	//
	// Ellipse.hit
	//
	// test against virtual co-ordinates
	// isPointInStroke() and isPointInPath() don't take into account clipping
	//
	Ellipse.prototype.hit = function(x, y) {
		//console.log("Ellipse.hit");
		let r = 0;
		if (this.drawOpacity || (this.options & HITINVISIBLE)) {
			ctx.save();
			if (this.isInClip(ctx, x, y) == 0) {	// test if in clip region and compute ctx.transform
				ctx.restore();
				return 0;
			}
			ctx.beginPath();
			let x0 = this.mbb0.x0;
			let y0 = this.mbb0.y0;
			let x1 = this.mbb0.x1;
			let y1 = this.mbb0.y1;
			ctx.ellipse((x0 + x1) / 2, (y0 + y1) / 2, (x1 - x0) / 2, (y1 - y0) / 2, 0, 0, 2 * Math.PI);
			r = ctx.isPointInPath(x, y);			// hit fill (inside)
			if (r == 0 && this.pen) {				// try border
				ctx.lineWidth = this.pen.width;
				r = ctx.isPointInStroke(x, y);		// hit border drawn by pen
			}
			ctx.restore();
		}
		return r ? 1 : 0;	// NEEDED
	}

	//
	// Ellipse.toString
	//
	Ellipse.prototype.toString = function() {
		return "Ellipse " + this.txt;
	}

	//
	// Ellipse.updateMbb
	//
	Ellipse.prototype.updateMbb = function(opacity, drawT2D, clipped, flags) {
		Rectangle.prototype.updateMbb.call(this, opacity, drawT2D, clipped, flags);
	}

	//
	// Ellipse.getNPt {joj 24/09/20}
	//
	Ellipse.prototype.getNPt = function() {
		return 2;
	}

	//
	// Ellipse.getPtX {joj 24/09/20}
	//
	Ellipse.prototype.getPtX = function(n) {
		return (n == 0) ? this.ptx[0] : (n == 1 ? this.ptx[1] : 0);
	}

	//
	// Ellipse.getPtY {joj 24/09/20}
	//
	Ellipse.prototype.getPtY = function(n) {
		return (n == 0) ? this.pty[0] : (n == 1 ? this.pty[1] : 0);
	}

	//
	// Ellipse.calculateMbb0 {joj 25/09/20}
	//
	Ellipse.prototype.calculateMbb0 = function() {
		return Rectangle.prototype.calculateMbb0.call(this);
	}

	//
	// Ellipse.translatePt {joj 24/09/20}
	//
	Ellipse.prototype.translatePt = function(n, dx, dy, nsteps, wait) {
		return Rectangle.prototype.translatePt.call(this, n, dx, dy, nsteps, wait);
	}

	//
	// Ellipse.setPt {joj 24/09/20}
	//
	Ellipse.prototype.setPt = function(n, x, y, nsteps, wait) {
		return Rectangle.prototype.setPt.call(this, n, x, y, nsteps, wait);
	}

	//
	// Ellipse2 (alternative constructor)
	//
	function Ellipse2(grp, layer, options, pen, brush, x, y, iw, ih, txtPen, font, txt) {
		var args = [grp, layer, options, pen, brush, x, y, 0, 0, iw, ih, txtPen, font, txt];
		for (let i = 12; i < arguments.length; i++)
			args.push(arguments[i]);
		Ellipse.apply(this, args);
	}
	Ellipse2.prototype = Ellipse.prototype;

	//
	// Image constuctor
	//
	// NB: images loaded asynchronously
	//
	function Image(grp, _layer, options, pen, url, x, y, ix, iy, iw, ih) {
		//console.log("Image url=" + url + " x=" + x + " y=" + y + " ix=" + ix + " iy=" + iy + " iw=" + iw + " ih=" + ih);
		GObj.call(this, grp, _layer, options, x, y);
		this.url = url;
		if (this.pen = pen)
			pen.add(this);
		this.brush = 0;
		Object.defineProperty(this, "ptx", {value:[], writable:1});	// non enumerable {joj 16/10/20}
		Object.defineProperty(this, "pty", {value:[], writable:1});	// non enumerable {joj 16/10/20}
		this.ptx[0] = ix;
		this.pty[0] = iy;
		this.ptx[1] = ix + iw;
		this.pty[1] = iy + ih;
		this.calculateMbb0();
		this.image = new window.Image();
		this.image.imageObj = this;
		this.image.onload = imageLoaded;
		this.image.src = url;
	}
	Image.prototype = Object.create(GObj.prototype);

	//
	// imageLoaded
	//
	// handles the asychronous loading of images
	// set aysncPhaseUpdate to image redrawn asynchronously
	//
	function imageLoaded(image) {
		this.imageObj.updFlags = UPDATE;
		asyncPhaseUpdate = 1;
		drawChanges();		// clears asyncPhaseUpdate {joj 28/11/17}
	}

	//
	// Image.calculateMbb0 {joj 25/09/20}
	//
	Image.prototype.calculateMbb0 = function() {
		return Rectangle.prototype.calculateMbb0.call(this);
	}

	//
	// Image.updateMbb
	//
	Image.prototype.updateMbb = function(opacity, drawT2D, clipped, flags) {
		//console.log("Image.updateMbb opacity=" + opacity);
		if (flags || this.layer.updFlags || (this.updFlags && (this.lastUpdate != this.lastUpdateDraw || asyncPhaseUpdate))) {	// async needed as multiple async events can be executed
			//console.log("tick=" + tick + " Image updateMbb txt=\"" + this.txt + "\" flags=" + flags + " lastUpdate=" + this.lastUpdate + " lastUpdateDraw=" + this.lastUpdateDraw + " asyncPhaseUpdate=" + asyncPhaseUpdate);
			this.drawOpacity = opacity * this.opacity;
			this.clipped = (clipped || this.clipPath) ? 1 : 0;
			this.layer.mbbs.add(this.mbb);
			let bTxt = this.txt.length && this.txtPen && this.font;
			if (this.drawOpacity) {
				this.drawT2D.set(this.transform2D);
				this.drawT2D.multiplyBA(drawT2D);
				this.mbb.set(this.ptx[0], this.pty[0], this.ptx[1], this.pty[1]);
				this.mbb.inflate(1);	// {joj 12/07/17}
				if (this.pen)
					this.mbb.inflate(((this.pen.width == 0) ? 1 : (this.pen.width + 1) / 2));
				this.updateTxtMbb(1, bTxt);
				this.drawT2D.transformMbb(this.mbb);
				this.layer.mbbs.add(this.mbb);
			} else {
				this.mbb.set();
			}
			this.lastUpdateDraw = this.lastUpdate;	// NB
		}
		this.updFlags = 0;
	}

	//
	// Image.draw
	//
	Image.prototype.draw = function(flags) {
		//console.log("Image.draw");
		if (this.drawOpacity && (flags || this.layer.mbbs.overlap(this.mbb))) {
			let ctx = this.layer.ctx;
			ctx.save();
			if (this.clipped) {
				ctx.setTransform(sx, 0, 0, sy, tx, ty);
				this.setClip(ctx);
			} else {
				ctx.transform(this.drawT2D.a, this.drawT2D.b, this.drawT2D.c, this.drawT2D.d, this.drawT2D.e, this.drawT2D.f);
			}
			let x0 = this.mbb0.x0;
			let y0 = this.mbb0.y0;
			let x1 = this.mbb0.x1;
			let y1 = this.mbb0.y1;
			ctx.globalAlpha = this.layer.opacity * this.drawOpacity;	// {joj 04/09/20}
			ctx.drawImage(this.image, x0, y0, x1 - x0, y1 - y0);
			if (this.pen) {
				ctx.beginPath();
				ctx.rect(x0, y0, x1 - x0, y1 - y0);
				this.setLineStyle(ctx, this.pen);
				ctx.stroke();
			}
			this.drawTxt(ctx);
			ctx.restore();
		}
	}

	//
	// Image.hit
	//
	Image.prototype.hit = function(x, y) {
		let r = 0;
		if (this.drawOpacity || (this.options & HITINVISIBLE)) {
			ctx.save();
			if (this.isInClip(ctx, x, y) == 0) {	// test if in clip region and computes ctx.transform
				ctx.restore();
				return 0;
			}
			ctx.beginPath();	// why is this needed?
			let x0 = this.mbb0.x0;
			let y0 = this.mbb0.y0;
			let x1 = this.mbb0.x1;
			let y1 = this.mbb0.y1;
			ctx.rect(x0, y0, x1 - x0, y1 - y0)
			r = ctx.isPointInPath(x, y);			// hit fill (inside)
			if (r == 0 && this.pen) {				// try border
				ctx.lineWidth = this.pen.width;
				r = ctx.isPointInStroke(x, y);		// hit border drawn by pen
			}
			ctx.restore();
		}
		return r ? 1 : 0;	// NEEDED
	}

	//
	// Line constructor
	//
	// a line is a list of points with the point <0,0> positioned at <x,y>
	// the points can be specified as either ABSOLUTE or RELATIVE to the previous point
	//
	function Line(grp, _layer, options, pen, x, y) {
		GObj.call(this, grp, _layer, options, x, y);
		if (this.pen = pen)
			pen.add(this);
		this.brush = 0;		// used by Polygon
		this.closed = 0;	// 0:Line 1:Polygon
		let npt = (arguments.length - 6) / 2;
		Object.defineProperty(this, "ptx", {value:[], writable:1});	// non enumerable {joj 16/10/20}
		Object.defineProperty(this, "pty", {value:[], writable:1});	// non enumerable {joj 16/10/20}
		for (let i = 0; i < npt; i++) {
			this.ptx[i] = arguments[6 + i*2];
			this.pty[i] = arguments[6 + i*2 + 1];
			if (((options & ABSOLUTE) == 0) && (i > 0)) {
				this.ptx[i] += this.ptx[i-1];
				this.pty[i] += this.pty[i-1]
			}
		}
		this.calculateMbb0();
	}
	Line.prototype = Object.create(GObj.prototype);

	//
	// Line.hit
	//
	Line.prototype.hit = function(x, y) {
		//console.log("Line.hit");
		let r = 0;
		if (this.drawOpacity || (this.options & HITINVISIBLE)) {
			ctx.save();
			if (this.isInClip(ctx, x, y) == 0) {	// test if in clip region and computes ctx.transform
				ctx.restore();
				return 0;
			}
			ctx.beginPath();
			ctx.moveTo(this.ptx[0], this.pty[0]);
			for (let i = 1; i < this.ptx.length; i++)
				ctx.lineTo(this.ptx[i], this.pty[i]);
			if (this.closed) {
				ctx.closePath();
				r = ctx.isPointInPath(x, y);		// hit fill (inside)
			}
			if (r == 0 && this.pen) {				// try border
				ctx.lineWidth = this.pen.width;
				r = ctx.isPointInStroke(x, y);		// hit border drawn by pen
			}
			ctx.restore();
		}
		return r ? 1 : 0;	// NEEDED
	}

	//
	// Line.setPt
	//
	Line.prototype.setPt = function(n, x, y, nsteps = 0, wait = 0) {
		if ((this.ptx[n] == x) && (this.pty[n] == y) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.ptx[n] = x;
			this.pty[n] = y;
			this.calculateMbb0();
			//debug("Line.setPt n=%d ptx[%d]=%d pty[%d]=%d", n, n, this.ptx[n], n, this.pty[n]);
			this.update();
			return 0;
		}
		new PtTracker(this, n, x - this.ptx[n], y - this.pty[n], nsteps, wait);
		return wait;
	}

	//
	// Line.drawCircleCap (helper)
	//
	GObj.prototype.drawCircleCap = function(ctx, start, scale, clip) {
		if (((this.pen.caps & 0xff) == ROUND_START) && ((this.pen.caps & 0xff0000) == ROUND_END)) {	// use native lineCap
			ctx.lineCap = "round";
			return;
		}
		let p = start ? 0 : this.ptx.length - 1;
		let sz = this.pen.width*scale;
		ctx.save();
		ctx.beginPath();
		if (clip) {
			ctx.save();
			ctx.setTransform(1, 0, 0, 1, 0, 0);
			ctx.rect(0, 0, canvas.width, canvas.height);
			ctx.restore();
			sz = sz/2 > 0.25 ? sz/2 - 0.25 : 0
			ctx.ellipse(this.ptx[p], this.pty[p], sz, sz, 0, 0, 2*Math.PI);
			ctx.restore();
			ctx.clip("evenodd");
		} else {
			ctx.ellipse(this.ptx[p], this.pty[p], sz/2, sz/2, 0, 0, 2*Math.PI);
			if (this.pen.options == 0)
				ctx.setTransform(1, 0, 0, 1, 0, 0);
			ctx.fill();
			ctx.restore();
		}
	}

	//
	// Line.drawSquareCap (helper)
	//
	GObj.prototype.drawSquareCap = function(ctx, start, scale, clip) {
		let p0 = 0, p1 = 1;
		if (start == 0) {
			p0 = this.ptx.length - 1;
			p1 = p0 - 1;
		}
		let dx = this.ptx[p0] - this.ptx[p1];
		let dy = this.pty[p0] - this.pty[p1];
		let radians = Math.atan2(dy, dx);
		let sz = this.pen.width * scale;
		ctx.save();
		ctx.beginPath();
		ctx.translate(this.ptx[p0], this.pty[p0]);
		ctx.rotate(radians);
		if (clip) {
			ctx.save();
			ctx.setTransform(1, 0, 0, 1, 0, 0);
			ctx.rect(0, 0, canvas.width, canvas.height);
			ctx.restore();
			ctx.rect(-sz/2, -sz/2, sz, sz);
			ctx.restore();
			ctx.clip("evenodd");
		} else {
			ctx.rect(-sz/2, -sz/2, sz, sz);
			if (this.pen.options == 0)
				ctx.setTransform(1, 0, 0, 1, 0, 0);
			ctx.fill();
			ctx.restore();
		}
	}

	//
	// Line.drawArrowCap (helper)
	//
	// rotate and translate line so it is horizontal with tip of arrow head at 0,0
	// make sure head is not bigger than length of line
	//
	GObj.prototype.drawArrowCap = function(ctx, start, angle, scale, clip) {
		let p0 = 0, p1 = 1;
		if (start == 0) {
			p0 = this.ptx.length - 1;
			p1 = p0 - 1;
		}
		let dx = this.ptx[p1] - this.ptx[p0];
		let dy = this.pty[p1] - this.pty[p0];
		let radians = Math.atan2(dy, dx);
		let l = Math.sqrt(dx*dx + dy*dy);
		angle = angle*Math.PI/180/2;	// half angle in radians
		if (clip) {
			dy = (scale > 1 ? this.pen.width*scale : this.pen.width) / 2;
			dx = Math.min(this.pen.width*scale / 2 / Math.tan(angle), l);
			ctx.save();
			ctx.setTransform(1, 0, 0, 1, 0, 0);
			ctx.beginPath();
			ctx.rect(0, 0, canvas.width, canvas.height);
			ctx.restore();
			ctx.save();
			ctx.translate(this.ptx[p0], this.pty[p0]);
			ctx.rotate(radians);
			ctx.rect(-0.75, -dy - 0.5, dx + 0.5, 2*dy + 1);
			ctx.restore();
			ctx.clip("evenodd");
		} else {
			dy = this.pen.width*scale / 2;
			dx = Math.min(dy / Math.tan(angle), l);
			ctx.save();
			ctx.translate(this.ptx[p0], this.pty[p0]);
			ctx.rotate(radians);
			ctx.beginPath();
			ctx.moveTo(0, 0);
			ctx.lineTo(dx, -dy);
			ctx.lineTo(dx, dy);
			ctx.closePath();
			if (this.pen.options == 0)
				ctx.setTransform(1, 0, 0, 1, 0, 0);
			ctx.fill();
			ctx.restore();
		}
	}

	//
	// Line.drawStartCap (helper)
	//
	GObj.prototype.drawStartCap = function(ctx, clip) {
		switch (this.pen.caps & 0x000000ff) {
		case BUTT_START:
			break;
		case ROUND_START:
			this.drawCircleCap(ctx, 1, 1, clip);
			break;
		case SQUARE_START:
			this.drawSquareCap(ctx, 1, this.pen.scs, clip);
			break;
		case ARROW40_START:
			this.drawArrowCap(ctx, 1, 40, this.pen.scs, clip);
			break;
		case ARROW60_START:
			this.drawArrowCap(ctx, 1, 60, this.pen.scs, clip);
			break;
		case ARROW90_START:
			this.drawArrowCap(ctx, 1, 90, this.pen.scs, clip);
			break;
		case CIRCLE_START:
			this.drawCircleCap(ctx, 1, this.pen.scs, clip);
			break;
		}
	}

	//
	// Line.drawEndCap (helper)
	//
	GObj.prototype.drawEndCap = function(ctx, clip) {
		switch (this.pen.caps & 0x00ff0000) {
		case BUTT_END:
			break;
		case ROUND_END:
			this.drawCircleCap(ctx, 0, 1, clip);
			break;
		case SQUARE_END:
			this.drawSquareCap(ctx, 0, this.pen.ecs, clip);
			break;
		case ARROW40_END:
			this.drawArrowCap(ctx, 0, 40, this.pen.ecs, clip);
			break;
		case ARROW60_END:
			this.drawArrowCap(ctx, 0, 60, this.pen.ecs, clip);
			break;
		case ARROW90_END:
			this.drawArrowCap(ctx, 0, 90, this.pen.ecs, clip);
			break;
		case CIRCLE_END:
			this.drawCircleCap(ctx, 0, this.pen.ecs, clip);
			break;
		}
	}

	//
	// Line.setLineWidth (helper)
	//
	// scales ctx.lineWidth when line stroked using identity matrix
	//
	GObj.prototype.setLineWidth = function(ctx) {
		let dx = this.ptx[1] - this.ptx[0];
		let dy = this.pty[1] - this.pty[0];
		let radians = Math.atan2(dy, dx);
		let x0 = this.drawT2D.e;
		let y0 = this.drawT2D.f;
		let x = this.pen.width*Math.cos(radians);
		let y = this.pen.width*Math.sin(radians);
		let x1 = this.drawT2D.a*x + this.drawT2D.b*y + this.drawT2D.e;
		let y1 = this.drawT2D.c*x + this.drawT2D.c*y + this.drawT2D.f;
		ctx.lineWidth = Math.sqrt((x0-x1)*(x0-x1)+(y0-y1)*(y0-y1));
	}

	//
	// Line.draw
	//
	Line.prototype.draw = function(flags) {
		if (this.drawOpacity && (flags || this.layer.mbbs.overlap(this.mbb))) {
			let ctx = this.layer.ctx;
			ctx.save();
			if (this.clipped) {
				ctx.setTransform(sx, 0, 0, sy, tx, ty);
				this.setClip(ctx);
			} else {
				ctx.transform(this.drawT2D.a, this.drawT2D.b, this.drawT2D.c, this.drawT2D.d, this.drawT2D.e, this.drawT2D.f);
			}
			ctx.globalAlpha = this.layer.opacity * this.drawOpacity;	// {joj 04/09/20}
			if (brushVisible(this.brush)) {								// fill line if there is a brush {joj 05/10/20}
				ctx.save();
				ctx.beginPath();
				ctx.moveTo(this.ptx[0], this.pty[0]);
				for (let i = 1; i < this.ptx.length; i++)
					ctx.lineTo(this.ptx[i], this.pty[i]);
				if (this.closed)
					ctx.closePath();
				ctx.fillStyle = this.brush.ctxFillStyle;
				ctx.fill();
				ctx.restore();
			}
			if (penVisible(this.pen)) {
				ctx.save();
				this.setLineStyle(ctx, this.pen);
				this.drawStartCap(ctx, 0);
				this.drawEndCap(ctx, 0);
				this.drawStartCap(ctx, 1);
				this.drawEndCap(ctx, 1);
				ctx.beginPath();
				ctx.moveTo(this.ptx[0], this.pty[0]);
				for (let i = 1; i < this.ptx.length; i++)
					ctx.lineTo(this.ptx[i], this.pty[i]);
				if (this.closed)
					ctx.closePath();
				if (this.pen.type == IMAGEPEN && this.pen.options == 0) {
					this.setLineWidth(ctx);
					ctx.setTransform(1, 0, 0, 1, 0, 0);
				}
				ctx.stroke();
				ctx.restore();
			}
			this.drawTxt(ctx);
			ctx.restore();
		}
	}

	//
	// Line.toString
	//
	Line.prototype.toString = function() {
		return "Line " + this.txt;
	}

	//
	// Line.updateMbbSquareCap (helper)
	//
	// cap at ptx[p],pty[p]
	// TOFIX: make more accurate
	//
	Line.prototype.updateMbbSquareCap = function(p, scale) {
		let sz = this.pen.width*scale;
		sz = Math.sqrt(2*sz*sz)/2 + 1;
		this.mbb.add(this.ptx[p] - sz, this.pty[p] - sz, this.ptx[p] + sz, this.pty[p] + sz);
	}

	//
	// Line.updateMbbArrowCap (helper)
	//
	// line p0 -> p1 arrow head at p1
	//
	Line.prototype.updateMbbArrowCap = function(p0, p1, angle, scale) {
		let dx = this.ptx[p1] - this.ptx[p0];
		let dy = this.pty[p1] - this.pty[p0];
		let radians = Math.atan2(dy, dx);
		angle = angle*Math.PI/180/2;	// half angle in radians
		let l = this.pen.width*scale/Math.sin(angle) / 2;
		dx = l*Math.sin(Math.PI/2 - angle - radians);
		dy = l*Math.cos(Math.PI/2 - angle - radians);
		this.mbb.addPt(this.ptx[p1] - dx, this.pty[p1] - dy);
		dx = l*Math.cos(radians - angle);
		dy = l*Math.sin(radians - angle);
		this.mbb.addPt(this.ptx[p1] - dx, this.pty[p1] - dy);
		this.mbb.inflate(1, 1); // to be sure!
	}

	//
	// Line.updateMbbCircleCap (helper)
	//
	// cap at ptx[p],pty[p]
	//
	Line.prototype.updateMbbCircleCap = function(p, scale) {
		let sz = this.pen.width*scale/2 + 1;
		this.mbb.add(this.ptx[p] - sz, this.pty[p] - sz, this.ptx[p] + sz, this.pty[p] + sz);
	}

	//
	// Line.updateMbbCaps (helper)
	//
	Line.prototype.updateMbbCaps = function() {
		let n = this.ptx.length - 1;
		switch (this.pen.caps & 0x000000ff) {
		case SQUARE_START:
			this.updateMbbSquareCap(0, this.pen.scs);				// start cap at pt[0]
			break;
		case ARROW40_START:
			this.updateMbbArrowCap(1, 0, 40, this.pen.scs);			// start cap at pt[0]
			break;
		case ARROW60_START:
			this.updateMbbArrowCap(1, 0, 60, this.pen.scs);			// start cap at pt[0]
			break;
		case ARROW90_START:
			this.updateMbbArrowCap(1, 0, 90, this.pen.scs);			// start cap at pt[0]
			break;
		case CIRCLE_START:
			this.updateMbbCircleCap(0, this.pen.scs);				// start cap at pt[0]
			break;
		}
		switch (this.pen.caps & 0x00ff0000) {
		case SQUARE_END:
			this.updateMbbSquareCap(n, this.pen.ecs);				// end cap at pt[n]
			break;
		case ARROW40_END:
			this.updateMbbArrowCap(n - 1, n, 40, this.pen.ecs);		// end cap at pt[n]
			break;
		case ARROW60_END:
			this.updateMbbArrowCap(n - 1, n, 60, this.pen.ecs);		// end cap at pt[n]
			break;
		case ARROW90_END:
			this.updateMbbArrowCap(n - 1, n, 90, this.pen.ecs);		// end cap at pt[n]
			break;
		case CIRCLE_END:
			this.updateMbbCircleCap(n, this.pen.ecs);				// end cap at pt[n] {joj 22/02/18}
			break;
		}
	}

	//
	// Line.updateMbb
	//
	Line.prototype.updateMbb = function(opacity, drawT2D, clipped, flags) {
		//console.log("Line.updateMbb updPlags=" + this.updFlags + " lastUpdate=" + this.lastUpdate + " lastUpdateDraw=" + this.lastUpdateDraw + " asyncPhaseUpdate=" + asyncPhaseUpdate);
		if (flags || this.layer.updFlags || (this.updFlags && (this.lastUpdate != this.lastUpdateDraw || asyncPhaseUpdate))) {	// async needed as multiple async events can be executed
			this.drawOpacity = opacity * this.opacity;
			this.clipped = (clipped || this.clipPath) ? 1 : 0;
			this.layer.mbbs.add(this.mbb);
			if (this.drawOpacity && (this.brush || this.pen || (this.txt.length && this.txtPen && this.font))) {
				this.drawT2D.set(this.transform2D);
				this.drawT2D.multiplyBA(drawT2D);
				let pw = this.pen.width / 2 + 1;
				this.mbb.set(this.mbb0.x0 - pw, this.mbb0.y0 - pw, this.mbb0.x1 + pw, this.mbb0.y1 + pw);	// not that accurate - oversized
				this.updateMbbCaps();
				this.drawT2D.transformMbb(this.mbb);
				this.layer.mbbs.add(this.mbb);
			} else {
				this.mbb.set(); // empty mbb
			}
			this.lastUpdateDraw = this.lastUpdate;	// NB:
		}
		this.updFlags = 0;
	}

	//
	// Line2 constructor
	//
	// alternative Line constructor
	// a line is a list of points with the notional point <0,0> positioned at <x,y>
	// the points can be specified as either ABSOLUTE or RELATIVE to the previous point
	// since Line2 uses the first point as <x,y>, the remaining points have
	// to be adjusted accordingly if specified as ABSOLUTE
	//
	function Line2(grp, layer, options, pen) {
		let arg4 = arguments[4] || 0;
		let arg5 = arguments[5] || 0;
		let a = [grp, layer, options, pen, arg4, arg5, 0, 0];
		for (let i = 6; i < arguments.length; i += 2) {
			if (options & ABSOLUTE) {
				a.push(arguments[i] - arg4);		// make ABSOLUTE for Line!
				a.push(arguments[i + 1] - arg5);
			} else {
				a.push(arguments[i]);				// still RELATIVE for Line
				a.push(arguments[i + 1]);
			}
		}
		Line.apply(this, a);
	}
	Line2.prototype = Line.prototype;

	//
	// Pie constructor
	//
	function Pie() {
		Arc.apply(this, arguments);
	}
	Pie.prototype = Object.create(Arc.prototype);	// {joj 17/10/20}

	//
	// Polygon constructor
	//
	function Polygon(grp, layer, options, pen, brush, x, y) {
		let a = [grp, layer, options, pen, x, y];
		for (let i = 7; i < arguments.length; i++)
			a.push(arguments[i]);
		Line.apply(this, a);
		this.closed = 1;
		this.brush = brush;
	}
	Polygon.prototype = Line.prototype;

	//
	// Rectangle constructor
	//
	// mbb0 		- mbb of rectangle (in its own co-ordinates)
	// mbb			- mbb of rectangle as drawn (including text and border) in canvas co-ordinates
	// transform2D	- local transform (translate (tx, ty), scaling (sx, sy) and rotation (angnle about pinX and pinY)
	// drawT2D		- transform used to draw Rectangle on canvas
	//
	function Rectangle(grp, _layer, options, pen, brush, x, y, ix, iy, iw, ih, txtPen, font, txt) {
		//console.log("new Rectangle");
		GObj.call(this, grp, _layer, options, x, y);
		if (this.pen = pen)
			pen.add(this);
		if (this.brush = brush)
			brush.add(this);
		Object.defineProperty(this, "ptx", {value:[], writable:1});	// non enumerable {joj 16/10/20}
		Object.defineProperty(this, "pty", {value:[], writable:1});	// non enumerable {joj 16/10/20}
		this.ptx[0] = ix;
		this.pty[0] = iy;
		this.ptx[1] = ix + iw;
		this.pty[1] = iy + ih;
		this.calculateMbb0();
		this.txtPen = txtPen || defaultPen;
		this.font = font || defaultFont;
		this.txt = txt || "";
		if (arguments.length > 14) {
			let args = [this.txt];
			for (let i = 14; i < arguments.length; i++)
				args.push(arguments[i]);
			this.txt = sprintf.apply(this, args);
		}
		this.rx = 0;
		this.ry = 0;
	}
	Rectangle.prototype = Object.create(GObj.prototype);

	//
	// Rectangle.getNPt {joj 18/09/20}
	//
	Rectangle.prototype.getNPt = function () {
		return 2;
	}

	//
	// Rectangle.getPtX {joj 24/09/20}
	//
	Rectangle.prototype.getPtX = function(n) {
		return (n == 0) ? this.ptx[0] : (n == 1 ? this.ptx[1] : 0);
	}

	//
	// Rectangle.getPtY {joj 24/09/20}
	//
	Rectangle.prototype.getPtY = function(n) {
		return (n == 0) ? this.pty[0] : (n == 1 ? this.pty[1] : 0);
	}

	//
	// Rectangle.getTxtH
	//
	Rectangle.prototype.getTxtH = function () {
		ctx.save();
		//ctx.font = this.font.toString();
		//let w = ctx.measureText(this.txt).width;
		ctx.restore();
		return 10;
	}

	//
	// Rectangle.getTxtW
	//
	Rectangle.prototype.getTxtW = function () {
		ctx.save();
		ctx.font = this.font.ctxFont;
		let w = ctx.measureText(this.txt).width;
		ctx.restore();
		return w;
	}

	//
	// Rectangle.calculateMbb0 {joj 25/09/20}
	//
	Rectangle.prototype.calculateMbb0 = function() {
		if (this.ptx[0] < this.ptx[1]) {
			this.mbb0.x0 = this.ptx[0];
			this.mbb0.x1 = this.ptx[1];
		} else {
			this.mbb0.x0 = this.ptx[1];
			this.mbb0.x1 = this.ptx[0];
		}
		if (this.pty[0] < this.pty[1]) {
			this.mbb0.y0 = this.pty[0];
			this.mbb0.y1 = this.pty[1];
		} else {
			this.mbb0.y0 = this.pty[1];
			this.mbb0.y1 = this.pty[0];
		}
	}

	//
	// Rectangle.translatePt
	//
	Rectangle.prototype.translatePt = function(n, dx, dy, nsteps = 0, wait = 0) {
		n = (n < 0) ? 0 : (n >= 1) ? 1 : 0;
		if ((dx == 0) && (dy == 0) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.ptx[n] += dx;
			this.pty[n] += dy;
			this.calculateMbb0();
			this.update();
			return 0;
		}
		new RectPtTracker(this, n, dx, dy, nsteps, wait);
		return wait;
	}

	//
	// Rectangle.setPt
	//
	Rectangle.prototype.setPt = function(n, x, y, nsteps, wait) {
		n = (n < 0) ? 0 : (n >= 1) ? 1 : 0;	// {joj 24/09/20}
		let dx = x - this.ptx[n];
		let dy = y - this.pty[n];
		return this.translatePt(n, dx, dy, nsteps, wait);
	}

	//
	// Rectangle.getRoundedX {joj 18/09/20}
	//
	Rectangle.prototype.getRoundedX = function() {
		return this.rx;
	}

	//
	// Rectangle.getRoundedY {joj 18/09/20}
	//
	Rectangle.prototype.getRoundedY = function() {
		return this.ry;
	}

	//
	// Rectangle.setRounded
	//
	Rectangle.prototype.setRounded = function(rx, ry, nsteps = 0, wait = 0) {
		if (rx < 0)	// must be +ve
			rx = 0;
		if (ry < 0)	// must be +ve
			ry = 0;
		if ((this.rx == rx) && (this.ry == ry) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.rx = rx;
			this.ry = ry;
			this.update();
			return 0;
		}
		new RoundedTracker(this, rx, ry, nsteps, wait);
		return wait;
	}

	//
	// Rectangle.hit
	//
	Rectangle.prototype.hit = function(x, y) {
		//console.log("Rectangle.hit(x=" + x + ", y=" +y +")");
		let r = 0;
		if (this.drawOpacity || (this.options & HITINVISIBLE)) {
			ctx.save();
			if (this.isInClip(ctx, x, y) == 0) {	// test if in clip region and computes ctx.transform
				ctx.restore();
				return 0;
			}
			ctx.beginPath();
			let x0 = this.mbb0.x0;
			let y0 = this.mbb0.y0;
			let x1 = this.mbb0.x1;
			let y1 = this.mbb0.y1;
			if ((this.rx != 0) || (this.ry != 0)) {				// rounded corners
				let rx = (x1 < x0) ? -this.rx : this.rx;		// {joj 10/05/21} as per draw
				let ry = (y1 < y0) ? -this.ry : this.ry;		// {joj 10/05/21} as per draw
				ctx.moveTo(x0, y0 + ry);
				ctx.quadraticCurveTo(x0, y0, x0 + rx, y0);
				ctx.lineTo(x1 - rx, y0);
				ctx.quadraticCurveTo(x1, y0, x1, y0 + ry);
				ctx.lineTo(x1, y1 - ry);
				ctx.quadraticCurveTo(x1, y1, x1 - rx, y1);
				ctx.lineTo(x0 + rx, y1);
				ctx.quadraticCurveTo(x0, y1, x0, y1 - ry);
				ctx.closePath();
			} else {
				ctx.rect(x0, y0, x1 - x0, y1 - y0);
			}
			r = ctx.isPointInPath(x, y);			// hit fill (inside)
			if (r == 0 && this.pen) {				// try border
				ctx.lineWidth = this.pen.width;
				r = ctx.isPointInStroke(x, y);		// hit border drawn by pen
			}
			ctx.restore();
		}
		return r ? 1 : 0;	// NEEDED
	}

	//
	// Rectangle.updateMbb
	//
	// called when traversing the GObj tree from the top GroupObj (see drawChanges)
	// drawT2D is the transformation so far
	// if the Rectangle has been updated since it was last drawn
	//		mbb added to the layer Mbbs (original position)
	//		this.drawT2D = this.transform2D*draw2D
	// 		this.drawT2D used to compute mbb of Rectangle as drawn (including text and border) on the canvas in canvas co-ordinates
	// 		mbb added to the layer Mbbs (new position)
	// when a layer is redrawn, all GObjs that overlap with its Mbbs are redrawn (clipped to the Mbbs)
	//
	Rectangle.prototype.updateMbb = function(opacity, drawT2D, clipped, flags) {
		//	console.log("Rectangle.updateMbb txt=" + this.txt + " this.updFlags=" + this.updFlags + " lastUpdate=" + this.lastUpdate + " lastUpdateDraw=" + this.lastUpdateDraw + " asyncPhaseUpdate=" + asyncPhaseUpdate);
		if (flags || this.layer.updFlags || (this.updFlags && (this.lastUpdate != this.lastUpdateDraw || asyncPhaseUpdate))) {	// async needed as multiple async events can be executed
			//console.log("Rectangle.updateMbb txt=\"" + this.txt + "\" flags=" + flags + " lastUpdate=" + this.lastUpdate + " lastUpdateDraw=" + this.lastUpdateDraw + " asyncPhaseUpdate=" + asyncPhaseUpdate);
			this.drawOpacity = opacity * this.opacity;
			this.clipped = (clipped || this.clipPath) ? 1 : 0;
			this.layer.mbbs.add(this.mbb);
			let bGObj = brushVisible(this.brush) || this.pen;
			let bTxt = this.txt.length && this.txtPen && this.font;
			if (this.drawOpacity && (bGObj || bTxt)) {
				this.drawT2D.set(this.transform2D);
				this.drawT2D.multiplyBA(drawT2D);
				this.mbb.set(this.mbb0);
				this.mbb.inflate(1);	// {joj 12/07/17}
				if (this.pen)
					this.mbb.inflate(((this.pen.width == 0) ? 1 : (this.pen.width + 1) / 2));
				this.updateTxtMbb(bGObj, bTxt);
				this.drawT2D.transformMbb(this.mbb);
				this.layer.mbbs.add(this.mbb);
			} else {
				this.mbb.set();
			}
			this.lastUpdateDraw = this.lastUpdate;	// NB
		}
		this.updFlags = 0;
	}

	//
	// Rectangle.draw
	//
	Rectangle.prototype.draw = function (flags) {
		//console.log("Rectangle.draw");
		if (this.drawOpacity && (flags || this.layer.mbbs.overlap(this.mbb))) {
			let ctx = this.layer.ctx;
			ctx.save();
			if (this.clipped) {
				ctx.setTransform(sx, 0, 0, sy, tx, ty);
				this.setClip(ctx);
			} else {
				ctx.transform(this.drawT2D.a, this.drawT2D.b, this.drawT2D.c, this.drawT2D.d, this.drawT2D.e, this.drawT2D.f);
			}
			ctx.beginPath();
			let x0 = this.mbb0.x0;
			let y0 = this.mbb0.y0;
			let x1 = this.mbb0.x1;
			let y1 = this.mbb0.y1;
			if ((this.rx != 0) || (this.ry != 0)) {	// rounded corners
				let rx = (x1 < x0) ? -this.rx : this.rx;
				let ry = (y1 < y0) ? -this.ry : this.ry;
				ctx.moveTo(x0, y0 + ry);
				ctx.quadraticCurveTo(x0, y0, x0 + rx, y0);
				ctx.lineTo(x1 - rx, y0);
				ctx.quadraticCurveTo(x1, y0, x1, y0 + ry);
				ctx.lineTo(x1, y1 - ry);
				ctx.quadraticCurveTo(x1, y1, x1 - rx, y1);
				ctx.lineTo(x0 + rx, y1);
				ctx.quadraticCurveTo(x0, y1, x0, y1 - ry);
				ctx.closePath();
			} else {
				ctx.rect(x0, y0, x1 - x0, y1 - y0);
			}
			ctx.globalAlpha = this.layer.opacity * this.drawOpacity;	// {joj 04/09/20}
			if (brushVisible(this.brush)) {
				ctx.save();
				ctx.fillStyle = this.brush.ctxFillStyle;
				if (this.brush.type == IMAGEBRUSH) {
					if (this.brush.options == 0) {
						ctx.setTransform(1, 0, 0, 1, 0, 0);
					} else {
						ctx.setTransform(1, 0, 0, 1, 0, 0);
						ctx.rotate(this.angle*Math.PI/180);
					}
				}
				ctx.fill();
				ctx.restore();
			}
			if (this.pen) {
				this.setLineStyle(ctx, this.pen);
				ctx.stroke();
			}
			this.drawTxt(ctx);
			ctx.restore();
		}
	}

	//
	// Rectangle2 constructor
	//
	function Rectangle2(grp, layer, options, pen, brush, x, y, iw, ih, txtPen, font, txt) {
		var args = [grp, layer, options, pen, brush, x, y, 0, 0, iw, ih, txtPen, font, txt];
		for (let i = 12; i < arguments.length; i++)
			args.push(arguments[i]);
		Rectangle.apply(this, args);
	}
	Rectangle2.prototype = Rectangle.prototype;

	//
	// Txt constructor
	//
	// NB: iw and ih set to 1 {joj 10/08/17}
	//
	function Txt(grp, layer, options, x, y, txtPen, font, txt) {
		var args = [grp, layer, options, 0, 0, x, y, 0, 0, 1, 1, txtPen, font, txt];
		for (let i = 8; i < arguments.length; i++)
			args.push(arguments[i]);
		Rectangle.apply(this, args);
	}
	Txt.prototype = Object.create(Rectangle.prototype);

	//
	// Spline constructor
	//
	// cardinal spline implemented using a Bezier curve
	//
	function Spline(grp, _layer, options, pen, brush, tension, x, y) {
		//console.log("new Spline()");
		GObj.call(this, grp, _layer, options, x, y);
		if (this.pen = pen)
			pen.add(this);
		if (this.brush = brush)
			brush.add(this);
		this.closed = 0;
		this.tension = tension;
		let npt = (arguments.length - 8) / 2;
		this.sptx = [];		// for spline points
		this.spty = [];
		Object.defineProperty(this, "ptx", {value:[], writable:1});	// non enumerable {joj 16/10/20}
		Object.defineProperty(this, "pty", {value:[], writable:1});	// non enumerable {joj 16/10/20}
		this.c0x = 0;		// used when converting spline to Bezier
		this.c0y = 0;
		this.c1x = 0;		// used when converting spline to Bezier
		this.c1y = 0;
		for (let i = 0; i < npt; i++) {
			this.sptx[i] = arguments[8 + i*2];
			this.spty[i] = arguments[8 + i*2 + 1];
			if (((options & ABSOLUTE) == 0) && (i > 0)) {
				this.sptx[i] += this.sptx[i-1];
				this.spty[i] += this.spty[i-1]
			}
		}
		//this.convertToBezier();	// {joj 07/10/20}
		//this.mbb0 = new Mbb();
		//this.calculateMbb0();
	}
	Spline.prototype = Object.create(GObj.prototype);

	//
	// Spline.calcCurve (helper)
	//
	Spline.prototype.calcCurve = function(p0, p1, p2, t) {
		let dx = this.sptx[p2] - this.sptx[p0];
		let dy = this.spty[p2] - this.spty[p0];
		this.c0x = this.sptx[p1] - t*dx;
		this.c0y = this.spty[p1] - t*dy;
		this.c1x = this.sptx[p1] + t*dx;
		this.c1y = this.spty[p1] + t*dy;
	}

	//
	// Spline.calcCurveEnd (helper)
	//
	Spline.prototype.calcCurveEnd = function(end, adj, t) {
		this.c0x = t*(this.sptx[adj] - this.sptx[end]) + this.sptx[end];
		this.c0y = t*(this.spty[adj] - this.spty[end]) + this.spty[end];
	}

	//
	// Spline.convertToBezier (helper)
	//
	// NB: code thanks to "a coder name Floris" http://floris.briolas.nl/floris/?p=144
	// NB: uses Bezier curves to generate Microsoft GDI+ cardinal splines
	//
	Spline.prototype.convertToBezier = function() {
		//console.log("convertToBezier");
		let npt = this.sptx.length;
		if (npt == 0)
			return;

		this.closed = this.options & CLOSED;
		let nbp = npt*3 - 2;
		if (this.closed)
			nbp += 3;
		this.ptx.length = nbp;	// {joj 12/07/17}
		this.pty.length = nbp;	// {joj 12/07/17}
		let t = this.tension / 3;

		if (this.closed == 0) {
			this.calcCurveEnd(0, 1, t);
			this.ptx[0] = this.sptx[0];
			this.pty[0] = this.spty[0]
			this.ptx[1] = this.c0x;
			this.pty[1] = this.c0y;
		}

		for (let i = 0; i < npt - ((this.closed) ? 1 : 2); i++) {
			this.calcCurve(i, i+1, (i+2) % npt, t);
			this.ptx[3*i + 2] = this.c0x;
			this.pty[3*i + 2] = this.c0y;
			this.ptx[3*i + 3] = this.sptx[i+1];
			this.pty[3*i + 3] = this.spty[i+1];
			this.ptx[3*i + 4] = this.c1x;
			this.pty[3*i + 4] = this.c1y;
		}

		if (this.closed) {
			this.calcCurve(npt-1, 0, 1, t);
			this.ptx[nbp-2] = this.c0x;
			this.pty[nbp-2] = this.c0y;
			this.ptx[0] = this.sptx[0];
			this.pty[0] = this.spty[0];
			this.ptx[1] = this.c1x;
			this.pty[1] = this.c1y;
			this.ptx[nbp-1] = this.sptx[0];
			this.pty[nbp-1] = this.spty[0];
		} else {
			this.calcCurveEnd(npt-1, npt-2, t);
			this.ptx[nbp-2] = this.c0x;
			this.pty[nbp-2] = this.c0y;
			this.ptx[nbp-1] = this.sptx[npt-1];
			this.pty[nbp-1] = this.spty[npt-1];
		}
		this.calculateMbb0();	// {joj 07/10/20}
	}

	//
	// Spline.draw
	//
	Spline.prototype.draw = function(flags) {
		Bezier.prototype.draw.call(this, flags);
	}

	//
	// Spline.getNPt {joj 02/11/20}
	//
	// NB: access sptx.length (not ptx.length)
	//
	Spline.prototype.getNPt = function() {
		return this.sptx.length;
	}

	//
	// Spline.getPtX {joj 02/11/20}
	//
	// NB: access sptx (not ptx)
	//
	Spline.prototype.getPtX = function(n) {
		return this.sptx[n];
	}

	//
	// Spline.getPtY {joj 02/11/20}
	//
	// NB: access spty (not pty)
	//
	Spline.prototype.getPtY = function(n) {
		return this.spty[n];
	}

	//
	// Spline.getTension
	//
	Spline.prototype.getTension = function() {
		return this.tension;
	}

	//
	// Spline.hit
	//
	Spline.prototype.hit = function(x, y) {
		//console.log("Spline.hit");
		return Bezier.prototype.hit.call(this, x, y);
	}

	//
	// Spline.setNPt {joj 03/11/20}
	//
	// NB: set sptx.length (not ptx.length)
	// NB: need to rethink setPt functions
	//
	Spline.prototype.setNPt = function(n) {
		let oldn = this.sptx.length;
		this.sptx.length = this.spty.length = n;
		for (let i = oldn; i < n; i++)
			this.sptx[i] = this.spty[i] = 0;
		return oldn;
	}

	//
	// Spline.setTension {joj 03/
	//
	Spline.prototype.setTension = function(tension, nsteps = 0, wait = 0) {
		if ((this.tension == tension) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.tension = tension;
			this.convertToBezier();	// expensive
			this.update();
			return 0;
		}
		new SplineTensionTracker(this, tension - this.tension, nsteps, wait);
		return wait;
	}

	//
	// Spline.translatePt {joj 03/11/20}
	//
	// NB: only ABSOLUTE points supported
	//
	Spline.prototype.translatePt = function(n, dx, dy, nsteps = 0, wait = 0) {
		if (n > this.sptx.length)
			return 0;
		if ((dx == 0) && (dy == 0) && (wait == 0))
			return 0;
		if (nsteps <= 0) {
			this.firstUpdate();
			this.sptx[n] += dx;
			this.spty[n] += dy;
			this.convertToBezier();	// expensive
			this.update();
			return 0;
		}
		new SplinePtTracker(this, n, dx, dy, nsteps, wait);
		return wait;
	}

	//
	// Spline.setPt
	//
	// NB: only ABSOLUTE points supported
	//
	Spline.prototype.setPt = function(n, x, y, nsteps, wait) {
		return this.translatePt(n, x - this.sptx[n], y - this.spty[n], nsteps, wait);
	}

	//
	// Spline.updateMbb
	//
	// differs from Bezier.update as it nees to call converToBezier
	//
	Spline.prototype.updateMbb = function(opacity, drawT2D, clipped, flags) {
		//console.log("Spline.updateMbb()");
		if (flags || this.layer.updFlags || (this.updFlags && (this.lastUpdate != this.lastUpdateDraw || asyncPhaseUpdate))) {	// async needed as multiple async events can be executed
			this.convertToBezier();
			this.drawOpacity = opacity * this.opacity;
			this.clipped = (clipped || this.clipPath) ? 1 : 0;
			this.layer.mbbs.add(this.mbb);
			let bGObj = brushVisible(this.brush) || this.pen;
			let bTxt = this.txt.length && this.txtPen && this.font;
			if (this.drawOpacity && (bGObj || bTxt)) {
				this.drawT2D.set(this.transform2D);
				this.drawT2D.multiplyBA(drawT2D);
				this.mbb.set(this.mbb0);
				this.mbb.inflate(1);
				if (this.pen)
					this.mbb.inflate(((this.pen.width == 0) ? 1 : (this.pen.width + 1) / 2));
				this.updateTxtMbb(bGObj, bTxt);
				this.drawT2D.transformMbb(this.mbb);
				this.layer.mbbs.add(this.mbb);
			} else {
				this.mbb.set();
			}
			this.lastUpdateDraw = this.lastUpdate;	// NB
		}
		this.updFlags = 0;
	}

	//
	// Group constructor
	//
	// gobjs always in order of creation
	// problem if gobj destroyed
	//
	function Group(grp, layer, options, x, y, ix, iy, iw, ih) {
		Rectangle.call(this, grp, layer, options, 0, 0, x, y, ix, iy, iw, ih);
		Object.defineProperty(this, "gobjs", {value:[], writable:1});	// non enumerable {joj 16/11/20}
	}
	Group.prototype = Object.create(Rectangle.prototype);

	//
	// Group.add
	//
	Group.prototype.add = function(gobj) {
		//console.log("Group.add length=" + (this.gobjs.length + 1));
		this.firstUpdate();
		this.gobjs.push(gobj);
		this.update(UPDATEGOBJSLEN);
	}

	// Group.draw
	Group.prototype.draw = function(flags) {
		if (this.drawOpacity)
			Rectangle.prototype.draw.call(this, flags);
	}

	//
	// Group.updateMbb
	//
	Group.prototype.updateMbb = function(opacity, drawT2D, clipped, flags) {
		//console.log("Group.updateMbb flags=" + flags + " gobsj.length=" + this.gobjs.length);
		this.drawOpacity = opacity * this.opacity;
		this.clipped = (clipped || this.clipPath) ? 1 : 0;
		Rectangle.prototype.updateMbb.call(this, opacity, drawT2D, this.clipped, flags);
		this.drawT2D.set(this.transform2D);
		this.drawT2D.multiplyBA(drawT2D);
		for (let i = 0; i < this.gobjs.length; i++)
			this.gobjs[i].updateMbb(opacity*this.opacity, this.drawT2D, this.clipped, flags);
	}

	//
	// Path constructor
	//
	// avoid using Path2D
	// not implemented in all browsers
	// difficult to do hit testing with clipping
	// isPointInStroke() and isPointInPath() don't take into account clipping
	//
	function Path(path) {
		this.path = [];
		this.path.push(path);
	}

	//
	// E$
	//
	function E$(x, y, w, h) {
		return new Path({type:E$PATH, x:x, y:y, w:w, h:h});
	}

	//
	// L$
	//
	function L$(x, y, w, h) {
		console.log("L$");
		//return new Path({type:E$PATH, x:x, y:y, w:w, h:h});
	}

	//
	// R$
	//
	function R$(x, y, w, h) {
		return new Path({type:R$PATH, x:x, y:y, w:w, h:h});
	}

	//
	// global functions
	//

	//
	// rgba
	//
	// red, green, blue and alpha in range 0 to 1)
	//
	function rgba(red, green, blue, alpha = 1.0) {
		red = ((red < 0.0) ? 0 : (red > 1.0) ? 1 : red) * 255;
		green = ((green < 0.0) ? 0 : (green > 1.0) ? 1 : green) * 255;
		blue = ((blue < 0.0) ? 0 : (blue > 1.0) ? 1 : blue) * 255;
		alpha = ((alpha < 0.0) ? 0 : (alpha > 1.0) ? 1 : alpha) * 255;
		return (alpha << 24) + (red << 16) + (green << 8) + (blue | 0);
	}

	//
	// toRGBA (helper)
	//
	function toRGBA(rgba) {
		let a = (rgba >> 24) & 0xff;
		let r = (rgba >> 16) & 0xff;
		let g = (rgba >> 8) & 0xff;
		let b = (rgba & 0xff);
		return "rgba(" + r + "," + g + "," + b + "," + a / 255 + ")";	// NB: alpha must be in range 0 to 1
	}

	//
	// penVisible (helper)
	//
	function penVisible(pen) {
		return pen && (pen.type != NULLPEN);
	}

	//
	// brushVisible (helper)
	//
	function brushVisible(brush ) {
		return brush && (brush.type != NULLBRUSH);
	}

	//
	// setBgBrush
	//
	function setBgBrush(brush) {
		if (bgBrush != brush) {
			$g[0].firstUpdate();
			bgBrush = brush;
			$g[0].update(UPDATEALL);
		}
	}

	//
	// setBgPen
	//
	function setBgPen(pen) {
		if (bgPen != pen) {
			$g[0].firstUpdate();
			bgPen = pen;
			$g[0].update(UPDATEALL);
		}
	}

	//
	// setVirtualWindow
	//
	function setVirtualWindow(_vx, _vy, _vw, _vh, _keepAspectRatio = 1) {
		vx = _vx;
		vy = _vy;
		vw = _vw;
		vh = _vh;
		keepAspectRatio = _keepAspectRatio;
		$g[0].ptx[0] = vx;
		$g[0].pty[0] = vy;
		$g[0].ptx[1] = vx + vw;
		$g[0].pty[1] = vy + vh;
		$g[0].calculateMbb0();
		resize();
	}

	//
	// resize
	//
	function resize() {

		if (canvas.width == canvas.clientWidth && canvas.height == canvas.clientHeight)	// {joj 22/12/21
			return;

		//alert("resize canvas width=" + canvas.width + " height=" + canvas.height + " clientWidth=" + canvas.clientWidth + " clientHeight=" + canvas.clientHeight);
		console.log("resize canvas width=" + canvas.width + " height=" + canvas.height);
		console.log("resize canvas.clientWidth=" + canvas.clientWidth + " canvas.clientHeight=" + canvas.clientHeight);
		rtStart = performance.now();

		//
		// prevent setting canvas.width = canvas.height = 0 when $testFlag=1 when there is no visible window
		//
		if ($testFlag == 0) {								// {joj 17/10/20}
			canvas.width = canvas.clientWidth;				// {joj 27/03/16}
			if (canvas.noInitialHeight === true) {			// {joj 22/12/21}
				canvas.height = canvas.clientWidth*vh/vw;	// {joj 20/12/21}
			} else {										// {joj 20/12/21}
				canvas.height = canvas.clientHeight;		// {joj 20/21/21}
			}
		}

		for (let i = 1; i < layer.length; i++) {
			layer[i].canvas.style.left = canvas.offsetLeft + "px";
			layer[i].canvas.style.top = canvas.offsetTop + "px";
			layer[i].canvas.width = canvas.width;
			layer[i].canvas.height = canvas.height;
		}

		//if (overlay) {
		//	overlay.style.left = canvas.offsetLeft + "px";
		//	overlay.style.top = canvas.offsetTop + "px";
		//	overlay.width = canvas.width;
		//	overlay.height = canvas.height;
		//}

		let h = (showStats) ? canvas.height - INFOFONTH - 4 : canvas.height;
		let w = canvas.width;
		sx = w/vw;
		sy = h/vh;
		tx = vx;
		ty = vy;
		if (keepAspectRatio) {
			if (sx > sy) {
				sx = sy;
				tx  -= (vw*sx - w)/2;
			} else {
				sy = sx;
				ty -= (vh*sy - h)/2;
			}
		}

		// {joj 25/09/20}
		if (handler["eventResize"]) {
			for (let i = 0; i < handler["eventResize"].length; i++)
				callEventHandler(handler["eventResize"][i].pc, handler["eventResize"][i].obj, tick);
		}

		//console.log("div clientWidth=" + canvas.parentNode.clientWidth + " clientHeight=" + canvas.parentNode.clientHeight);
		console.log("canvas width=" + canvas.width + " height=" + canvas.height);
		console.log("canvas clientWidth=" + canvas.clientWidth + " clientHeight=" + canvas.clientHeight);
		console.log("vx=" + vx + " vy=" + vy + " vw=" + vw + " vh=" + vh);
		//console.log("sx=" + sx + " sy=" + sy + " tx=" + tx + " ty=" + ty);

		if (!playZero)
			drawAll();
	}

	//
	// drawAll
	//
	function drawAll() {
		$g[0].updFlags |= UPDATEALL;
		drawChanges();
	}

	//
	// drawChanges
	//
	// the GObj mbbs are updated by transversing the GObj tree starting at the "top" GroupObj (g[0])
	// set the initial transform t2D (virtual window => canvas)
	// each layer is drawn in turn
	// if any GObj on the layer has changed
	//		set to clip region (layer Mbbs)
	//		fill clipped region with the background or transparent colour
	// 		redraw each GObj if it overlaps with the clipped rgion
	// performance data can optionally be displayed in the infoWnd
	// the update Mbbs can optionally be displayed
	//
	function drawChanges(flags = 0) {
		//console.log("drawChanges tick=" + tick +" flags=" + flags + " asyncPhaseUpdate=" + asyncPhaseUpdate);
		flags |= $g[0].updFlags & UPDATEALL;	// setBgBrush or setBgPen
		let dtStart = performance.now();
		clearMbbs();							// clear mbbs on overlay canvas
		t2D.set(sx, 0, 0, sy, tx, ty);
		$g[0].updateMbb(1, t2D, 0, flags);
		let redraw = 0;
		for (let i = 0; i < layer.length; i++) {
			if ((layer[i].visible == 0) && (layer[i].opacity == 0) && (layer[i].updFlags == 0) && (layer[i].mbbs.nmbb == 0) && (flags == 0))
				continue
			redraw |= 1;
			let ctx = layer[i].ctx;
			ctx.save();
			ctx.beginPath();
			if (flags || layer[i].updFlags) {
				ctx.rect(0, 0, canvas.width, (showStats) ? canvas.height - INFOFONTH - 4 : canvas.height);
			} else {
				let mbbs = layer[i].mbbs;
				for (let j = 0; j < mbbs.nmbb; j++)
					ctx.rect(mbbs.mbb[j].x0, mbbs.mbb[j].y0, mbbs.mbb[j].width(), mbbs.mbb[j].height());
			}
			ctx.clip();														// create clip region from path
			if (i == 0) {
				ctx.fillStyle = bgBrush ? bgBrush.ctxFillStyle : "white";	// globalAlpha?
				ctx.fillRect(0, 0, canvas.width, canvas.height);
				if (bgPen) {
					$g[0].setLineStyle(ctx, bgPen);
					ctx.strokeRect(0, 0, canvas.width, canvas.height);;
				}
			} else {
				ctx.clearRect(0, 0, canvas.width, canvas.height);
			}
			if (layer[i].visible && layer[i].opacity) {	// {joj 21/10/20}
				//console.log("drawChanges layer=" + i + " gobjs.length=" + layer[i].gobjs.length);	// {joj 03/01/20}
				for (let j = 0; j < layer[i].gobjs.length; j++)
					layer[i].gobjs[j].draw(flags);
			}
			layer[i].updFlags = 0;
			ctx.restore();
		}
		if (redraw)
			(dir == 1) ? nff++ : nfb++;		// number of frames
		let now = performance.now();
		dtLast = now - dtStart;
		rtLast = now - rtStart;
		if (dir == 1) {
			dtf += dtLast;
			rtf += rtLast
		} else {
			dtb += dtLast;
			rtb += rtLast
		}
		swapMbbs();				// swap layer mbbs and clear mbbs
		drawInfoTip();
		drawStats();
		drawMbbs();
		asyncPhaseUpdate = 0;	// clear asyncPhaseUpdate {joj 28/11/17}
	}

	//
	// wheelHandler
	//
	// no need to set asyncPhaseUpdate = 1 as no Vivio events handlers called
	//
	function wheelHandler(e) {
		//console.log("wheelEvent e.deltaY=" + e.deltaY + " e.deltaX=" + e.deltaX);
		let delta = e.deltaX + e.deltaY;	// fix for chrome when shift button pressed
		if (e.ctrlKey) {
			if (delta > 0) {
				if (tps < 100)
					setTPS(tps + 1, 1);
			} else {
				if (tps > 1)
					setTPS(tps - 1, 1);
			}
			showInfoTip("set speed: " + tps + " tps", 0, 1);
		} else {
			if (delta > 0) {
				dir = 1;
				if (e.shiftKey) {
					stop(1);
					if (eventQ) {
						playMode = SNAPTOCHKPT;
						rtStart = performance.now();
						playTo(-1);			// play to next check point or end of animation
						showInfoTip("snap to next checkpoint at tick " + tick, 0, 0);
						drawChanges();
						playMode = PLAY;
					} else {
						showInfoTip("NO more checkpoints (eventQ empty) tick " + tick, 0, 1);
					}
				} else if (eventQ && timer == 0) {
					playMode = SINGLESTEP;
					showInfoTip("step forward: tick " + (tick + 1), 0, 0);
					rtStart = performance.now();
					playTo(tick + 1);
					drawChanges();
					playMode = PLAY;
				}
			} else {
				if (e.shiftKey) {
					stop(1);
					dir = -1;
					let newTick = getPreviousCheckPt();
					if (newTick < tick) {
						playMode = SNAPTOCHKPT;
						showInfoTip("snap to previous checkpoint at tick " + newTick, 0, 0);
						rtStart = performance.now();
						goto(newTick);
						drawChanges();
						playMode = PLAY;
					} else {
						showInfoTip("NO more checkpoints tick " + newTick, 0, 1);
					}
				} else if (timer == 0) {
					playMode = SINGLESTEP;
					dir = -1;
					let newTick = (tick > 0) ? tick - 1 : 0;
					showInfoTip("step backward: tick " + newTick, 0, 0);
					rtStart = performance.now();
					//console.log("wheelHandler: goTo(" + newTick + ")");
					goto(newTick);	// goto 0 even if at 0
					drawChanges();
					playMode = PLAY;
				}
			}
		}
		e.preventDefault();	// why?
		return false;
	}

	//
	// mouseMoveHandler
	//
	// always called from user interface
	// don't remember mouse move events
	// set asyncPhaseUpdate so changes redrawn asynchronously
	//
	function mouseMoveHandler(e) {
		//console.log("mouseMoveHandler");
		mouseX = e.offsetX;
		mouseY = e.offsetY;
		let x = (mouseX - tx) / sx;		// convert to viewport co-ordinates
		let y = (mouseY - ty) / sy;		// convert to viewport co-ordinates

		let flags = 0;
		if (e.button == 0)
			flags |= MB_LEFT;
		else if (e.button == 1)
			flags |= MB_MIDDLE;
		else if (e.button == 2)
			flags |= MB_RIGHT;
		if (e.shiftKey)
			flags |= MB_SHIFT;
		if (e.ctrlKey)
			flags |= MB_CTRL;
		if (e.altKey)
			flags |= MB_ALT;

		if (grab) {
			asyncPhaseUpdate = 1;				// {joj 08/02/19}
			if (typeof grab == "function") { 	// {joj 28/11/17}
				grab(e);						// Menu.grab
			} else {
				//callEventHandler(grab.pc, grab.obj, MM, 0, flags, x, y);
				let handler = grab.handler["eventGRABBED"];
				for (let k = 0; k < handler.length; k++) {
					let r = callEventHandler(handler[k].pc, handler[k].obj, MM, 0, flags, x, y);
					if ((r & PROPAGATE) == 0)
						break;
				}
			}
			drawChanges();	// {joj 28/11/17} clears asyncPhaseUpdate
			e.preventDefault();
			return false;
		}

		let redraw = 0;	// {joj 04/01/21}
		rtStart = performance.now();
		//console.log("mouseMoveHandler x=" + x + " y=" + y);
		FOR: {
			for (let i = layer.length - 1; i >= 0; i--) {
				if (layer[i].opacity == 0)	// {joj 17/10/16}
					continue;
				for (let j = layer[i].gobjs.length - 1; j >= 0; j--) {
					let g = layer[i].gobjs[j];
					if (g.handler["eventMM"]) {	// {joj 07/10/20}
						let handler = g.handler["eventMM"];
						asyncPhaseUpdate = redraw = 1; 	// {joj 04/01/21}
						for (let k = 0; k < handler.length; k++) {
							let r = callEventHandler(handler[k].pc, handler[k].obj, flags, x, y);
							if ((r & PROPAGATE) == 0)
								break FOR;
						}
					}

					if (g.handler["eventEE"]) {
						let hit = ((g.hit(x, y) || (g == $g[0] && (g.options & HITWINDOW) ? 1 : 0)) && inCanvas) ? 1 : 0;	// {joj 22/09/17}
						//if (hit)
						//	console.log("eventEE hit " + g.txt + " enter=" + g.enter);
						if (hit != g.enter) {
							g.enter = hit;
							asyncPhaseUpdate = redraw = 1;	// {joj 04/01/21}
							let handler = g.handler["eventEE"];
							for (let k = 0; k < handler.length; k++) {
								let r = callEventHandler(handler[k].pc, handler[k].obj, hit, x, y);
								if ((r & PROPAGATE) == 0)
									break FOR;
							}
						}
					}

				}
			}
		}
		if (redraw)
			drawChanges();			// {joj 04/01/21}
		canvas.focus();
		//asyncPhaseUpdate = 0;		// {joj 21/11/17} {joj 01/01/21}
	}

	//
	// mouseEnterHandler
	//
	// fired when mouse enters canvas {joj 15/07/17	NOT generated by QBrowserEngine}
	// simply calls mouseMoveHandler
	//
	function mouseEnterHandler(e) {
		//console.log("mouseEnterHandler");
		inCanvas = 1;
		mouseMoveHandler(e);
	}

	//
	// mouseLeaveHandler
	//
	// fired when mouse leaves canvas {joj 15/07/17	NOT generated by QBrowserEngine}
	// simply calls mouseMoveHandler
	//
	function mouseLeaveHandler(e) {
		//console.log("mouseLeaveHandler");
		inCanvas = 0;
		mouseMoveHandler(e);
	}

	//
	// getLayers {joj 08/10/21}
	//
	// called from VivioIDE only
	//
	// returns list of numbers [zIndex, visible, zInidex, visible, ...]
	//
	function getLayers() {
		let a = []
		for (let i = 0; i < layer.length - 1; i++) {
			a.push(layer[i].canvas.style.zIndex)
			a.push(layer[i].visible)
		}
		return a
	}

	//
	// getUIState {joj 07/10/21}
	//
	// called from VivioIDE only
	//
	function getUIState() {
		return ((showMbbs & 1) << 1) | (showStats & 1)
	}

	//
	// setShowMbbs {joj 07/10/21}
	//
	// called internally and from VivioIDE
	//
	function setShowMbbs(b, centred = 1) {
		clearMbbs()
		showMbbs = b
		sessionStorage.setItem("showMbbs", showMbbs)
		showInfoTip(showMbbs ? "show mbbs" : "hide mbbs", centred, 0)
		contextMenu.item[1].txt = showMbbs ? "hide mbbs" : "show mbbs"
		contextMenu.updFlags = 1
		resize();
	}

	//
	// toggleShowMbbs {joj 11/10/21}
	//
	function toggleShowMbbs(centred) {
		setShowMbbs(showMbbs ? 0 : 1, centred)
	}

	//
	// setShowStats {joj 05/10/21}
	//
	// called internally and from VivioIDE
	//
	function setShowStats(b, centred = 1) {
		showStats = b;
		sessionStorage.setItem("showStats", showStats);
		showInfoTip(showStats ? "show runtime stats" : "hide runtime stats", centred, 0)
		contextMenu.item[0].txt = showStats ? "hide runtime stats" : "show runtime stats"
		contextMenu.updFlags = 1
		resize();
	}

	//
	// toggleShowStats {joj 05/10/21}
	//
	// called internally and from VivioIDE
	//
	function toggleShowStats(centred) {
		setShowStats(showStats ? 0 : 1, centred)
	}

	//
	// handleMBEvent
	//
	// always called from user interface
	// callEventHandler creates a separate thread to execute event function
	//
	function handleMBEvent(e, down, flags, x, y) {
		//console.log("handleMBEvent down=" + down + " flags=" + (flags | 0) + " x=" + x + " y=" + y);
		rtStart = performance.now();
		let eventHandled = 0;	// {joj 04/01/21}
		FOR: {
			for (let i = layer.length - 1; i >= 0; i--) {
				if (layer[i].opacity == 0)	// {joj 17/10/16}
					continue;
				for (let j = layer[i].gobjs.length - 1; j >= 0 ; j--) {
					if (layer[i].gobjs[j].handler["eventMB"]) {
						if (layer[i].gobjs[j].hit(x, y)) {
							let handler = layer[i].gobjs[j].handler["eventMB"];
							removeFutureAsyncEvents();				// remove future ASYNC events
							asyncPhaseUpdate = eventHandled = 1;	// {joj 04/01/21}
							for (let k = 0; k < handler.length; k++) {
								addToAsyncEventQ(new AsyncEvent(tick, callEventHandler.bind(null, handler[k].pc, handler[k].obj, down, flags, x, y)));
								let r = callEventHandler(handler[k].pc, handler[k].obj, down, flags, x, y);
//								if (r & REMEMBER)
//									addToAsyncEventQ(new AsyncEvent(tick, callEventHandler.bind(null, handler[k].pc, handler[k].obj, down, flags, x, y)));
								if ((r & PROPAGATE) == 0)
									break FOR;
							}
						}
					}
				}
			}
		}

		if (asyncPhaseUpdate)		// {joj 04/01/21}
			drawChanges();			// {joj 28/11/17} clears asyncPhaseUpdate

		if (eventHandled)			// {joj 04/01/21}
			return;

		if (down) {
			//console.log("handleMBEvent: DEFAULT");
			if (flags & MB_LEFT) {
				if (flags & MB_SHIFT && flags & MB_CTRL) {
					et = nff = nfb = rtf = rtb = dtf = dtb = 0;
					etStart = performance.now();
					showInfoTip("clear stats", 1, 1);
					drawStats();
				} else if (flags & MB_SHIFT) {
					toggleShowStats(0);	// {joj 05/10/21}
				} else if (flags & MB_CTRL) {
					toggleShowMbbs(0);
				} else {
					if (timer) {
						stop(1);
						showInfoTip("stop", 0, 1);
					} else {
						if (eventQ) {						// {joj 12/07/17}
							start(1);
							showInfoTip("start", 0, 1);
						}
					}
				}
			} else if (flags & MB_RIGHT) {	// {joj 28/11/17}
				contextMenu.show(x, y);		// {joj 15/09/20}
			}
		}

	}

	//
	// mouseButtonHandler
	//
	// callEventHandler creates a separate thread to execute event function
	// set asyncPhaseUpdate so changes redrawn asynchronously
	//
	function mouseButtonHandler(e) {
		mouseX = e.offsetX;
		mouseY = e.offsetY;
		if (showStats && (mouseY > canvas.clientHeight - INFOFONTH - 4 - 1))
			return;
		let down = (e.type == "mousedown") ? 1 : 0;	// {joj 11/07/17}
		let flags = 0;
		let x = (mouseX - tx) / sx;		// convert to viewport co-ordinates
		let y = (mouseY - ty) / sy;		// convert to viewport co-ordinates
		if (e.button == 0)
			flags |= MB_LEFT;
		else if (e.button == 1)
			flags |= MB_MIDDLE;
		else if (e.button == 2)
			flags |= MB_RIGHT;
		if (e.shiftKey)
			flags |= MB_SHIFT;
		if (e.ctrlKey)
			flags |= MB_CTRL;
		if (e.altKey)
			flags |= MB_ALT;
		if (grab) {
			asyncPhaseUpdate = 1;				// {joj 03/01/21}
			if (typeof grab == "function") { 	// {joj 28/11/17}
				grab(e);						// Menu.grab
			} else {
				let handler = grab.handler["eventGRABBED"];
				for (let k = 0; k < handler.length; k++) {
					let r = callEventHandler(handler[k].pc, handler[k].obj, MB, down, flags, x, y);
					if ((r & PROPAGATE) == 0)
						break;
				}
			}
			drawChanges();					// clears asyncPhaseUpdate {joj 03/01/21}
		} else {
			handleMBEvent(e, down, flags, x, y);
		}
		e.preventDefault();
		return false;
	}

	//
	// keyDownHandler
	//
	// use e.which for key {joj 26/10/17}
	//
	// NO need to set asyncPhaseUpdate as no events handlers called
	//
	function keyDownHandler(e) {
		var key = e.key === undefined ? e.code : e.key;
		//console.log("keyDownHandler: which=" + e.which + " key=" + key + " repeat=" + e.repeat);
		if (key == "ArrowUp") {
			if (e.shiftKey && e.ctrlKey) {
				etf = etb = nff = nfb = rtf = rtb = dtf = dtb = 0;
				etStart = performance.now();
				showInfoTip("clear stats", 1, 1);
				drawStats();
			} else if (e.shiftKey) {
				toggleShowStats(1);	// {joj 05/10/21}
			} else if (e.ctrlKey) {
				toggleShowMbbs(1);
			} else {
				stop(0);
				etf = etb = nff = nfb = rtf = rtb = dtf = dtb = 0;
				rtStart = performance.now();
				goto(-1);
				showInfoTip("reset", 1, 0);
				drawAll();
			}
			e.preventDefault();
		} else if (key == "ArrowRight") {
			dir = 1;
			if (e.ctrlKey) {
				if (tps < 100)
					setTPS(tps + 1, 1);
				showInfoTip("speed: " + tps + " tps", 1, 1);
			} else if (e.shiftKey) {
				playMode = SNAPTOCHKPT;
				rtStart = performance.now();
				playTo(-1);
				showInfoTip("snap to next checkpoint at tick " + tick, 1, 0);
				drawChanges();
				playMode = PLAY;
			} else if (eventQ && timer == 0) {
				showInfoTip("step forward: tick " + (tick + 1), 1, 0);
				rtStart = performance.now();
				playTo(tick + 1);
				drawChanges();
			}
			e.preventDefault();
		} else if (key == "ArrowLeft") {
			if (e.ctrlKey) {
				if (tps > 1)
					setTPS(tps - 1, 1);
				showInfoTip("speed: " + tps + " tps", 1, 1);
			} else if (e.shiftKey) {
				playMode = SNAPTOCHKPT;
				dir = -1;
				stop(1);
				let toTick = getPreviousCheckPt();
				showInfoTip("snap to previous checkpoint at tick " + toTick, 1, 0);
				rtStart = performance.now();
				goto(toTick);
				drawChanges();
				playMode = PLAY;
			} else if (timer == 0) {
				playMode = SINGLESTEP;
				dir = -1;
				let newTick = (tick > 0) ? tick - 1 : 0;
				showInfoTip("step backward: tick " + newTick, 1, 0);
				rtStart = performance.now();
				goto(newTick);	// goto 0 even if at 0
				drawChanges();
				playMode = PLAY;
			}
			e.preventDefault();
		} else if (key == "ArrowDown") {
			if (timer) {
				stop(1);
				showInfoTip("stop", 1, 1);
			} else {
				start(1);
				showInfoTip("start", 1, 1);
			}
			e.preventDefault();
		}
		return false;
	}

	//
	// keyPressHandler
	//
	// use e.which for key {joj 26/10/17}
	// set asyncPhaseUpdate = 1 so changes redrawn asynchronously
	//
	function keyPressHandler(e) {
		asyncPhaseUpdate = 1;
		let flags = 0;
		if (e.button == 0)
			flags |= MB_LEFT;
		else if (e.button == 1)
			flags |= MB_MIDDLE;
		else if (e.button == 2)
			flags |= MB_RIGHT;
		if (e.shiftKey)
			flags |= MB_SHIFT;
		if (e.ctrlKey)
			flags |= MB_CTRL;
		if (e.altKey)
			flags |= MB_ALT;
		//console.log("keyPressHandler: which=" + e.which + " flags=" + flags + " repeat=" + e.repeat);
		let key = e.which;
		if (e.key == "Delete")
			key = 127;
		let x = (mouseX - tx) / sx;		// convert to viewport co-ordinates
		let y = (mouseY - ty) / sy;		// convert to viewport co-ordinates
		let redraw = 0;
		let remember = 0;
		rtStart = performance.now();
		FOR: {
			for (let i = layer.length - 1; i >= 0; i--) {
				for (let j = layer[i].gobjs.length - 1; j >= 0 ; j--) {
					if (layer[i].gobjs[j].handler["eventKEY"]) {
						let g = layer[i].gobjs[j];
						if (g.hit(x, y)) {
							if (redraw == 0)				// first hit indicates a new path so...
								removeFutureAsyncEvents();	// need to remove future ASYNC events
							redraw = 1;
							let handler = g.handler["eventKEY"];
							for (let k = 0; k < handler.length; k++) {
								let r = callEventHandler(handler[k].pc, handler[k].obj, key, flags, x, y);
								if (r & REMEMBER)
									addToAsyncEventQ(new AsyncEvent(tick, callEventHandler.bind(null, handler[k].pc, handler[k].obj, key, flags, x, y)));
								if ((r & PROPAGATE) == 0)
									break FOR;
							}
						}
					}
				}
			}
		}
		if (redraw)
			drawChanges();
		return false;
	}

	//
	// playAddedEvents
	//
	// play events added by async events (eg fork)
	//
	//function playAddedEvents() {
	//	//console.log("playAddedEvents");
	//	while (eventQ) {
	//		let e = eventQ;
	//		if (e.tick > tick || e.typ == ASYNC)
	//			break;
	//		eventQ = eventQ.next;
	//		if (e.typ == TRACKER) {
	//			e.tracker.action();
	//		} else if (e.typ == RESUMETHREAD) {
	//			$execute(e.thread);
	//		}
	//	}
	//}

	//
	// playTo
	//
	// play events as quickly as possible (ie not governed speed by requestAnimationFrame)
	// saveState at fixed intervals (eg every 1024 ticks) when playing backwards
	// sst can be changed programatically
	//
	function playTo(toTick) {
		//console.log("playTo toTick=" +  toTick);
		atCheckPoint = 0;
		while (eventQ) {
			let e = eventQ;
			if (toTick == -1) {
				if (atCheckPoint && ($testFlag == 0)) // {joj 03/09/20) // && ((e.tick > tick) || (e.tick == tick && e.typ == ASYNC)))
					break;
			} else if ((e.tick > toTick) || (e.tick == toTick && e.typ == ASYNC)) {
				tick = toTick;
				break;
			}
			if (dir == -1) { // check position of this code relative to tests above
				let nextSaveTick = ((tick + sst - 1) / sst | 0) * sst;	// integer
				if (((e.tick > nextSaveTick) || (e.tick == nextSaveTick && e.typ == ASYNC)) && savedState.tick != nextSaveTick) {
					tick = nextSaveTick;	// make sure savedState has correct timestamp
					saveState();
				}
			}
			eventQ = eventQ.next;
			tick = e.tick;
			if (e.typ == TRACKER) {
				e.tracker.action();
			} else if (e.typ == RESUMETHREAD) {
				$execute(e.thread);
			} else {	// ASYNC
				//console.log("play async event");
				asyncPhaseUpdate = 1;
				e.handler();
				lastAsyncEvent = e;
				//playAddedEvents();		// {joj 22/11/17}
				asyncPhaseUpdate = 0;		// clear after playAddedEvents so changes redrawn
			}
			if (gotoTickExecuted) {									// {joj 14/10/20}
				toTick = (playMode == SINGLESTEP) ? tick : toTick;	// {joj 23/10/20}
				gotoTickExecuted = 0;								// {joj 14/10/20}
			}
		}
		if (handler["eventTick"]) {	// {joj 03/09/20}
			for (let i = 0; i < handler["eventTick"].length; i++) // {joj 03/09/20}
				callEventHandler(handler["eventTick"][i].pc, handler["eventTick"][i].obj, tick); // {joj 03/09/20}
		}
	}

	//
	// goto
	//
	function goto(toTick) {
		//console.log("goto(" + toTick + ")");
		asyncPhaseUpdate = 0;									// {joj 15/07/16}
		if (toTick == -1) {										// resets animation (tick = 0)
			//asyncPhaseUpdate = 0;								// {joj 04/09/20}
			stop(1);											//
			for (let i = 1; i < layer.length; i++)				// destroy all layers except...
				canvas.parentNode.removeChild(layer[i].canvas);	// layer [0] {joj 05/10/20}
			layer.length = 0;									// {joj 16/10/16}
			overlayLayer = 0;									// {joj 26/09/20}
			$g.length = 0;										// reset globals
			pens.length = 1;									// {joj 05/01/21} keep defaultPen
			brushes.length = 0;									// {joj 05/01/21}
			fonts.length = 2;									// {joj 05/01/21} keep defaultFont and defaultMenuFont
			vobjs.length = 0;									// reset vobjs
			aobjs.length = 0;									// reset aobjs
			threads.length = 0;									// {joj 05/01/21}
			interrupt.length = 0;								// {joj 05/01/21}
			handler.length = 0;									// reset global event handlers
			checkPt.length = 0;
			savedState = 0;
			atCheckPoint = 0;
			eventQ = 0;
			asyncEventQ.length = 0;
			tick = 0;
			new Layer(0);										// {joj 04/09/20}
			createOverlayLayer();								// {joj 22/10/20}
			makeContextMenu();									// call after overlayLayer created
			playZero = 1;
			$g[0] = new Group(0, 0, 0, 0, 0, 0, 0, canvas.clientWidth, canvas.clientHeight);
			fork(0);
			playTo(0);
			saveState();
			playZero = 0;
			playMode = PLAY;
			return;
		}
		restoreState(toTick);	// restore to nearest state
		playTo(toTick);			// play to tick
		//if (handler["eventStartStop"]) {
		//	for (let i = 0; i < handler["eventStartStop"].length; i++)
		//		callEventHandler(handler["eventStartStop"][i].pc, handler["eventStartStop"][i].obj, isRunning());
		//}
		//if (handler["eventSetTPS"]) {
		//	for (let i = 0; i < handler["eventSetTPS"].length; i++)
		//		callEventHandler(handler["eventSetTPS"][i].pc, handler["eventSetTPS"][i].obj, tps);
		//}
		//drawChanges();
	}

	//
	// gotoTick {joj 14/10/20}
	//
	function gotoTick(toTick) {
		//restoreState(toTick); // restore to nearest state
		gotoTickExecuted = 1; //toTick;
		goto(toTick);
		return 1;		// acts as a wait function
	}

	//
	// animate
	//
	// requestAnimationFrame callback
	//
	function animate(timeStamp) {
		timer = requestAnimationFrame(animate);
		rtStart = performance.now();
		let dTick = ((timeStamp - startTimeStamp) / 1000 * tps) | 0;
		let toTick = (dir == 1) ? startTick + dTick : startTick - dTick;
		if (dir == 1) {	// forward
			if (tick == toTick)
				return;
			while (eventQ) {
				let e = eventQ;
				if ((e.tick > toTick) || (e.tick == toTick && e.typ == ASYNC))
					break;
				tick = e.tick;
				eventQ = eventQ.next;
				if (e.typ == TRACKER) {
					e.tracker.action();
				} else if (e.typ == RESUMETHREAD){
					$execute(e.thread);
				} else { // ASYNC
					e.handler();
					lastAsyncEvent = e;
				}
				if (gotoTickExecuted) {					// {joj 14/10/20}
					toTick = tick;						// {joj 14/10/20}
					gotoTickExecuted = 0;				// {joj 14/10/20}
					startTick = tick;					// {joj 14/10/20}
					startTimeStamp = performance.now();	// {joj 14/10/20}
					break;								// {joj 14/10/20}
				}										// {joj 14/10/20}
			}
			if (handler["eventTick"]) {	// {joj 03/09/20}
				for (let i = 0; i < handler["eventTick"].length; i++) // {joj 03/09/20}
					callEventHandler(handler["eventTick"][i].pc, handler["eventTick"][i].obj, tick); // {joj 03/09/20}
			}
			tick = toTick;
			drawChanges();
			if (eventQ == 0)
				stop(1);	// {joj 27/10/17}
		} else {	// backwards
			if (toTick < 0)
				toTick = 0;
			if (tick == toTick)
				return;
			goto(toTick);
			if (tick == 0) {
				stop(0);
				dir = 1;
			}
			drawChanges();
		}
	}

	//
	// fire
	//
	// call global "eventFire" handlers
	//
	function fire(s) {
		//console.log(s);
		if (handler["eventFire"]) {
			for (let i = 0; i < handler["eventFire"].length; i++)
				callEventHandler(handler["eventFire"][i], 0, s);
		}
		drawChanges();
	}

	//
	// load
	//
	function load(VCode) {
		//console.log("load");
		vcode = new VCode(this);							// vcode changed
		$getCurrentThread = vcode.$getCurrentThread;
		$execute = vcode.$execute;
		$resumeThread = vcode.$resumeThread;
		$suspendThread = vcode.$suspendThread;
		$switchToThread = vcode.$switchToThread;			// {joj 02/01/21}
		$testFlag = vcode.$testFlag;						// {joj 29/10/21}
		asyncPhaseUpdate = 0;								// {joj 15/07/16}
		stop(1);
		for (let i = 1; i < layer.length; i++)				// destroy all canvases except...
			canvas.parentNode.removeChild(layer[i].canvas);	// layer [0] {joj 05/10/20}
		layer.length = 0;									// {joj 16/10/16}
		$g.length = 0;										// reset globals
		vobjs.length = 0;									// reset vobjs
		aobjs.length = 0;									// reset aobjs
		handler.length = 0;									// reset global event handlers
		checkPt.length = 0;
		savedState = 0;
		atCheckPoint = 0;
		eventQ = 0;
		asyncEventQ.length = 0;
		tick = 0;
		new Layer(0);										// {joj 04/09/20}
		createOverlayLayer();								// {joj 22/10/20}
		makeContextMenu();									// call after overlayLayer created
		playZero = 1;
		$g[0] = new Group(0, 0, 0, 0, 0, 0, 0, canvas.clientWidth, canvas.clientHeight);
		fork(0);
		playTo(0);
		saveState();
		playZero = 0;
		playMode = PLAY;
		drawAll();
	}

	//
	// run
	//
	function run() {

		console.log("run()\n");

		isChrome = navigator.userAgent.indexOf("Chrome") > -1;
		isFF = navigator.userAgent.indexOf("Firefox") > -1;

		addEventListener("resize", resize, false);
		canvas = document.getElementById(canvasID);
		canvas.noInitialHeight = canvas.style.height == "";	// {joj 22/12/21}

		if (canvas.mozOpaque !== undefined)
			canvas.mozOpaque = true;
		if (canvas.style.zIndex == "")
			canvas.style.zIndex = 0;
		canvas.focus();
		canvas.addEventListener("wheel", wheelHandler, false);
		canvas.addEventListener("mousemove", mouseMoveHandler, false);
		canvas.addEventListener("mouseenter", mouseEnterHandler, false);
		canvas.addEventListener("mouseleave", mouseLeaveHandler, false);
		canvas.addEventListener("mousedown", mouseButtonHandler, false);
		canvas.addEventListener("mouseup", mouseButtonHandler, false);
		canvas.addEventListener("keydown", keyDownHandler, false);
		canvas.addEventListener("keypress", keyPressHandler, false);
		ctx = canvas.getContext("2d");
		new Layer(0);			// {joj 04/09/20}
		createOverlayLayer();	// {joj 22/10/20}
		canvas.addEventListener("contextmenu", mouseButtonHandler, false);		// {joj 28/11/17}
		showStats = sessionStorage.getItem("showStats") || 0
		showMbbs = sessionStorage.getItem("showMbbs") || 0
		makeContextMenu();		// call after overlayLayer created and showStats and showMbbs have been initialised
		rtStart = performance.now();
		playZero = 1;
		$g[0] = new Group(0, 0, 0, 0, 0, 0, 0, canvas.clientWidth, canvas.clientHeight);
		fork(0);
		playTo(0);
		saveState();
		playZero = 0;
		if ($testFlag)	// {joj 14/10/20}
			playTo(-1);	// {joj 14/10/20}
		drawAll();
		//resize();

	}

	// Menu -------------------------------------- Menu -------------------------------------- Menu

	//
	// Menu constructor
	//
	// menus drawn directly on overlay canvas
	//
	// NB: Menu NOT derived from GObj
	//
	function Menu(bgRGBA, fgRGBA, font, actionf) {
		//console.log("new Menu");
		this.bgRGBA = bgRGBA;
		this.fgRGBA = fgRGBA;
		this.font = font || defaultMenuFont;
		this.actionf = actionf;
		this.layer = overlayLayer;
		this.pen = defaultPen;
		this.h = 0;
		this.w = 0;
		this.item = [];
		this.tx = 0;
		this.ty = 0;
		this.txtPen = defaultPen;
		this.selected = -1;
		this.state = 0;
		this.parent = 0;
		this.updFlags = 1;
	}

	//
	// Menu.addItem
	//
	Menu.prototype.addItem = function(txt, action, flags, value) {
		//console.log("Menu.addItem(txt=\"" + txt + "\", action=" + action + ", flags=" + flags + ", submenu=" + submenu + ")");
		action = action || 0;
		flags = flags || 0;
		value = value || 0;
		this.item.push({"txt":txt, "submenu":0, "action":action, "flags":flags, "value":value, "y":0, "h":0, "updFlags":1});
		this.updFlags = 1;
	}

	//
	// Menu.addSeparator
	//
	Menu.prototype.addSeparator = function() {
		this.item.push({"txt":"", "submenu":0, "action":0, "flags":MENU_SEPARATOR, "value":0, "y":0, "h":0, "updFlags":1});
		this.updFlags = 1;
	}

	//
	// Menu.addSubmenu
	//
	Menu.prototype.addSubmenu = function(txt, action, flags, submenu) {
		action = action || 0;
		flags = flags || 0;
		this.item.push({"txt":txt, "submenu":submenu, "action":action, "flags":MENU_SUBMENU | flags, "value":0, "y":0, "h":0, "updFlags":1});
		this.updFlags = 1;
	}

	//
	// showSubMenu {21/10/20}
	//
	// called from timer
	//
	function showSubMenu(menu, i) {
		let item = menu.item[i];
		item.submenu.parent = menu; // hide link
		item.submenu.show((mouseX - tx) / sx, (mouseY - ty) / sy);
		item.submenu.state = 1;
	}

	//
	// Menu.show
	//
	// x,y co-ordinates in virtual window
	// make sure menu is always visible
	//
	Menu.prototype.show = function(x, y) {
		//console.log("Menu.show x=" + x + ", y=" + y);
		this.state = 0;
		this.selected = -1;
		this.calculateMbbs();
		this.timeStamp = performance.now() | 0;
		x = x*sx + tx;	// convert to canvas co-ordinates
		y = y*sy + ty;	// convert to canvas co-ordinates
		this.tx = (x + this.w > canvas.clientWidth) ? canvas.clientWidth - this.w - 2: x;	// make sure visible
		this.ty = (y + this.h > canvas.clientHeight) ? canvas.clientHeight - this.h - 2: y;	// make sure visible
		this.draw();
		this.thread = $getCurrentThread();
		grab = this.grab.bind(this);
	}

	//
	// Menu.hide
	//
	// last submenu hides all "parent" menus
	//
	Menu.prototype.hide = function() {
		//console.log("Menu.hide");
		grab = 0;
		let ctx = this.layer.ctx;
		ctx.save();
		let menu = this;
		while (menu) {
			ctx.clearRect(menu.tx - 2, menu.ty - 2, menu.w + 4, menu.h + 4);
			menu = menu.parent;
		}
		ctx.restore();
	}

	//
	// Menu.calculateMbbs
	//
	Menu.prototype.calculateMbbs = function() {
		//console.log("Menu.calculateMbbs this.updFlags=" + this.updFlags + " asyncPhaseUpdate=" + asyncPhaseUpdate);
		if (this.updFlags) {
			let ctx = this.layer.ctx;
			ctx.save();
			ctx.font = this.font.ctxFont;
			let fh = this.font.sz;
			this.w = 0;
			this.h = 0;
			let y = 0;
			for (let i = 0; i < this.item.length; i++) {
				this.item[i].y = y;
				if ((this.item[i].flags & MENU_ITEM_TYPE_MASK) == MENU_SEPARATOR) {
					y += this.item[i].h = MENU_SEPARATOR_H;
				} else {
					let w = ctx.measureText(this.item[i].txt).width + 2*MENU_ITEM_BORDER;
					if (((this.item[i].flags & MENU_ITEM_TYPE_MASK) == MENU_SUBMENU) || (this.item[i].flags & MENU_ITEM_CHECKBOX))
						w += fh + 2*MENU_ITEM_BORDER;
					if (w > this.w)
						this.w = w;
					y += this.item[i].h = fh + 2*MENU_ITEM_BORDER;
				}
			}
			this.h = y;
			ctx.restore();
			this.updFlags = 0;
		}
	}

	//
	// Menu.drawItem
	//
	// menu item drawn on overlay canvas
	//
	Menu.prototype.drawItem = function(i) {

		//console.log("Menu.drawItem");
		let x = this.tx;
		let y = this.ty + this.item[i].y;
		let w = this.w;
		let h = this.item[i].h;

		// get context
		let ctx = this.layer.ctx;
		ctx.save();

		// fill background
		ctx.fillStyle = toRGBA(this.bgRGBA);
		ctx.fillRect(x, y, w, h);

		if ((this.item[i].flags & MENU_ITEM_TYPE_MASK) == MENU_SEPARATOR) {	// separator
			ctx.beginPath();
			ctx.moveTo(x + MENU_ITEM_BORDER, y + MENU_SEPARATOR_H / 2);
			ctx.lineTo(x + w - MENU_ITEM_BORDER, y + MENU_SEPARATOR_H / 2);
			ctx.stroke();
		} else {
			ctx.font = this.font.ctxFont;	// item
			let fh = this.font.sz;
			ctx.textBaseline = "top";
			if ((this.selected == i) && ((this.item[i].flags & MENU_ITEM_DISABLED) == 0)) {
				ctx.fillStyle = toRGBA(this.fgRGBA);
				ctx.fillRect(x, y, w, fh + 2*MENU_ITEM_BORDER);
			}
			ctx.fillStyle = (this.item[i].flags & MENU_ITEM_DISABLED) ? "#c0c0c0" : "#000000";
			ctx.fillText(this.item[i].txt, x + MENU_ITEM_BORDER, y + MENU_ITEM_BORDER);

			if ((this.item[i].flags & MENU_ITEM_TYPE_MASK) == MENU_SUBMENU) {	// submenu
				let sz = fh - 2*MENU_ITEM_BORDER;
				ctx.beginPath();
				ctx.moveTo(x + w - sz - 2*MENU_ITEM_BORDER, y + 2*MENU_ITEM_BORDER);
				ctx.lineTo(x + w - 2*MENU_ITEM_BORDER, y + 2*MENU_ITEM_BORDER + sz/2);
				ctx.lineTo(x + w - sz - 2*MENU_ITEM_BORDER, y + sz + 2*MENU_ITEM_BORDER);
				ctx.lineTo(x + w - sz - 2*MENU_ITEM_BORDER, y + 2*MENU_ITEM_BORDER);
				ctx.fill();
			} else if (this.item[i].flags & MENU_ITEM_CHECKBOX) {
				let sz = fh - 2*MENU_ITEM_BORDER;
				ctx.beginPath();
				ctx.rect(x + w - sz - 2*MENU_ITEM_BORDER, y + 2*MENU_ITEM_BORDER, sz, sz);
				ctx.stroke();
				if (this.item[i].value) {
					ctx.beginPath();
					ctx.rect(x + w - sz - 2*MENU_ITEM_BORDER + 2, y + 2*MENU_ITEM_BORDER + 2, sz - 4, sz - 4);
					ctx.fill();
				}
			}
		}

		// draw menu top border if first item
		if (i == 0) {
			ctx.beginPath();
			ctx.moveTo(x, y);
			ctx.lineTo(x + w, y);
			ctx.stroke();
		}

		// draw menu left border
		ctx.beginPath();
		ctx.moveTo(x, y);
		ctx.lineTo(x, y + h);
		ctx.stroke();

		// draw menu right border
		ctx.beginPath();
		ctx.moveTo(x + w, y);
		ctx.lineTo(x + w, y + h);
		ctx.stroke();

		// draw menu bottom border if last item
		if (i == this.item.length  - 1) {
			ctx.beginPath();
			ctx.moveTo(x, y + h);
			ctx.lineTo(x + w, y + h);
			ctx.stroke();
		}

		ctx.restore();

	}

	//
	// Menu.draw
	//
	Menu.prototype.draw = function(flags) {
		for (let i = 0; i < this.item.length; i++)
			this.drawItem(i);
	}

	//
	// Menu.grab
	//
	// handle grabbed events
	//
	Menu.prototype.grab = function(e) {

		//console.log("grab e.type=" + e.type + " e.button=" + e.button);

		mouseX = e.offsetX;				// remember co-ordinates of last mouse event
		mouseY = e.offsetY;				//
		let x0 = this.tx;
		let y0 = this.ty;
		let x1 = x0 + this.w;
		let y1 = y0 + this.h;

		// test if mouse in menu
		let hit = (mouseX > x0) && (mouseX < x1) && (mouseY > y0) && (mouseY < y1);

		// mouse NOT in menu
		if (hit == 0) {
			if ((e.type == "mousemove") && (this.selected != -1)) {
				let wasSelected = this.selected;
				this.selected = -1;
				this.drawItem(wasSelected);
				clearTimeout(this.timer);
			} else if (e.type == "mouseup") {
				if (this.state == 1)
					this.hide()
				this.state = 1;
			}
			e.preventDefault();
			return;
		}

		// find menu item and update display
		// if submenu item, start timer to display submenu
		let h = this.font.sz + 2*MENU_ITEM_BORDER;
		for (let i = 0; i < this.item.length; i++) {
			if (this.item[i].txt.length) {
				if (mouseY > y0 && mouseY < y0 + h) {
					if (this.selected != i) {
						clearTimeout(this.timer);
						let wasSelected = this.selected;
						this.selected = i;
						if (wasSelected != -1)
							this.drawItem(wasSelected);
						this.drawItem(i);
						if ((this.item[i].flags & MENU_ITEM_TYPE_MASK) == MENU_SUBMENU)
							this.timer = setTimeout(showSubMenu, 500, this, i);
						break;
					}
				}
				y0 += h;
			} else {
				y0 += MENU_SEPARATOR_H;
			}
		}

		// handle left mouse button up event (menu action)
		if ((e.type == "mouseup") && (e.button == 0) && ((e.timeStamp - this.timeStamp) > 200)) {
			if (this.selected != -1) {
				let item = this.item[this.selected];
				if ((item.flags & MENU_ITEM_DISABLED) == 0) {
					if ((item.flags & MENU_ITEM_TYPE_MASK) != MENU_SUBMENU) {
						if ((item.flags & MENU_NOHIDE) == 0)		// {joj 22/10/20}
							this.hide();
						if (this.actionf) {
							fork(this.actionf, this, item.action);	// NB fork
						} else {
							console.log("Menu.grab returning selected=" + this.selected);
							this.thread.acc = this.selected;		// {joj 09/06/21}
							$execute(this.thread);
						}
					}
				}
			}
		}
		e.preventDefault();
	}

	//
	// makeContextMenu
	//
	function makeContextMenu() {
		contextMenu = new Menu(WHITE, GRAY192, 0, contextMenuAction)
		contextMenu.addItem(showStats ? "hide runtime stats" : "show runtime stats", 0)
		contextMenu.addItem(showMbbs ? "hide mbbs" : "show mbbs", 1)
		contextMenu.addSeparator()
		contextMenu.addItem("reset", 2)
	}

	//
	// contextMenuAction
	//
	function contextMenuAction(action, x, y) {
		//console.log("contextMenuAction action=" + action);
		switch (action) {
		case 0:
			toggleShowStats(1)	// {joj 12/10/21}
			break
		case 1:
			toggleShowMbbs(1)	// {joj 12/10/21}
			break
		case 2:
			grab = 0			// {joj 26/09/20}
			reset()
			break
		default:
			break
		}
	}

	// sprintf -------------------------------- sprintf -------------------------------- sprintf

	//
	// sprintf
	//
	// Copyright (C) 2010 Jakob Westhoff
	//
	function sprintf(format) {

		if (typeof format != 'string')
			throw "sprintf: The first arguments need to be a valid format string.";

		//
		// Define the regex to match a formating string
		// The regex consists of the following parts:
		// percent sign to indicate the start
		// (optional) sign specifier
		// (optional) padding specifier
		// (optional) alignment specifier
		// (optional) width specifier
		// (optional) precision specifier
		// type specifier:
		//  % - literal percent sign
		//  b - binary number
		//  c - ASCII character represented by the given value
		//  d - signed decimal number
		//  f - floating point value
		//  o - octal number
		//  s - string
		//  x - hexadecimal number (lowercase characters)
		//  X - hexadecimal number (uppercase characters)
		//
		var r = new RegExp( /%(\+)?([0 ]|'(.))?(-)?([0-9]+)?(\.([0-9]+))?([%bcdfosxX])/g );

		//
		// Each format string is splitted into the following parts:
		// 0: Full format string
		// 1: sign specifier (+)
		// 2: padding specifier (0/<space>/'<any char>)
		// 3: if the padding character starts with a ' this will be the real
		//    padding character
		// 4: alignment specifier
		// 5: width specifier
		// 6: precision specifier including the dot
		// 7: precision specifier without the dot
		// 8: type specifier
		//
		var parts = [];
		var paramIndex = 1;
		var part; // {joj}
		while (part = r.exec(format)) {

			// Check if an input value has been provided, for the current
			// format string (no argument needed for %%)
			if (( paramIndex >= arguments.length) && (part[8] != '%'))
				throw "sprintf: At least one argument was missing.";

			parts[parts.length] = {
				begin: part.index,													// beginning of the part in the string
				end: part.index + part[0].length,									// end of the part in the string
				sign: (part[1] == '+'),												// force sign
				negative: (parseFloat(arguments[paramIndex]) < 0) ? true : false,	// is the given data negative
				padding: (part[2] == undefined)										// padding character (default: <space>)
					? (' ')															// default
					: (( part[2].substring(0, 1) == "'")
					? (part[3])														// use special char
					: (part[2])														// use normal <space> or zero
				),
				alignLeft: (part[4] == '-'),										// should the output be aligned left?
				width: ( part[5] != undefined ) ? part[5] : false,					// width specifier (number or false)
				precision: ( part[7] != undefined ) ? part[7] : false,				// precision specifier (number or false)
				type: part[8],														// type specifier
				data: (part[8] != '%') ? String(arguments[paramIndex++]) : false	// the given data associated with this part converted to a string
			}
		}

		var newString = "";
		var start = 0;
		for (var i = 0; i < parts.length; ++i) {									// generate our new formated string
			newString += format.substring(start, parts[i].begin);					// add first unformated string part
			start = parts[i].end;													// mark the new string start

			//
			// create the appropriate preformat substitution
			// this substitution is only the correct type conversion. All the
			// different options and flags haven't been applied to it at this
			// point
			//
			var preSubstitution = "";
			switch (parts[i].type) {
			case '%':
				preSubstitution = "%";
				break;
			case 'b':
				parts[i].data = parts[i].data == "false"  ? "0" : parts[i].data == "true" ? "1" : parts[i].data;	// {joj 24/09/20}
				preSubstitution = Math.abs(parseInt(parts[i].data)).toString(2);
				break;
			case 'c':
				preSubstitution = String.fromCharCode(Math.abs(parseInt(parts[i].data)));
				break;
			case 'd':
				parts[i].data = parts[i].data == "false"  ? "0" : parts[i].data == "true" ? "1" : parts[i].data;	// {joj 24/09/20}
				preSubstitution = String(Math.abs(parseInt(parts[i].data)));
				break;
			case 'f':
				parts[i].data = parts[i].data == "false"  ? "0" : parts[i].data == "true" ? "1" : parts[i].data;	// {joj 24/09/20}
				preSubstitution = (parts[i].precision === false)
					? (String((Math.abs(parseFloat( parts[i].data)))))
					: (Math.abs(parseFloat(parts[i].data)).toFixed(parts[i].precision));
				break;
			case 'o':
				parts[i].data = parts[i].data == "false"  ? "0" : parts[i].data == "true" ? "1" : parts[i].data;	// {joj 24/09/20}
				preSubstitution = Math.abs(parseInt(parts[i].data)).toString(8);
				break;
			case 's':
				preSubstitution = parts[i].data.substring(0, parts[i].precision ? parts[i].precision : parts[i].data.length);	// Cut if precision is defined
				break;
			case 'x':
				parts[i].data = parts[i].data == "false"  ? "0" : parts[i].data == "true" ? "1" : parts[i].data;	// {joj 24/09/20}
				preSubstitution = Math.abs(parseInt(parts[i].data)).toString(16).toLowerCase();
				break;
			case 'X':
				parts[i].data = parts[i].data == "false"  ? "0" : parts[i].data == "true" ? "1" : parts[i].data;	// {joj 24/09/20}
				preSubstitution = Math.abs(parseInt(parts[i].data)).toString(16).toUpperCase();
				break;
			default:
				throw 'sprintf: Unknown type "' + parts[i].type + '" detected. This should never happen. Maybe the regex is wrong.';
			}

			//
			// % character is a special type and does not need further processing
			//
			if (parts[i].type ==  "%") {
				newString += preSubstitution;
				continue;
			}

			//
			// modify the preSubstitution by taking sign, padding and width into account
			// pad the string based on the given width
			//
			//if (parts[i].width != false) {
			if (parts[i].width) {
				if (parts[i].width > preSubstitution.length) {				// padding needed?
					var origLength = preSubstitution.length;
					for (var j = 0; j < parts[i].width - origLength; ++j)
						//preSubstitution = (parts[i].alignLeft == true) ? (preSubstitution + parts[i].padding) : (parts[i].padding + preSubstitution);
						preSubstitution = (parts[i].alignLeft) ? (preSubstitution + parts[i].padding) : (parts[i].padding + preSubstitution);
				}
			}

			//
			// add a sign symbol if neccessary or enforced, but only if we are not handling a string
			//
			if (parts[i].type == 'b' || parts[i].type == 'd' || parts[i].type == 'o' || parts[i].type == 'f' || parts[i].type == 'x' || parts[i].type == 'X') {
				//if (parts[i].negative == true) {
				if (parts[i].negative) {
					preSubstitution = "-" + preSubstitution;
				//} else if (parts[i].sign == true ) {
				} else if (parts[i].sign) {
					preSubstitution = "+" + preSubstitution;
				}
			}
			newString += preSubstitution;							// add the substitution to the new string

		}

		newString += format.substring(start, format.length);		// add the remaining part of format string
		return newString;
	}

	// sprintf -------------------------------- sprintf -------------------------------- sprintf

	//
	// clock
	//
	function clock() {
		return Date.now();
	}

	//
	// setTPS
	//
	function setTPS(newTPS, fromUI = 0) {
		tps = newTPS
		if (fromUI && handler["eventSetTPS"]) {
			for (let i = 0; i < handler["eventSetTPS"].length; i++) {
				asyncPhaseUpdate = 1;
				callEventHandler(handler["eventSetTPS"][i].pc, handler["eventSetTPS"][i].obj, tps);
			}
			if (asyncPhaseUpdate)
				drawChanges();		// clears asyncPhaseUpdate {joj 28/11/17}
		}
		if (timer) {
			startTick = tick;
			startTimeStamp = performance.now();
		}
	}

	//
	// start
	//
	// fromUI == 1 if called directly from a mouse click or key press
	// return 1 if timer already started {joj 24/11/20}
	//
	function start(fromUI = 0) {
		//console.log("start fromUI=" + fromUI + " dir=" + dir + " timer=" + timer);
		let r = (timer) ? 1 : 0;				// {joj 24/11/20}
		if (fromUI == 0 && dir == -1)
			return r;
		if (fromUI == 0 && (playMode == SINGLESTEP || playMode == SNAPTOCHKPT))
			return r;
		if (timer == 0) {
			if (handler["eventStartStop"]) {	// {joj 04/01/21}
				rtStart = performance.now();
				for (let i = 0; i < handler["eventStartStop"].length; i++)
					callEventHandler(handler["eventStartStop"][i].pc, handler["eventStartStop"][i].obj, 1);
				if (fromUI) {					// {joj 04/01/21}
					asyncPhaseUpdate = 1;		// {joj 04/01/21}
					drawChanges();				// {joj 28/11/17} clears asyncPhaseUpdate
				}
			}
			if (tick == 0)
				dir = 1;
			startTick = tick;
			startTimeStamp = etStart = performance.now();
			timer = requestAnimationFrame(animate);
		}
		return r;								// {joj 24/11/20}
	}

	//
	// stop
	//
	// fromUI == 1 if called directly from a mouse click on canvas background
	// return 1 already stopped {joj 24/11/20}
	//
	function stop(fromUI = 0) {
		//console.log("stop(fromUI=" + fromUI + ") timer=" + timer);
		let r = (timer) ? 0 : 1;	// {joj 24/11/20}
		if (timer) {
			cancelAnimationFrame(timer);
			if (handler["eventStartStop"]) {	// {joj 04/01/21}
				rtStart = performance.now();
				for (let i = 0; i < handler["eventStartStop"].length; i++)
					callEventHandler(handler["eventStartStop"][i].pc, handler["eventStartStop"][i].obj, 0);
				if (fromUI) {					// {joj 04/01/21}
					asyncPhaseUpdate = 1;		// {joj 04/01/21}
					drawChanges();				// {joj 28/11/17} clears asyncPhaseUpdate
				}
			}
			timer = 0;							// set here so et reported correctly in stats
			if (dir == 1) {
				etf += performance.now() - etStart;
			} else {
				etb += performance.now() - etStart;
			}
		}
		return r;								// {joj 24/11/20}
	}

	//
	// $addGlobalEventHandler
	//
	function $addGlobalEventHandler(e, obj, pc) {
		//console.log("$addGlobalEventHandler(" + e + ", " + obj + ", " + pc + ")");
		if (handler[e] == undefined)
			handler[e] = [];
		handler[e].push({"obj":obj, "pc":pc});
	}

	//
	// getArgs
	//
	// args passed as a string parameter to VPlayer
	// args is a space separated list of name=value pairs
	//
	// eg. arg0=0 arg1=1 arg2="2 3" arg3='3 4 5'
	//
	// values are treated as strings and can optionally be
	// in single or double quotes
	//
	// NB: can also set arguments directly using JavaScript
	//
	// vplayer.arg["arg0"] = "0";
	// vplayer.arg["arg1"] = "1";
	// vplayer.arg["arg2"] = "2 3";
	// vplayer.arg["arg3"] = "3 4 5";
	//
	function getArgs() {
		try { // catch invalid range
			//console.log(args);
			let i = 0, len = args.length;
			//console.log(len);
			while (1) {
				let argName = "";
				if ((args[i] >= "A" && args[i] <= "Z") || (args[i] >= "a" && args[i] <= "z"))
					argName += args[i++];
				while ((args[i] >= "A" && args[i] <= "Z") || (args[i] >= "a" && args[i] <= "z") || (args[i] >= "0" && args[i] <= "9"))
					argName += args[i++];
				if (args[i] != "=")
					break;
				++i;
				let end = " ";
				if ((args[i] == "'") || (args[i] == '"'))
					end = args[i++];
				let argValue = "";
				while (i < len && args[i] != end)
					argValue += args[i++];
				if (i == len && end != " " && args[i-1] != end)	// check non matching quote to end string
					break;

				argName = "vivioArg[" + argName + "]";	// {joj 22/10/18}
				arg[argName] = argValue;

				i++; // skip space ' or "
				while (args[i] == " ")
					i++;
			}
		} finally {
			// ignore exception
		}
		//for (var a in arg)
		//	console.log("arg[" + a + "]=" + arg[a]);
	}

	//
	// getArg {joj 05/10/20}
	//
	// access arg["vivioArg[" + name +"]"
	//
	// NB: array["fill"] returns the array fill function so wrap...
	//	   name as "vivioArg[name]" avoids such conflicts
	//
	function getArg(name, defaultValue, fromStore = -1) {
		name = "vivioArg[" + name + "]";
		if (fromStore = -1) {
			if (arg[name]) {
				return arg[name];
			} else if (sessionStorage.getItem(name)) {
				return sessionStorage.getItem(name);
			} else if (localStorage.getItem(name)) {
				return localStorage.getItem(name);
			} else {
				return defaultValue;
			}
		} else if (fromStore == 0) {
			return arg[name] | defaultValue;
		} else if (fromStore == 1) {
			return sessionStorage.getItem[name] | defaultValue;
		} else {
			return localStorage.getItem[name] | defaultValue;
		}
	}

	//
	// getArgAsNum {joj 05/10/20}
	//
	function getArgAsNum(name, defaultValue, fromStore) {
		return Number(getArg(name, defaultValue.toString(), fromStore));
	}

	//
	// setArg
	//
	// store	0	arg
	//			1	sessionStorage
	//			2	localStorage
	//
	function setArg(name, v, store = 0) {
		console.log("setArg name=" + name + " v=" + v + " store=" + store);
		name = "vivioArg[" + name + "]";
		if (store == 0) {
			arg[name] = v;
		} else if (store == 1) {
			sessionStorage.setItem(name, v);
		} else {
			localStorage.setItem(name, v);
		}
	}

	//
	// setArgFromNum
	//
	function setArgFromNum(name, v, store) {
		setArg(name, v.toString(), store);
	}

	//
	// T2D constructor
	//
	function T2D() {
		if (arguments.length == 1) {			// T2D(t2d)
			this.a = arguments[0].a;
			this.b = arguments[0].b;
			this.c = arguments[0].c;
			this.e = arguments[0].d;
			this.e = arguments[0].e;
			this.f = arguments[0].f;
		} else {
			this.a = 1;							// T2D()
			this.b = 0;
			this.c = 0;
			this.d = 1;
			this.e = 0;
			this.f = 0;
		}
	}

	//
	// getURL
	//
	function getURL(url) {
		let http = new XMLHttpRequest();
		http.open("HEAD", url, false);
		try {
			http.send();
			window.location.assign(url)
		} catch (error) {
		}
		let xxx = document.URL.replace("demos", "www");
		let v = xxx.lastIndexOf("/");
		xxx = xxx.substr(0, v) + "/" + url;
		window.location.assign(xxx)
	}

	//
	// T2D.setIdentity
	//
	T2D.prototype.setIdentity = function() {
		this.a = 1;
		this.b = 0;
		this.c = 0;
		this.d = 1;
		this.e = 0;
		this.f = 0;
	}

	//
	// T2D.set
	//
	T2D.prototype.set = function() {
		if (arguments.length == 1) {		// set(t2d)
			this.a = arguments[0].a;
			this.b = arguments[0].b;
			this.c = arguments[0].c;
			this.d = arguments[0].d;
			this.e = arguments[0].e;
			this.f = arguments[0].f;
		} else {
			this.a = arguments[0];			// set(a, b, c, d, e, f)
			this.b = arguments[1];
			this.c = arguments[2];
			this.d = arguments[3];
			this.e = arguments[4];
			this.f = arguments[5];
		}
	}

	//
	// T2D.multiplyBA
	//
	// transform A followed by B represented by matrix BA
	//
	T2D.prototype.multiplyBA = function() {
		let ba, bb, bc, bd, be, bf;
		if (arguments.length == 1) {		// multiplyBA(t2D)
			ba = arguments[0].a;
			bb = arguments[0].b;
			bc = arguments[0].c;
			bd = arguments[0].d;
			be = arguments[0].e;
			bf = arguments[0].f;
		} else {							// multiplyBA(ba, bb, bc, bd, be, bf)
			ba = arguments[0];
			bb = arguments[1];
			bc = arguments[2];
			bd = arguments[3];
			be = arguments[4];
			bf = arguments[5];
		}
		let aa = this.a;
		let ab = this.b;
		let ac = this.c;
		let ad = this.d;
		let ae = this.e;
		let af = this.f;
		this.a = ba*aa + bc*ab;
		this.b = bb*aa + bd*ab;
		this.c = ba*ac + bc*ad;
		this.d = bb*ac + bd*ad;
		this.e = ba*ae + bc*af + be;
		this.f = bb*ae + bd*af + bf;
	}

	//
	// T2D.multiplyAB
	//
	// transform B followed by A represented by matrix AB
	//
	T2D.prototype.multiplyAB = function() {
		let ba, bb, bc, bd, be, bf;
		if (arguments.length == 1) {		// multiplyBA(ba, bb, bc, bd, be, bf)
			ba = arguments[0].a;
			bb = arguments[0].b;
			bc = arguments[0].c;
			bd = arguments[0].d;
			be = arguments[0].e;
			bf = arguments[0].f;
		} else {							// multiplyBA(t2D)
			ba = arguments[0];
			bb = arguments[1];
			bc = arguments[2];
			bd = arguments[3];
			be = arguments[4];
			bf = arguments[5];
		}
		let aa = this.a;
		let ab = this.b;
		let ac = this.c;
		let ad = this.d;
		let ae = this.e;
		let af = this.f;
		this.a = aa*ba + ac*bb;
		this.b = ab*ba + ad*bb;
		this.c = aa*bc + ac*bd;
		this.d = ab*bc + ad*bd;
		this.e = aa*be + ac*bf + ae;
		this.f = ab*be + ad*bf + af;
	}

	//
	// T2D.rotate
	//
	T2D.prototype.rotate = function(angle) {
		let cos = Math.cos(angle * Math.PI / 180);
		let sin = Math.sin(angle * Math.PI / 180);
		this.multiplyBA(cos, sin, -sin, cos, 0, 0);
	}

	//
	// T2D.scale
	//
	T2D.prototype.scale = function(sx, sy) {
		this.multiplyBA(sx, 0, 0, sy, 0, 0);
	}

	//
	// T2D.translate
	//
	T2D.prototype.translate = function(tx, ty) {
		this.multiplyBA(1, 0, 0, 1, tx, ty);
	}

	//
	// T2D.transformMbb
	//
	T2D.prototype.transformMbb = function(mbb) {
		//console.log("transformMbb");
		let x = [mbb.x0, mbb.x1, mbb.x1, mbb.x0];
		let y = [mbb.y0, mbb.y0, mbb.y1, mbb.y1];
		let xx, yy, x0 = Number.MAX_VALUE, y0 = Number.MAX_VALUE, x1 = Number.MIN_VALUE, y1 = Number.MIN_VALUE;
		for (let i = 0; i < 4; i++) {
			xx = x[i]*this.a + y[i]*this.c + this.e;
			yy = x[i]*this.b + y[i]*this.d + this.f;
			if (xx < x0)
				x0 = xx;
			if (xx > x1)
				x1 = xx;
			if (yy < y0)
				y0 = yy
			if (yy > y1)
				y1 = yy;
		}
		mbb.set(x0, y0, x1, y1);
	}

	//
	// T2D.transformXY
	//
	//T2D.prototype.transformXY = function(x, y) {
	//	return new Pt(x*this.a + y*this.c + this.e, x*this.b + y*this.d + this.f);
	//}

	//
	// T2D.toString
	//
	T2D.prototype.toString = function() {
		return "a=" + this.a + " b=" + this.b + " c=" + this.c + " d=" + this.d + " e=" + this.e + " f=" + this.f;
	}

	//
	// checkpoint
	//
	function checkPoint() {
		//console.log("checkPoint() tick=" + tick);
		checkPt.push(tick);
		atCheckPoint = 1;
	}

	//
	// getPreviousCheckPt
	//
	function getPreviousCheckPt() {
		let l;
		while (1) {
			l = checkPt.length;
			if ((l == 0) || (checkPt[l - 1] < tick))
				break;
			checkPt.pop();
		}
		return (l > 0) ? checkPt.pop() : 0;
	}

	//
	// drawStats
	//
	// tick=0 tps=100 last frame=4.9ms (3.7ms) FORWARD t=41.3s nf=2480 avg=0.9ms (0.8ms) BACK t=41.3s nf=2466 avg4.5ms (0.7ms) sst=1024
	//
	function drawStats() {
		if (showStats) {
			const h = INFOFONTH + 4;
			let t, txt, w;
			ctx.save();

			// background
			ctx.fillStyle = "#f0f0f0";			// light gray
			ctx.fillRect(0, canvas.clientHeight - h, canvas.clientWidth, h);

			// top border
			ctx.beginPath();
			ctx.strokeStyle = "#a0a0a0";		// darker gray
			ctx.lineWidth = 0;
			ctx.moveTo(0, canvas.clientHeight - h);
			ctx.lineTo(canvas.clientWidth, canvas.clientHeight - h);
			ctx.stroke();

			// text
			ctx.font = INFOFONTH + "px Calibri";
			ctx.textBaseline = "middle";
			ctx.fillStyle = "#000000";		// black
			txt = canvas.clientWidth + "x" + canvas.clientHeight + " tick=" + tick + " tps=" + tps + " last frame=" + rtLast.toFixed(1) + "ms (" + dtLast.toFixed(1) + "ms)";
			ctx.fillText(txt, 5, canvas.clientHeight - h/2 + 1);
			w = ctx.measureText(txt).width;
			t = (dir == 1 && timer) ? etf + performance.now() - etStart : etf;
			txt = " FORWARD " +
				" t=" + (t / 1000).toFixed(1) + "s" +
				" nf=" + nff +
				" avg=" + (nff ? (rtf / nff).toFixed(1) + "ms (" + (dtf / nff).toFixed(1) + "ms)" : "0.0ms (0.0ms)");
			ctx.fillStyle = "#008000";		// green
			ctx.fillText(txt, 5 + w, canvas.clientHeight - h/2 + 1);
			w += ctx.measureText(txt).width;
			t = (dir == -1 && timer) ? etb + performance.now() - etStart : etb;
			txt = " BACK " +
				" t=" + (t / 1000).toFixed(1) + "s" +
				" nf=" + nfb +
				" avg=" + (nfb ? (rtb / nfb).toFixed(1) + "ms (" + (dtb / nfb).toFixed(1) + "ms)" : "0.0ms (0.0ms)") +
				" sst=" + sst;
			ctx.fillStyle = "#ff0000";		// red
			ctx.fillText(txt, 5 + w, canvas.clientHeight - h/2 + 1);

			ctx.restore();
		}
	}

	//
	// InfoTip constructor
	//
	// need to keep mbb of InfoTip
	//
	function InfoTip() {
		this.mbb = new Mbb();
		//this.mbbs = new Mbbs();
		infoTipTimer = 0;
	}

	//
	// InfoTip.draw
	//
	// draws infoTip on top layer
	//
	InfoTip.prototype.draw = function() {
		if (infoTipTimer) {
			let ctx = layer[layer.length - 1].ctx;
			ctx.save();
			ctx.font = INFOFONTH + "px Calibri";
			ctx.fillStyle = "#ffffe0";		// ivory
			ctx.fillRect(this.x, this.y, this.w, this.h);
			ctx.strokeStyle = "#ff0000";	// red
			ctx.strokeRect(this.x, this.y, this.w, this.h);
			ctx.textAlign = "left";
			ctx.textBaseline = "middle";
			ctx.fillStyle = "#000000";
			ctx.fillText(this.txt, this.x + 5, this.y + this.h/2);
			ctx.restore();
			this.mbb.set(this.x, this.y, this.x + this.w, this.y + this.h);
		}
	}

	//
	// InfoTip.drawChanges
	//
	// draw infoTip on top layer
	// layer[nlayer - 1].mmbbs.nmbb will be 0
	//
	InfoTip.prototype.drawChanges = function() {
		//console.assert(layer[nlayer-1].mbbs.nmbb == 0, "InfoTip.drawChanges nmbb=%d", layer[nlayer-1].mbbs.nmbb);
		if (this.mbb.isEmpty() == 0) {								// clear existing infoTip if being displayed
			let ctx = layer[layer.length - 1].ctx;					// top layer
			layer[layer.length - 1].mbbs.add(this.mbb);				// add infoTip mbb to top layer mbbs (will be empty)
			ctx.save();
			ctx.beginPath();
			//console.log("drawChanges x0=" + this.mbb.x0 + " y0=" + this.mbb.y0 + " x1=" + this.mbb.x1 + " y1=" + this.mbb.y1);
			ctx.rect(this.mbb.x0 - 1, this.mbb.y0 - 1, this.mbb.width() + 2, this.mbb.height() + 2);
			ctx.clip();
			ctx.fillStyle = bgBrush ? bgBrush.ctxFillStyle : "white";		// gobalAlpha?
			if (layer.length == 1) {
				ctx.fillStyle = bgBrush ? bgBrush.ctxFillStyle : "white";	// gobalAlpha?
				ctx.fillRect(0, 0, canvas.width, canvas.height);
			} else {
				ctx.clearRect(0, 0, canvas.width, canvas.height);
			}
			if (layer[layer.length - 1].opacity) {								// {joj 17/10/16}
				for (let i = 0; i < layer[layer.length - 1].gobjs.length; i++)	// redraw gobjs which overlap infoTip
					layer[layer.length - 1].gobjs[i].draw(0);
			}
			layer[layer.length - 1].mbbs.nmbb = 0;								// reset top layer mbbs
			ctx.restore();
		}
		this.draw();													// draw infoTip
	}

	//
	// InfoTip.set
	//
	InfoTip.prototype.set = function(txt, centred) {
		this.txt = txt
		this.centred = centred;
		this.mouseX = mouseX;
		this.mouseY = mouseY;
		let x, y, w, h = INFOFONTH + 4;
		ctx.save();
		ctx.font = INFOFONTH + "px Calibri";
		w = ctx.measureText(this.txt).width + 10;
		ctx.restore();
		if (this.centred) {
			x = (canvas.clientWidth - w) / 2;
			y = (canvas.clientHeight - h) / 2;
		} else {
			x = this.mouseX;
			if (x + w > canvas.clientWidth)
				x = canvas.clientWidth - w - 2;
			y = this.mouseY - h;
			if (y - h < 0)
				y = 0;
		}
		this.x = Math.round(x) + 0.5;
		this.y = Math.round(y) + 0.5;
		this.w = Math.round(w);
		this.h = Math.round(h);
	}

	//
	// InfoTip.drawInfoTip
	//
	function drawInfoTip() {
		if (infoTipTimer)
			infoTip.drawChanges();
	}

	//
	// InfoTip.hideInfoTip
	//
	function hideInfoTip() {
		infoTipTimer = 0;
		infoTip.drawChanges();
		infoTip.mbb.set(0, 0, 0, 0);
	}

	//
	// InfoTip.showInfoTip
	//
	function showInfoTip(txt, centred, draw) {
		infoTip.set(txt, centred);
		if (infoTipTimer)
			clearTimeout(infoTipTimer);
		infoTipTimer = setTimeout(hideInfoTip, 1000);
		if (draw)
			infoTip.drawChanges();
	}

	var infoTip = new InfoTip();

	//
	// reset
	//
	function reset() {
		console.log("reset");
		stop(0);
		etf = etb = nff = nfb = rtf = rtb = dtf = dtb = 0;
		rtStart = performance.now();
		goto(-1);
		//let e = new Event();		// is this the wrong event?
		//e.offsetX = mouseX;
		//e.offsetY = mouseY;
		//mouseMoveHandler(e);
		drawAll();
	}

	//
	// isRunning
	//
	function isRunning() {			// vivio function
		return timer > 0;
	}

	//
	// setSST
	//
	// save snapshot every newSST ticks
	//
	function setSST(newSST) {
		sst = newSST;
	}

	//
	// debug
	//
	function debug() {
		console.log(sprintf.apply(this, arguments));
	}

	//
	// $closeIDE
	//
	function $closeIDE() {
		if ($testFlag)
			window.close();
	}

	//
	// String.find
	//
	String.prototype.find = function(s) {
		return this.indexOf(s);
	}

	//
	// String.left
	//
	String.prototype.left = function(cnt) {
		return this.slice(0, cnt);
	}

	//
	// String.len
	//
	String.prototype.len = function() {
		return this.length;
	}

	//
	// String.mid
	//
	String.prototype.mid = function(startPos, cnt) {
		return this.slice(startPos, startPos + cnt);
	}

	//
	// String.rfind
	//
	String.prototype.rfind = function(s) {
		return this.lastIndexOf(s);
	}

	//
	// String.regExpSplit
	//
	String.prototype.regExpSplit = function(pattern, flags, limit) {
		return this.split(RegExp(pattern, flags), limit);
	}

	//
	// String.right
	//
	String.prototype.right = function(cnt) {
		return this.slice(-cnt);
	}

	//
	// String.toNum
	//
	String.prototype.toNum = function() {
		return Number(this);
	}

	//
	// Date.isLeapYear (code from stackoverflow)
	//
	Date.prototype.isLeapYear = function() {
    	var year = this.getFullYear();
    	if((year & 3) != 0) return false;
    	return ((year % 100) != 0 || (year % 400) == 0);
	};

	//
	// Date.getDOY (code from stackoverflow)
	//
	Date.prototype.getDOY = function() {
    	var dayCount = [0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334];
    	var mn = this.getMonth();
    	var dn = this.getDate();
    	var dayOfYear = dayCount[mn] + dn;
    	if(mn > 1 && this.isLeapYear()) dayOfYear++;
    	return dayOfYear;
	};

	//
	// Date.getWeek (ISO 8601 code from weekNumber.net)
	//
	Date.prototype.getWeek = function() {
  		var date = new Date(this.getTime());
   		date.setHours(0, 0, 0, 0);
  		// Thursday in current week decides the year
  		date.setDate(date.getDate() + 3 - (date.getDay() + 6) % 7);
		// January 4 is always in week 1
		var week1 = new Date(date.getFullYear(), 0, 4);
		// Adjust to Thursday in week 1 and count number of weeks from date to week1
  		return 1 + Math.round(((date.getTime() - week1.getTime()) / 86400000   - 3 + (week1.getDay() + 6) % 7) / 7);
	}

	// apologies for English only
	var day = ["Sun", "Mon", "Tues", "Wed", "Thurs", "Fri", "Sat"];
	var dayF = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
	var month = ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"];
	var monthF = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];

	//
	// getLastModifiedMS
	//
	function getLastModifiedMS() {
		return Date.parse(document.lastModified);
	}

	//
	// fromCodePoint {joj 27/03/18}
	//
	function fromCodePoint(code) {
		return String.fromCodePoint(code);
	}

	//
	// timeToString
	//
	function timeToString(t, fs) {
		t = new Date(t);
		if (fs === undefined)
			return t.toLocaleString();
		let str = "";
		let v = 0;
		for (let i = 0; i < fs.length; i++) {
			if ((fs[i] != '%') || (i + 1 == fs.length)) {
				str += fs[i];
				continue;
			}
			switch (fs[i + 1]) {
			case 'a':
				str += day[t.getDay()]; i++; break;
			case 'A':
				str += dayF[t.getDay()]; i++; break;
			case 'b':
			case 'h':
				str += month[t.getMonth()]; i++; break;
			case 'B':
				str += monthF[t.getMonth()]; i++; break;
			case 'd':
				v = t.getDate();
				str += (v < 10) ? "0" + v : v; i++; break;
			case 'e':
				v = t.getDate();
				str += (v < 10) ? " " + v : v; i++; break;
			case 'H':
				v = t.getHours();
				str += (v < 10) ? "0" + v : v; i++; break;
			case 'I ':
				v = t.getHours() % 12;
				str += (v < 10) ? "0" + v : v; i++; break;
			case 'j':
				v = t.getDOY();
				str += (v < 10) ? "00" + v : (v < 100) ? "0" + v : v; i++; break;
			case 'k':
				v = t.getHours();
				str += (v < 10) ? " " + v : v; i++; break;
			case 'l':
				v = t.getHours() % 12;
				str += (v < 10) ? " " + v : v; i++; break;
			case 'm':
				v = t.getMonth() + 1;
				str += (v < 10) ? "0" + v : v; i++; break;
			case 'M':
				v = t.getMinutes();
				str += (v < 10) ? "0" + v : v; i++; break;
			case 'p':
				v = t.getHours() % 12;
				str += (v < 12) ? "PM" : "AM"; i++; break;
			case 'P':
				v = t.getHours() % 12;
				str += (v < 12) ? "pm" : "am"; i++; break;
			case 'r':
				v = t.getHours() % 12;
				str += (v < 10) ? "0" + v : v;
				str += ":";
				v = t.getMinutes();
				str += (v < 10) ? "0" + v : v;
				str += ":";
				v = t.getSeconds();
				str += (v < 10) ? "0" + v : v;
				v = t.getHours() % 12;
				str += (v < 12) ? " PM" : " AM";
				i++;
				break;
			case 'R':
				v = t.getHours();
				str += (v < 10) ? "0" + v : v;
				str += ":";
				v = t.getMinutes();
				str += (v < 10) ? "0" + v : v;
				i++;
				break;
			case 'S':
				v = t.getSeconds();
				str += (v < 10) ? "0" + v : v; i++; break;
			case 'T':
				v = t.getHours();
				str += (v < 10) ? "0" + v : v;
				str += ":";
				v = t.getMinutes();
				str += (v < 10) ? "0" + v : v;
				str += ":";
				v = t.getSeconds();
				str += (v < 10) ? "0" + v : v;
				i++;
				break;
			case "u":
				v = t.getDay();
				str += (v == 0) ? 7 : v;
			case 'V':
				v = t.getWeek();
				str += (v < 10) ? "0" + v : v; i++; break;
			case "w":
				str += t.getDay(); i++; break;
			case 'y':
				v = t.getFullYear() % 100;
				str += (v < 10) ? "0" + v : v; i++; break;
			case 'Y':
				str += t.getFullYear(); i++; break;
			default:
				str += fs[i];
			}
		}
		return str;
	}

	//
	// VPlayer (main)
	//
	// constants (in alphabetic order)
	//
	this.ABSOLUTE = ABSOLUTE;
	this.ARROW40_END = ARROW40_END;
	this.ARROW60_END = ARROW60_END;
	this.ARROW90_END = ARROW90_END;
	this.ARROW40_START = ARROW40_START;
	this.ARROW60_START = ARROW60_START;
	this.ARROW90_START = ARROW90_START;
	this.BEVEL_JOIN = BEVEL_JOIN;
	this.BLACK = BLACK;
	this.BLUE = BLUE;
	this.BOLD = BOLD;
	this.BUTT_END = BUTT_END;
	this.BUTT_START = BUTT_START;
	this.CLOSED = CLOSED;
	this.CIRCLE_END = CIRCLE_END;
	this.CIRCLE_START = CIRCLE_START;
	this.CYAN = CYAN;
	this.DASH = DASH;
	this.DASH_DOT = DASH_DOT;
	this.DASH_DOT_DOT = DASH_DOT_DOT;
	this.DOT = DOT;
	this.EXTENDEDGOBJ = EXTENDEDGOBJ;		// {joj 18/10/20}
	this.GRADIENTBRUSH = GRADIENTBRUSH;
	this.GRAY32 = GRAY32;
	this.GRAY64 = GRAY64;
	this.GRAY96	= GRAY96;
	this.GRAY128 = GRAY128;
	this.GRAY160 = GRAY160;
	this.GRAY192 = GRAY192;
	this.GRAY224 = GRAY224;
	this.GREEN = GREEN;
	this.HCENTRE = HCENTRE;
	this.HITINVISIBLE = HITINVISIBLE;
	this.HITWINDOW = HITWINDOW;
	this.HLEFT = HLEFT;
	this.HMASK = HMASK;
	this.HRIGHT = HRIGHT;
	this.IMAGEBRUSH = IMAGEBRUSH;
	this.IMAGEPEN = IMAGEPEN;
	this.ITALIC = ITALIC;
	this.KEY = KEY;
	this.MAGENTA = MAGENTA;
	this.MAX_CAP = MAX_CAP;
	this.MAX_JOIN = MAX_JOIN;
	this.MAX_SAFE_INTEGER = Number.MAX_SAFE_INTEGER;	// {joj 20/11/20}
	this.MAX_STYLE = MAX_STYLE;
	this.MB_ALT = MB_ALT;
	this.MB_CTRL = MB_CTRL;
	this.MB_LEFT = MB_LEFT;
	this.MB_MIDDLE = MB_MIDDLE;
	this.MB_RIGHT = MB_RIGHT;
	this.MB_SHIFT = MB_SHIFT;
	this.MM = MM;
	this.MB = MB;
	this.MENU_ITEM_CHECKBOX = MENU_ITEM_CHECKBOX;		// {joj 28/09/20}
	this.MENU_ITEM_DISABLED = MENU_ITEM_DISABLED;		// {joj 28/09/20}
	this.MENU_NOHIDE = MENU_NOHIDE;						// {joj 22/10/20}
	this.NOATTACH = NOATTACH;							// {joj 03/11/17}
	this.MIN_SAFE_INTEGER = Number.MIN_SAFE_INTEGER;	// {joj 20/11/20}
	this.NULLBRUSH = NULLBRUSH;
	this.NULLPEN = NULLPEN;
	this.PROPAGATE = PROPAGATE;
	this.RADIALBRUSH = RADIALBRUSH;
	this.RED = RED;
	this.REMEMBER = REMEMBER;
	this.ROUND_END = ROUND_END;
	this.ROUND_JOIN = ROUND_JOIN;
	this.ROUND_START = ROUND_START;
	this.RT = RT;							// {joj 06/07/17}
	this.SMALLCAPS = SMALLCAPS;
	this.SOLID = SOLID;
	this.SOLIDBRUSH = SOLIDBRUSH;			// {joj 13/09/17}
	this.SOLIDPEN = SOLIDPEN;				// {joj 13/09/17}
	this.SQUARE_END = SQUARE_END;
	this.SQUARE_START = SQUARE_START;
	this.STRIKETHROUGH = STRIKETHROUGH;
	this.UNDERLINE = UNDERLINE;
	this.VBOTTOM = VBOTTOM;
	this.VCENTRE = VCENTRE;
	this.VERSION = VERSION;
	this.VTOP = VTOP;
	this.VMASK = VMASK;
	this.WHITE = WHITE;
	this.YELLOW = YELLOW;

	//
	// globals (in alphabetic order)
	//
	// names start with $ so they don't clash with Vivio variable names
	//
	this.$g = $g;

	// functions (in alphabetic order)

	this.abs = Math.abs;
	this.acos = function(x) {return Math.acos(x) * 180 / Math.PI};
	this.$addGlobalEventHandler = $addGlobalEventHandler;
	this.$addWaitToEventQ = $addWaitToEventQ;
	this.Arc = Arc;
	this.arg = arg;
	this.asin = function(x) {return Math.asin(x) * 180 / Math.PI};
	this.atan = function(x) {return Math.atan(x) * 180 / Math.PI};
	this.atan2 = function(y, x) {return Math.atan2(y, x) * 180 / Math.PI};
	this.Bezier = Bezier;
	this.ceil = Math.ceil;
	this.checkPoint = checkPoint;
	this.clock = clock;
	this.$closeIDE = $closeIDE;
	this.cos = function(d) {return Math.cos(d * Math.PI/180)};
	this.debug = debug;
	this.endInterrupt = endInterrupt;
	this.exp = Math.exp;
	this.E$ = E$;
	this.getArg = getArg;
	this.getArgAsNum = getArgAsNum;
	this.getCanvasH = function() {return canvas.clientHeight};		// {joj 17/09/20}
	this.getCanvasW = function() {return canvas.clientWidth};		// {joj 17/09/20}
	this.getDefaultFont = function() {return defaultFont};			// {joj 26/09/20}
	this.getDefaultPen = function() {return defaultPen};			// {joj 26/09/20}
	this.getURL = getURL;											// {joj 14/07/21}
	this.getTick = function() {return tick};
	this.getTPS = function() {return tps};
	this.getCanvasVX = function() {return -tx/sx};					// {joj 17/09/20}
	this.getCanvasVY = function() {return -ty/sy};					// {joj 17/09/20}
	this.getCanvasVH = function() {return canvas.clientHeight/sy};	// {joj 17/09/20}
	this.getCanvasVW = function() {return canvas.clientWidth/sx};	// {joj 17/09/20}
	this.getUIState = getUIState;									// {joj 07/10/21}
	this.gotoTick = gotoTick;										// {joj 13/10/20}
	this.Ellipse = Ellipse;
	this.Ellipse2 = Ellipse2;
	this.fire = fire;
	this.floor = Math.floor;
	this.Font = Font;
	this.fork = fork;
	this.fromCodePoint = fromCodePoint;								// {joj 27/03/18}
	this.getLastModifiedMS = getLastModifiedMS;						// {joj 16/09/20}
	this.getLayers = getLayers;										// {joj 08/10/21}
	this.GradientBrush = GradientBrush;
	this.Group = Group;
	this.Image = Image;
	this.ImageBrush = ImageBrush;
	this.ImagePen = ImagePen;
	this.isRunning = isRunning;
	this.L$ = L$;
	this.Layer = Layer;
	this.Line = Line;
	this.Line2 = Line2;
	this.load = load;
	this.log = Math.log;
	this.log10 = Math.log10;
	this.mkTime = function(y, m, d, h, min, s) {return new Date(y, m, d, h, min, s).getTime()};
	this.Menu = Menu;
	this.newArray = newArray;
	this.NullBrush = NullBrush;
	this.NullPen = NullPen;
	this.Pie = Pie;
	this.Polygon = Polygon;
	this.pow = Math.pow;
	this.R$ = R$;
	this.RadialBrush = RadialBrush;
	this.random = function() {return Math.random()};
	this.Rectangle = Rectangle;
	this.Rectangle2 = Rectangle2;
	this.reset = reset;
	this.rgba = rgba;
	this.round = Math.round;
	this.setArg = setArg;
	this.setArgFromNum = setArgFromNum;
	this.setBgBrush = setBgBrush;
	this.setBgPen = setBgPen;
	this.setLayerVisibility = setLayerVisibility;	// {joj 11/10/21}
	this.setShowMbbs = setShowMbbs;					// {joj 11/10/21}
	this.setShowStats = setShowStats;				// {joj 11/10/21}
	this.setSST = setSST;
	this.setTPS = setTPS;
	this.setVirtualWindow = setVirtualWindow; 		// {joj 31/01/19}
	this.sin = function(d) {return Math.sin(d * Math.PI/180)};
	this.sizeOf = function(a) {return a.length};
	this.SolidBrush = SolidBrush;
	this.SolidPen = SolidPen;
	this.Spline = Spline;
	this.sprintf = sprintf;
	this.sqrt = Math.sqrt;
	this.start = start;
	this.startInterrupt = startInterrupt;
	this.stop = stop;
	this.tan = function(d) {return Math.tan(d * Math.PI/180)};
	this.$terminateThread = $terminateThread;
	this.timeMS = function() {return Date.now()};
	this.timeToString = timeToString;
	this.trunc = Math.trunc;
	this.Txt = Txt;
	this.VObj = VObj;

	//
	// VCode
	//
	var vcode = new VCode(this);
	var $getCurrentThread = vcode.$getCurrentThread;
	var $execute = vcode.$execute;
	var $resumeThread = vcode.$resumeThread;
	var $suspendThread = vcode.$suspendThread;
	var $switchToThread = vcode.$switchToThread;		// {joj 02/01/21}
	var $testFlag = vcode.$testFlag;					// {joj 29/10/21}

	//
	// wait for fonts to load
	//
	document.fonts.ready.then(run);						// {joj 08/05/21}

}

// eof
