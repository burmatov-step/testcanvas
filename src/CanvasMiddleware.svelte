<script>
	import { getContext } from "svelte";
	import { writable } from "svelte/store";
	const draw = getContext("draw");
	const canvas = getContext("canvas");
	const stateCanvas = getContext("stateCanvas");

	class Canvas {
		constructor(canvas) {
			this.canvas = canvas;
			this.context = canvas.getContext("2d");
		}
	}

	class StateCanvas {
		constructor(canvas) {
			this.canvas = canvas;
			this.context = canvas.getContext("2d");
			this.fillStyle = writable("#659b41");
			this.lineWidth = writable(2);
			this.undoList = [];
			this.redoList = [];
			this.nowImageCanvas = writable(
				this.context.getImageData(0, 0, canvas.width, canvas.height)
			);
		}

		setFillStyle(str) {
			this.fillStyle.set(str);
		}
		setLineWidth(num) {
			this.lineWidth.set(num);
		}

		pushUndo(obj){
			this.undoList.push(obj)
			this.nowImageCanvas.set(obj)
		}

		undo() {}
	}

	class CanvasEvents extends Canvas {
		constructor(canvas) {
			super(canvas);
			this.paint = false;
			this.clickX = writable(null);
			this.clickY = writable(null);
			this.restore_array = [];
			this.index = -1;
			this.type = writable(null);
		}

		addClick(x, y) {
			this.clickX.set(x);
			this.clickY.set(y);
		}

		mousedown(e) {
			var mouseX = e.pageX - e.target.offsetLeft;
			var mouseY = e.pageY - e.target.offsetTop;
			this.paint = true;
			this.addClick(mouseX, mouseY);
			this.type.set("mousedown");
		}

		mousemove(e) {
			if (this.paint) {
				var mouseX = e.pageX - e.target.offsetLeft;
				var mouseY = e.pageY - e.target.offsetTop;
				this.addClick(mouseX, mouseY, true);
				this.type.set("mousemove");
			}
		}
		mouseleave() {
			this.paint = false;
			this.type.set(null);
		}
		mouseup() {
			this.paint = false;
			this.type.set(null);
			console.log(this.context.getImageData(
					0,
					0,
					this.canvas.width,
					this.canvas.height
				))
			$stateCanvas.pushUndo(this.context.getImageData(
					0,
					0,
					this.canvas.width,
					this.canvas.height
				))
		}

		events() {
			this.canvas.onmousedown = this.mousedown.bind(this);
			this.canvas.onmousemove = this.mousemove.bind(this);
			this.canvas.onmouseup = this.mouseup.bind(this);
			this.canvas.mouseleave = this.mouseleave.bind(this);
		}
	}

	class CanvasActions extends Canvas {
		constructor(canvas) {
			super(canvas);
		}

		drawing(x, y, type) {
			if (type == "mousedown") {
				this.context.putImageData($nowImageCanvas, 0, 0);
				this.context.beginPath();
				this.context.lineTo(x, y);
				this.context.stroke();
			}
			if (type == "mousemove") {
				this.context.lineTo(x, y);
				this.context.stroke();
			}
		}
	}

	class Draw {
		constructor(canvas) {
			this.canvas = canvas;
			this.context = canvas.getContext("2d");
			this.paint = false;
			this.clickX = writable(null);
			this.clickY = writable(null);
			this.clickDrag = [];
			this.restore_array = [];
			this.index = -1;
			this.type = writable("");
		}

		addClick(x, y, dragging) {
			this.clickX.set(x);
			this.clickY.set(y);
		}

		clear_canvas() {
			this.context.clearRect(
				0,
				0,
				this.context.canvas.width,
				this.context.canvas.height
			);
			this.context.fillRect(
				0,
				0,
				this.context.canvas.width,
				this.context.canvas.height
			);
			this.restore_array = [];
			this.index = -1;
		}
		undo_last() {
			if (this.index <= 0) {
				this.clear_canvas();
			} else {
				this.index -= 1;
				this.restore_array.pop();
				this.context.putImageData(this.restore_array[this.index], 0, 0);
			}
		}

		mousedown(e) {
			var mouseX = e.pageX - e.target.offsetLeft;
			var mouseY = e.pageY - e.target.offsetTop;
			this.paint = true;
			console.log(this);
			this.addClick(mouseX, mouseY);
			this.type.set("mousedown");
		}

		mousemove(e) {
			if (this.paint) {
				var mouseX = e.pageX - e.target.offsetLeft;
				var mouseY = e.pageY - e.target.offsetTop;
				this.addClick(mouseX, mouseY, true);
				this.type.set("mousemove");
			}
		}
		mouseleave(e) {
			this.paint = false;
			this.type.set("");
		}
		mouseup(e) {
			this.paint = false;
			this.type.set("");

			this.restore_array.push(
				this.context.getImageData(
					0,
					0,
					this.canvas.width,
					this.canvas.height
				)
			);
			if (this._restore_array$ !== undefined) {
				this._restore_array$.set(this.restore_array);
			}
			this.index++;
		}

		drawing(x, y, type) {
			if (type == "mousedown") {
				this.context.beginPath();
				this.context.lineTo(x, y);
				this.context.stroke();
			}
			if (type == "mousemove") {
				this.context.lineTo(x, y);
				this.context.stroke();
			}
		}

		events() {
			this.canvas.onmousedown = this.mousedown.bind(this);
			this.canvas.onmousemove = this.mousemove.bind(this);
			this.canvas.onmouseup = this.mouseup.bind(this);
		}
	}
	const eventCanvas = new CanvasEvents(canvas.canvas);
	const action = new CanvasActions(canvas.canvas);
	$stateCanvas = new StateCanvas(canvas.canvas);
	eventCanvas.events();
	const x = eventCanvas.clickX;
	const y = eventCanvas.clickY;
	const type = eventCanvas.type;
	const nowImageCanvas = $stateCanvas.nowImageCanvas;
	// $draw = new Draw(canvas.canvas);
	// $draw.events();
	// const x = $draw.clickX;
	// const y = $draw.clickY;
	// const type = $draw.type;
	$: action.drawing($x, $y, $type), $x, $y;
	// $: if ($draw) $draw.drawing($x, $y, $type), $x, $y;
</script>
