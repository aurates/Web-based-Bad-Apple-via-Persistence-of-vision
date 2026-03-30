# Web-based Bad Apple via Persistence of Vision

**Status:** Work in Progress (WIP)

## Project Overview

This project aims to recreate the famous "Bad Apple!!" video using web-based text rendering and persistence of vision effects. The concept leverages rapid text updates and visual persistence to create the illusion of motion and imagery through character-based graphics.

## Core Concept

The project is based on **persistence of vision** - a phenomenon where the human eye retains an image for a fraction of a second after it disappears, creating smooth motion from rapidly changing static images. By rendering frames using text characters at high speed, we can create video-like animations in a web browser.

## Foreseeable Challenges

- **Text density optimization**: Finding the right balance of character density for clear imagery
- **Character selection**: Choosing characters that best represent different brightness levels
- **Video format conversion**: Efficiently converting and storing Bad Apple frames in a text-based format
- **Rendering performance**: Maintaining smooth frame rates in the browser

## Sources & Inspiration

### Core Technologies

- **[pretext](https://github.com/chenglou/pretext)** - Pure JavaScript/TypeScript library for multiline text measurement & layout without DOM reflows. Essential for optimizing text rendering performance.
- **[noise-shader](https://github.com/brantagames/noise-shader)** - Godot engine shader effects using noise patterns to create persistence of vision effects.

### ASCII Video Players & Implementations

- **[tplay](https://github.com/maxcurzi/tplay)** (562★) - Terminal ASCII media player supporting images, gifs, videos, webcam, and YouTube in Rust. Great reference for video-to-ASCII conversion.
- **[asciiplayer](https://github.com/qeesung/asciiplayer)** (237★) - ASCII gif/video player written in Go.
- **[ASCIIPlayer](https://github.com/Esser50K/ASCIIPlayer)** (91★) - Terminal video player using ASCII characters in Python.

### Bad Apple Implementations

- **[ASCII_bad_apple](https://github.com/Chion82/ASCII_bad_apple)** (69★) - Python implementation of Bad Apple as ASCII art video.
- **[Bad-Apple-Console](https://github.com/laike9m/Bad-Apple-Console)** (50★) - ASCII Art Bad Apple in Windows console using C++.
- **[bad-apple-ascii](https://github.com/trung-kieen/bad-apple-ascii)** (46★) - Display video frames in terminal using Python/Bash.
- **[ASCII-Player](https://github.com/BGStudio2021/ASCII-Player)** (34★) - Character art player that can play Bad Apple, implemented in HTML.

### Additional Resources

- **[bad_apple](https://github.com/S0raWasTaken/bad_apple)** (21★) - Rust crates for compiling images/video into asciinema with audio synchronization.
- **[AsciiVid](https://github.com/LeadRDRK/AsciiVid)** (23★) - ASCII video codec and player in C++.

### Persistence of Vision Hardware Projects

- **[pov_led_globe](https://github.com/arttupii/pov_led_globe)** - ESP32-based POV display with APA102 LED strips.
- **[POV_Wheel_Display](https://github.com/aleksei-severin/POV_Wheel_Display)** - ESP32-S3 POV display with web interface for images and GIF animations.

## Technical Approach

1. **Frame Extraction**: Convert Bad Apple video into individual frames
2. **ASCII Conversion**: Map pixel brightness to ASCII characters
3. **Optimization**: Use pretext library for efficient text layout without DOM reflows
4. **Rendering**: Display frames rapidly to achieve persistence of vision effect
5. **Performance**: Maintain consistent frame rate for smooth playback

## License

See LICENSE file for details.
