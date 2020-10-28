# EDITLY 🏄‍♀️

[![demo](https://github.com/mifi/gifs/raw/master/commonFeatures.gif)](https://youtu.be/LNeclLkxUEY)

This GIF / YouTube was created with this command: "editly [commonFeatures.json5](https://github.com/mifi/editly/blob/master/examples/commonFeatures.json5)". See [more examples here](https://github.com/mifi/editly/tree/master/examples#examples).

**Editly** is a tool and framework for declarative NLE (**non-linear video editing**) using Node.js and ffmpeg. Editly allows you to easily and **programmatically create a video** from a **set of clips, images, audio and titles**, with smooth transitions and music overlaid.

Editly has a simple CLI for quickly assembling a video from a set of clips or images, or you can use its more flexible JavaScript API.

Inspired by [ffmpeg-concat](https://github.com/transitive-bullshit/ffmpeg-concat), editly is much faster and doesn't require much storage because it uses **streaming** editing. Editly aims to be very extensible and feature rich with a pluggable interface for adding new **dynamic content**.

## Features

- Edit videos with code! 🤓
- Declarative API with fun defaults
- Create colorful videos with random colors generated from aesthetically pleasing palettes and random effects
- Supports any input size, e.g. 4K video and DSLR photos
- Can output to any dimensions and aspect ratio, e.g. *Instagram post* (1:1), *Instagram story* (9:16), *YouTube* (16:9), or any other dimensions you like.
- Content is scaled and letterboxed automatically, even if the input aspect ratio is not the same and the framerate will be converted.
- Speed up / slow down videos automatically to match the `cutFrom`/`cutTo` segment length with each clip's `duration`
- Overlay text and subtitles on videos, images or backgrounds
- Accepts custom HTML5 Canvas / Fabric.js JavaScript code for custom screens or dynamic overlays
- Render custom GL shaders (for example from [shadertoy](https://www.shadertoy.com/))
- Can output GIF
- Overlay transparent images or even videos with alpha channel
- Show different sub-clips for parts of a clips duration (B-roll)
- Picture-in-picture
- Preserve/mix multiple audio sources
- Automatic audio crossfading
- Automatic audio ducking and normalization

## Use cases

- Create a slideshow from a set of pictures with text overlay
- Create a fast-paced trailer or promo video
- Create a tutorial video with help text
- Create news stories
- Create an animated GIF
- Resize video to any size or framerate and with automatic letterboxing/cropping (e.g. if you need to upload a video somewhere but the site complains `Video must be 1337x1000 30fps`)
- Create a podcast with multiple mixed tracks

See [examples](https://github.com/mifi/editly/tree/master/examples)

## Requirements

- Windows, MacOS or Linux
- [Node.js installed](https://nodejs.org/en/) (Use of the latest LTS version is recommended, [v12.16.2 or newer on MacOS](https://github.com/sindresorhus/meow/issues/144).)
- `ffmpeg` (and `ffprobe`) [installed](http://ffmpeg.org/) and available in `PATH`
- (Linux) may require some extra steps. See [headless-gl](https://github.com/stackgl/headless-gl#system-dependencies).

## Installing

`npm i -g editly`

## Usage: Command line video editor

Run `editly --help` for usage

Create a simple randomized video edit from videos, images and text with an audio track:

```sh
editly \
  title:'My video' \
  clip1.mov \
  clip2.mov \
  title:'My slideshow' \
  img1.jpg \
  img2.jpg \
  title:'THE END' \
  --fast \
  --audio-file-path /path/to/music.mp3
```

Or create an MP4 (or GIF) from a JSON or JSON5 edit spec *(JSON5 is just a more user friendly JSON format)*:

```sh
editly my-spec.json5 --fast --keep-source-audio --out output.gif
```

For examples of how to make a JSON edit spec, see below or [examples](https://github.com/mifi/editly/tree/master/examples).

Without `--fast`, it will default to using the **width**, **height** and **frame rate** from the **first** input video. **All other clips will be converted to these dimensions.** You can of course override any or all of these parameters.

- **TIP:** Use this tool in conjunction with [LosslessCut](https://github.com/mifi/lossless-cut)
- **TIP:** If you need catchy music for your video, have a look at [this YouTube](https://www.youtube.com/channel/UCht8qITGkBvXKsR1Byln-wA) or the [YouTube audio library](https://www.youtube.com/audiolibrary/music?nv=1). Then use [youtube-dl](https://github.com/ytdl-org/youtube-dl) to download the video, and then point `--audio-file-path` at the video file. *Be sure to respect their license!*

## JavaScript library

```js
const editly = require('editly');

// See editSpec documentation
await editly(editSpec)
  .catch(console.error);
```

## Edit spec

Edit specs are JavaScript / JSON objects describing the whole edit operation with the following structure:

```js
{
  outPath,
  width,
  height,
  fps,
  allowRemoteRequests: false,
  defaults: {
    duration: 4,
    transition: {
      duration: 0.5,
      name: 'random',
      audioOutCurve: 'tri',
      audioInCurve: 'tri',
    },
    layer: {
      fontPath,
      // ...more layer defaults
    },
    layerType: {
      'fill-color': {
        color: '#ff6666',
      }
      // ...more per-layer-type defaults
    },
  },
  clips: [
    {
      transition,
      duration,
      layers: [
        {
          type,
          // ...more layer-specific options
        }
        // ...more layers
      ],
    }
    // ...more clips
  ],
  audioFilePath,
  loopAudio: false,
  keepSourceAudio: false,
  clipsAudioVolume: 1,
  audio: [
    {
      path,
      mixVolume: 1,
      cutFrom: 0,
      cutTo,
      start: 0,
    },
    // ...more audio tracks
  ],
  audioNorm: {
    enable: false,
    gaussSize: 5,
    maxGain: 30,
  }

  // Testing options:
  enableFfmpegLog: false,
  verbose: false,
  fast: false,
}
```

### Parameters

| Parameter | CLI equivalent | Description | Default | |
|-|-|-|-|-|
| `outPath` | `--out` | Output path (mp4, mkv), can also be a `.gif` | | |
| `width` | `--width` | Width which all media will be converted to | `640` | |
| `height` | `--height` | Height which all media will be converted to | auto based on `width` and aspect ratio of **first video** | |
| `fps` | `--fps` | FPS which all videos will be converted to | First video FPS or `25` | |
| `allowRemoteRequests` | `--allow-remote-requests` | Allow remote URLs as paths | `false` | |
| `fast` | `--fast`, `-f` | Fast mode (low resolution and FPS, useful for getting a quick preview ⏩) | `false` | |
| `defaults.layer.fontPath` | `--font-path` | Set default font to a .ttf | System font | |
| `defaults.layer.*` | | Set any layer parameter that all layers will inherit | | |
| `defaults.duration` | `--clip-duration` | Set default clip duration for clips that don't have an own duration | `4` | sec |
| `defaults.transition` | | An object `{ name, duration }` describing the default transition. Set to **null** to disable transitions | | |
| `defaults.transition.duration` | `--transition-duration` | Default transition duration | `0.5` | sec |
| `defaults.transition.name` | `--transition-name` | Default transition type. See [Transition types](#transition-types) | `random` | |
| `defaults.transition.audioOutCurve` | | Default [fade out curve](https://trac.ffmpeg.org/wiki/AfadeCurves) in audio cross fades | `tri` | |
| `defaults.transition.audioInCurve` | | Default [fade in curve](https://trac.ffmpeg.org/wiki/AfadeCurves) in audio cross fades | `tri` | |
| `clips[]` | | List of clip objects that will be played in sequence. Each clip can have one or more layers. | | |
| `clips[].duration` | | Clip duration. See `defaults.duration`. If unset, the clip duration will be that of the **first video layer**. | `defaults.duration` | |
| `clips[].transition` | | Specify transition at the **end** of this clip. See `defaults.transition` | `defaults.transition` | |
| `clips[].layers[]` | | List of layers within the current clip that will be overlaid in their natural order (final layer on top) | | |
| `clips[].layers[].type` | | Layer type, see below | | |
| `clips[].layers[].visibleFrom` | | What time into the clip should this layer start | | sec |
| `clips[].layers[].visibleUntil` | | What time into the clip should this layer stop | | sec |
| `audioTracks[]` | | List of arbitrary audio tracks. See [audio tracks](#arbitrary-audio-tracks). | `[]` | |
| `audioFilePath` | `--audio-file-path` | Set an audio track for the whole video. See also [audio tracks](#arbitrary-audio-tracks) | | |
| `loopAudio` | `--loop-audio` | Loop the audio track if it is shorter than video? | `false` | |
| `keepSourceAudio` | `--keep-source-audio` | Keep source audio from `clips`? | `false` | |
| `clipsAudioVolume` | | Volume of audio from `clips` relative to `audioTracks`. See [audio tracks](#arbitrary-audio-tracks). | `1` | |
| `audioNorm.enable` | | Enable audio normalization? See [audio normalization](#audio-normalization). | `false` | |
| `audioNorm.gaussSize` | | Audio normalization gauss size. See [audio normalization](#audio-normalization). | `5` | |
| `audioNorm.maxGain` | | Audio normalization max gain. See [audio normalization](#audio-normalization). | `30` | |

### Transition types

`transition.name` can be any of [gl-transitions](https://gl-transitions.com/gallery), or any of the following: `directional-left`, `directional-right`, `directional-up`, `directional-down`, `random` or `dummy`.

### Layer types

See [examples](https://github.com/mifi/editly/tree/master/examples) and [commonFeatures.json5](https://github.com/mifi/editly/blob/master/examples/commonFeatures.json5)

#### Layer type 'video'

For video layers, if parent `clip.duration` is specified, the video will be slowed/sped-up to match `clip.duration`. If `cutFrom`/`cutTo` is set, the resulting segment (`cutTo`-`cutFrom`) will be slowed/sped-up to fit `clip.duration`. If the layer has audio, it will be kept (and mixed with other audio layers if present.)

| Parameter  | Description | Default | |
|-|-|-|-|
| `path` | Path to video file | | |
| `resizeMode` | See [Resize modes](#resize-modes) | | |
| `cutFrom` | Time value to cut from | `0` | sec |
| `cutTo` | Time value to cut to | *end of video* | sec |
| `width` | Width relative to screen width | `1` | `0` to `1` |
| `height` | Height relative to screen height | `1` | `0` to `1` |
| `left` | X-position relative to screen width | `0` | `0` to `1` |
| `top` | Y-position relative to screen height | `0` | `0` to `1` |
| `originX` | X anchor | `left` | `left` or `right` |
| `originY` | Y anchor | `top` | `top` or `bottom` |
| `mixVolume` | Relative volume when mixing this video's audio track with others | `1` | |


#### Layer type 'audio'

Audio layers will be mixed together. If `cutFrom`/`cutTo` is set, the resulting segment (`cutTo`-`cutFrom`) will be slowed/sped-up to fit `clip.duration`. The slow down/speed-up operation is limited to values between `0.5x` and `100x`.

| Parameter  | Description | Default | |
|-|-|-|-|
| `path` | Path to audio file | | |
| `cutFrom` | Time value to cut from | `0` | sec |
| `cutTo` | Time value to cut to | `clip.duration` | sec |
| `mixVolume` | Relative volume when mixing this audio track with others | `1` | |

#### Layer type 'detached-audio'

This is a special case of `audioTracks` that makes it easier to start the audio relative to `clips` start times without having to calculate global start times.

`detached-audio` has the exact same properties as [audioTracks](#arbitrary-audio-tracks), except `start` time is relative to the clip's start.

[Example of detached audio tracks](https://github.com/mifi/editly/blob/master/examples/audio3.json5)

#### Layer type 'image'

Full screen image

| Parameter  | Description | Default | |
|-|-|-|-|
| `path` | Path to image file | | |
| `resizeMode` | See [Resize modes](#resize-modes) | | |

See also See [Ken Burns parameters](#ken-burns-parameters).

#### Layer type 'image-overlay'

Image overlay with a custom position and size on the screen.

| Parameter  | Description | Default | |
|-|-|-|-|
| `path` | Path to image file | | |
| `position` | See [Position parameter](#position-parameter) | | |
| `width` | Width (from 0 to 1) where 1 is screen width | | |
| `height` | Height (from 0 to 1) where 1 is screen height | | |

See also [Ken Burns parameters](#ken-burns-parameters).

#### Layer type 'title'
- `fontPath` - See `defaults.layer.fontPath`
- `text` - Title text to show, keep it short
- `textColor` - default `#ffffff`
- `position` - See [Position parameter](#position-parameter)

See also [Ken Burns parameters](#ken-burns-parameters)

#### Layer type 'subtitle'
- `fontPath` - See `defaults.layer.fontPath`
- `text` - Subtitle text to show
- `textColor` - default `#ffffff`

#### Layer type 'title-background'

Title with background

- `text` - See type `title`
- `textColor` - See type `title`
- `background` - `{ type, ... }` - See type `radial-gradient`, `linear-gradient` or `fill-color`
- `fontPath` - See type `title`

#### Layer type 'news-title'
- `fontPath` - See `defaults.layer.fontPath`
- `text`
- `textColor` - default `#ffffff`
- `backgroundColor` - default `#d02a42`
- `position` - See [Position parameter](#position-parameter)

#### Layer type 'slide-in-text'
- `fontPath` - See `defaults.layer.fontPath`
- `text`
- `fontSize`
- `charSpacing`
- `color`
- `position` - See [Position parameter](#position-parameter)

#### Layer type 'fill-color', 'pause'
- `color` - Color to fill background, default: randomize

#### Layer type 'radial-gradient'
- `colors` - Array of two colors, default: randomize

#### Layer type 'linear-gradient'
- `colors` - Array of two colors, default: randomize

#### Layer type 'rainbow-colors'

🌈🌈🌈

#### Layer type 'canvas'

See [customCanvas.js](https://github.com/mifi/editly/blob/master/examples/customCanvas.js)

- `func` - Custom JavaScript function

#### Layer type 'fabric'

See [customFabric.js](https://github.com/mifi/editly/blob/master/examples/customFabric.js)

- `func` - Custom JavaScript function

#### Layer type 'gl'

Loads a GLSL shader. See [gl.json5](https://github.com/mifi/editly/blob/master/examples/gl.json5) and [rainbow-colors.frag](https://github.com/mifi/editly/blob/master/shaders/rainbow-colors.frag)

- `fragmentPath`
- `vertexPath` (optional)

#### Arbitrary audio tracks

`audioTracks` property can optionally contain a list of objects which specify audio tracks that can be started at arbitrary times in the final video. These tracks will be mixed (`mixVolume` specifying a relative number for how loud each track is compared to the other tracks). `clipsAudioVolume` specifies the volume of **all** the audio from `clips` relative to the volume of **all** the `audioTracks`.

| Parameter | Description | Default | |
|-|-|-|-|
| `audioTracks[].path` | File path for this track | | |
| `audioTracks[].mixVolume` | Relative volume for this track | `1` | |
| `audioTracks[].cutFrom` | Time value to cut source file **from** | `0` | sec |
| `audioTracks[].cutTo` | Time value to cut source file **to** | | sec |
| `audioTracks[].start` | How many seconds into video to start this audio track | `0` | sec |

The difference between `audioTracks` and **Layer type 'audio'** is that `audioTracks` will continue to play across multiple `clips` and can start and stop whenever needed.

See `audioTracks` [example](https://github.com/mifi/editly/blob/master/examples/audio2.json5)

See also **Layer type 'detached-audio'**.

#### Audio normalization

You can enable audio normalization of the final output audio. This is useful if you want to achieve Audio Ducking (e.g. automatically lower volume of all other tracks when voice-over speaks).

`audioNorm` parameters are [documented here.](https://ffmpeg.org/ffmpeg-filters.html#dynaudnorm)

[Example of audio ducking](https://github.com/mifi/editly/blob/master/examples/audio2.json5)

### Resize modes

`resizeMode` - How to fit image to screen. Can be one of:
- `contain` - All the video will be contained within the frame and letterboxed
- `contain-blur` - Like `contain`, but with a blurred copy as the letterbox
- `cover` - Video be cropped to cover the whole screen (aspect ratio preserved)
- `stretch` - Video will be stretched to cover the whole screen (aspect ratio ignored).

Default `contain-blur`.

See:
- [image.json5](https://github.com/mifi/editly/blob/master/examples/image.json5)
- [videos.json5](https://github.com/mifi/editly/blob/master/examples/videos.json5)

### Position parameter

Certain layers support the position parameter

`position` can be one of either:
  - `top`, `bottom` `center`, `top-left`, `top-right`, `center-left`, `center-right`, `bottom-left`, `bottom-right`
  - An object `{ x, y, originX = 'left', originY = 'top' }`, where `{ x: 0, y: 0 }` is the upper left corner of the screen, and `{ x: 1, y: 1 }` is the lower right corner, `x` is relative to video width, `y` to video height. `originX` and `originY` are optional, and specify the position's origin (anchor position) of the object.

See [position.json5](https://github.com/mifi/editly/blob/master/examples/position.json5)

### Ken Burns parameters

| Parameter  | Description | Default | |
|-|-|-|-|
| `zoomDirection` | Zoom direction for Ken Burns effect: `in`, `out` or `null` to disable | | |
| `zoomAmount` | Zoom amount for Ken Burns effect | `0.1` | |

## Troubleshooting

- If you get `Error: The specified module could not be found.`, try: `npm un -g editly && npm i -g --build-from-source editly` (see [#15](https://github.com/mifi/editly/issues/15))
- If you get an error about gl returning null, see Requirements.
- If you get an error `/bin/sh: pkg-config: command not found`, try to use newest Node.js LTS version

## Donate 🙏

This project is maintained by me alone. The project will always remain free and open source, but if it's useful for you, consider supporting me. :) It will give me extra motivation to improve it.

[Paypal](https://paypal.me/mifino)

## See also

- https://github.com/h2non/videoshow - Inspired editly
- https://github.com/transitive-bullshit/ffmpeg-concat - Inspired editly
- http://www.quasimondo.com/BoxBlurForCanvas/FastBlurDemo.html - Fast blur effect used in editly
- https://github.com/transitive-bullshit/awesome-ffmpeg
- https://github.com/sjfricke/awesome-webgl
- https://www.mltframework.org/docs/melt/

## Videos made by you

Submit a PR if you want to share your videos or project created with editly here.

---

Made with ❤️ in 🇳🇴

[More apps by mifi.no](https://mifi.no/)

Follow me on [GitHub](https://github.com/mifi/), [YouTube](https://www.youtube.com/channel/UC6XlvVH63g0H54HSJubURQA), [IG](https://www.instagram.com/mifi.no/), [Twitter](https://twitter.com/mifi_no) for more awesome content!
