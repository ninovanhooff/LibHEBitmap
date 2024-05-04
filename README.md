<picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark-512.png">
    <img src="assets/logo-light-512.png" width="400" alt="HEBitmap logo">
</picture>

## LibHEBitmap
This is just a slight modification of the [original HEBitmap](https://github.com/risolvipro/HEBitmap) build process that will produce static library (libhebitmap.a) files instead of a full game. If you are programming in C, you might also just include the original C files in your project. If you are programming in another language such as [Nim](https://github.com/samdze/playdate-nim) (see below), this lib might help.

## Building libhebitmap.a

### For device (make sure you have the arm toolchain installed)
```bash
mkdir build
cd build
cmake -DCMAKE_TOOLCHAIN_FILE=/Users/ninovanhooff/Developer/PlaydateSDK/C_API/buildsupport/arm.cmake -DCMAKE_BUILD_TYPE=Release ..
make
```

### For simulator
```bash
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Debug ..
make
```

## Usage in Nim
When using Nim, you can use the [bindings in the nim folder](nim/hebitmap.nim) to use the lib from your project

Note that this is in the Proof of Concept stage. It is usable, but I haven't tested it for memory leaks. Pull requests are welcome.

Make sure to call the initializer once at the start of your program

```nim
import playdate/api
import hebitmap

proc handler(event: PDSystemEvent, keycode: uint) {.raises: [].} =
  if event == kEventInit:
    heBitmapSetPlaydateAPI(playdate)
    # ... other setup code
```

Then

```nim
import playdate/api
import hebitmap

# from file
let heCoinImage = gfx.newHeBitmap("images/coin")
heCoinImage.draw(100, 100)

# custom image
let customImage = gfx.newBitmap(20, 20, kColorBlack)
let heCustomImage = newHeBitmap(customImage)
heCustomImage.draw(50,50)
    
```


## HEBitmap (Playdate)

HEBitmap (**H**igh **E**fficiency Bitmap) is a custom implementation of the drawBitmap function (Playdate SDK).

This implementation is up to 4x faster than the SDK drawBitmap function, but native features (clipRect, flip, stencil) are not supported.

## Benchmark

Bitmap info: 48x22 (masked)

| N Bitmaps | HEBitmap | drawBitmap | Faster
|:---|:---|:---|:---|
| 2000 | 30 ms | 120 ms | 4x
| 1000 | 20 ms | 63 ms | 3x
| 500 | 19 ms | 33 ms | 1.7x

## Example

```c
#include "hebitmap.h"

// Set Playdate API pointer
HEBitmapSetPlaydateAPI(pd);

// Load a LCDBitmap
LCDBitmap *lcd_bitmap = playdate->graphics->loadBitmap("test", NULL);

// New HEBitmap from LCDBitmap
HEBitmap *he_bitmap = HEBitmapNew(lcd_bitmap);

// Draw
HEBitmapDraw(he_bitmap, 0, 0);

// Free
HEBitmapFree(he_bitmap);

// Free original LCDBitmap
playdate->graphics->freeBitmap(lcd_bitmap);
```

## Demo

You can get the compiled PDX demo [here](demo/HEBitmap.pdx.zip "PDX demo"). Press A to switch between custom / SDK implementation.
