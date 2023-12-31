## esp32 extension

#### esp32 extension is using [Scratch VM](https://github.com/scratchfoundation/scratch-vm), [Scratch Gui](https://github.com/scratchfoundation/scratch-gui), [esp-idf](https://github.com/espressif/esp-idf/tree/master/examples/bluetooth/nimble/bleprph)

[![Build Status](https://travis-ci.org/LLK/scratch-vm.svg?branch=develop)](https://travis-ci.org/LLK/scratch-vm)
[![Coverage Status](https://coveralls.io/repos/github/LLK/scratch-vm/badge.svg?branch=develop)](https://coveralls.io/github/LLK/scratch-vm?branch=develop)

##Intro

![스크린샷 2023-11-10 오후 4 04 57](https://github.com/2023-Education-Board-Project/.github/assets/69339846/b8b04a4b-1d47-40ab-bb7b-3f239693cfa9)

This project connects the esp32 board and scratch, provides a block to create rtos tasks.

## Instructions

### ESP32 side

Instructions are divided into an opcode part and a data part.
COMMAND BLOCK makes task to run function.

```c
void	mapping_block_task(uint8_t *func)
{
	switch (func[0])
	{
		case 0x81://opcode
			task_init(&(func[1]));
			break;
		default:
			;
	}
}

static void task_init(uint8_t *data)
{
	uint8_t	*func;
	size_t	len;

	len = strlen((char *)data) + 1;
	
	func = heap_caps_malloc(sizeof(uint8_t) * len, MALLOC_CAP_8BIT);
	assert(func != 0);
	memset(func, 0, len);

	strcpy((char *)func, (char *)data);
	//less priority than ble notify
	xTaskCreate(task_behaviour, "task_behaviour", 4096, (void *)func, tskIDLE_PRIORITY - 1, NULL);
}
```

FUNCTION BLOCKs have the same structure.

```c
static void execute_func(uint8_t *data)
{
	switch (data[0])//opcode
	{
		case 0xC1://display
			display_func((char *)(&(data[1])));
			break;
		case 0xC2://loop
			loop_func(&(data[1]));
			break;
		case 0xC3://wait
			wait_func(&(data[1]));
			break;
		default:
			print_text_lcd("%X\n", data[0]);
	}
}
```

### Scratch side

![스크린샷 2023-11-10 오후 1 16 49](https://github.com/2023-Education-Board-Project/.github/assets/69339846/1994296f-6eed-45f1-b9c8-31385e3cee9a)

Command Block's parameter contains func variable

```javascript
/**
* @param {string} func - the func to run.
* @return {Promise} - a Promise that resolves when writing to peripheral.
*/
runTask (func) {
    const output = Base64Util.base64ToUint8Array(func);
    return this.send(BLECommand.CMD_TASK, output);
}
```

![스크린샷 2023-11-10 오후 1 15 04](https://github.com/2023-Education-Board-Project/.github/assets/69339846/133e95c9-ea90-4c12-aaf2-2c4bf23bd2bd)

Function Blocks must return string type

```javascript
/**
* @param {string} text - the text to display.
* @return {string} - a string contains func and data.
*/
displayText (text) {
    const output = new Uint8Array(text.length);
    for (let i = 0; i < text.length; i++) {
        output[i] = text.charCodeAt(i);
    }
    const cmd = this._funcPrefix(PramFunc.FUNC_DISPLAY_TEXT, output, null);
    return cmd;
}
```

![스크린샷 2023-11-10 오후 1 16 59](https://github.com/2023-Education-Board-Project/.github/assets/69339846/5406cd0a-6b56-447f-a33a-8263c26e27c9)

Function Control Block's parameter contains func variable 
```javascript
/**
* @param {number} loop - the text to display.
* @param {string} func - func script string
* @return {string} - a string contains func and data.
*/
loop (loop, func) {
    onst output = new Uint8Array(1);
    output[0] = loop;
    const cmd = this._funcPrefix(PramFunc.FUNC_LOOP, output, func);
    return cmd;
}
```

## Installation

### Scratch

This requires you to have Git and Node.js installed.

To install as a dependency for your own application:
```bash
npm install scratch-vm
```
To set up a development environment to edit scratch-vm yourself:
```bash
git clone https://github.com/LLK/scratch-vm.git
cd scratch-vm
npm install
```
To install as a dependency for your own application:
```bash
npm install scratch-gui
```
To set up a development environment to edit scratch-vm yourself:
```bash
git clone https://github.com/LLK/scratch-gui.git
cd scratch-gui
npm install
```

### ESP32 Board

install esp-idf by following [instruction](https://github.com/espressif/esp-idf/tree/master)

## Development Server
This requires Node.js to be installed.

For convenience, we've included a development server with the VM. This is sometimes useful when running in an environment that's loading remote resources (e.g., SVGs from the Scratch server). If you would like to use your modified VM with the full Scratch 3.0 GUI, [follow the instructions to link the VM to the GUI](https://github.com/LLK/scratch-gui/wiki/Getting-Started).

## Running the Development Server
Open a Command Prompt or Terminal in the repository and run:
```bash
npm start
```

## Testing
```bash
npm test
```

```bash
npm run coverage
```

## Publishing to GitHub Pages
```bash
npm run deploy
```
