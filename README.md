# A beginner's Cookbook for FFmpeg
**@readwithai** - [**X**](https://x.com/readwithai) - [**blog**](https://readwithai.substack.com/) - [**machine aided reading**](https://www.reddit.com/r/machineAidedReading/comments/1jv7zlw/welcome_to_machine_assisted_reading/)

This is a beginner's cookbook for the audio-video command-line tool, [FFmpeg](https://www.ffmpeg.org/). It builds up the reader's knowledge of FFmpeg's features through examples, before providing more applied examples which link back to early examples to aid understanding and adaptation. You may now  want to jump to the [introduction](#introduction).

I created this guide because I found working out how FFmpeg worked from snippets too difficult and needed to form an understanding from first principles. FFmpeg already has complete reference documentation, but this can be difficult to understanding. By adding concrete examples of features, these features become easier to understand, by giving the user something they can run they have something working that they can adapt, by linking to simpler examples the user can learn on the simple example sand then apply the more complicated features, by providing documentation that links to examples the user is given an opportunity to understand a feature through documentation and then continue.

This guide provides is an [index of features](#features) which goes through different FFmpeg features linked to recipes. FFmpeg has more features than a cookbook can cover completely but provides the ability to [list filters](#list) and [display their parameters](#parameters) as well as providing [terse but useable reference documentation](https://ffmpeg.org/ffmpeg-filters.html). I try to label links with `(example)` if they link to an earlier example of a feature or `(doc)` if they link to external documentation.

>  *I wrote all of this if it seems intersting enough and you'd like to ask me some questions. Then I'm open to doing some [**FFmpeg tutoring**](https://readwithai.substack.com/p/online-ffmpeg-tutoring) on the topic. I could walk through these commands, or apply this approach to your problems.

# Attribution
Thanks to [Ventz Petkov](https://github.com/ventz) for providing me with [various additional recipes](https://github.com/talwrii/ffmpeg-cookbook/issues/1). This is particularly valuable as I am not (at the time of writing) an expert in FFmpeg.

If you think a particularly recipe belongs in this guide, feel free to [add it as an issue](https://github.com/talwrii/ffmpeg-cookbook/issues). It may take me a little time for me to add recipes as I want to keep the guide accessible to beginners, so bear this in mind when considering the whether submitting me recipes is worth your time!

<a name="alternatives"> </a>
# Alternatives and prior work
There are a few other cookbooks. [ffmprovisr](https://amiaopensource.github.io/ffmprovisr/) is quite complete, but doesn't feel like it covers the initial parts of understanding FFmpeg.
[ffmpeg by example](https://ffmpegbyexample.com/) is a website which focuses on examples, but does cover initial use.

haileys has [the beginnings of a cookbook](https://github.com/haileys/ffmpeg-cookbook), as does  [Lorna Jane](https://lornajane.net/posts/2014/my-ffmpeg-cookbook) but these only a few examples. There is [quite complete reference documentation on filters](https://ffmpeg.org/ffmpeg-filters.html#Filtering-Introduction).

There are [various books](https://trac.ffmpeg.org/wiki/BooksAndOtherExternalResources) on FFmpeg, including one which is [available for free](https://github.com/jdriselvato/FFmpeg-For-Beginners-Ebook) online

The [FFmpeg wiki](https://trac.ffmpeg.org/) contains some examples particularly the [filtering section](https://trac.ffmpeg.org/#Filtering). You can also [ask questions on reddit](https://www.reddit.com/r/ffmpeg).

What distinguishes this cookbook is that is is available for free, focuses on good internal and external linking, tries to use an innovative approach to ordering of material and focuses on examples that can be immediately run without any media.

<a name="introduction"> </a>
# Introduction: The core of FFmpeg
`ffmpeg` has a number of core features that when understood help you do a range on things. This introduction tries to cover most of these core features. If you cannot understand the FFmpeg reference documentation or an example that you have found on the internet sufficiently to use it, I would suggest working through this guide or the feature closest to what you want to do. But don't worry - later examples will link back to this introduction.

Once you have read this (or if you want to jump ahead) you might like to look at some [specific topics](#topics) or search for a recipe.


<a name="source"> </a>
<a name="output-filter"> </a>
<a name="output-stream"> </a>
<a name="filter-complex"> </a>
## Create 10 seconds of silence
```bash
ffmpeg -filter_complex 'anullsrc=duration=10s [out]' -map '[out]' -ac 1 silence.wav
```
You can play the silence you have created with `ffplay silence.wav`.

This uses the [source filter (doc)](https://ffmpeg.org/ffmpeg-filters.html#toc-Filtering-Introduction), [anullsrc (doc)](https://ffmpeg.org/ffmpeg-filters.html#anullsrc) which generates ten seconds of silence (specified through the `duration` parameter) and then writes this to a stream labelled '[out]'. The `-map` command specifies stream labelled `[out]` is used in the output.

You can also run this as: `ffmpeg -filter_complex 'anullsrc=duration=10s' -ac 1 silence.wav` and an the output of the filter is implicitly used as the stream for output.

There are other ways to create filters such as [-af](#audio-filter) or [-vf](#video-filter) but both of these require precisely one input and one output. In general, you can also use [`-filter_complex`](#filter-complex) instead of these filters.

<a name="play"> </a>
<a name="sine-play"> </a>
<a name="source-filter"> </a>
<a name="lavfi"> </a>
<a name="colons"> </a>
<a name="parameter-example"> </a>
<a name="ffplay"> </a>
<a name="audio-source"> </a>
## Play a sine wav at 256 hertz for ten seconds
```bash
ffplay -f lavfi 'sine=frequency=256:duration=10'
```

This command specifies that the format of the input is a `libavformat` filter ("LibAVFIlter"). `ffplay` does not have a `-filter_complex` argument but you can use this instead for complex filters. `-f lavfi` can also be used with wht `ffmpeg` - but you must use `-i` to specify the input filter.

The parameters for the source filter, [`sine`(doc)](https://ffmpeg.org/ffmpeg-filters.html#sine), are initiated with an `=` and then separated by `:`.

You do not need to use ffplay to play instead, you can [use the xv output device](#xv).  This can be useful if you have multiple input sources - which `ffplay` does not accept.

> **See also**: [create sine wave with aevalsrc](#aevalsrc-sine), [create arbitrary waveforms](#aevalsrc), [output to an GUI window](#xv)

<a name="write_sine"></a>
## Write a sine wave to a file
```bash
ffmpeg -filter_complex 'sine=frequency=256:duration=10' sine.wav
ffplay sine.wav
```
Note how this compares to [the previous example](#sine-play). This shows how you can convert between filters that play media and those that create a file containing the media.

> **See also**: [a sine wave with aevalsrc](#aevalsrc-sine), [an arbitrary waveform](#aevalsrc)

<a name="stream-example"> </a>
<a name="labels"> </a>
<a name="amix"> </a>
## Combine two sine waves into a chord.
```bash
ffplay -f lavfi 'sine=frequency=256:duration=10s [one]; sine=frequency=512:duration=5s [two]; [one][two]amix'
```
Here we create two streams, `[one]` and `[two]` using the [sine filter (example)](#sine-play). We mix these together into the final output streamm using the [amix filter (doc)](https://ffmpeg.org/ffmpeg-filters.html#amix). `amix` takes two streams as input and mixes them together into a single output.

If you look at the entry for amix in [ffmpeg -filters](#list-filters) with `ffmpeg -filters | grep amix` you will see that it you will see that it is of type `N->A` meaning it takes a number of steams as input and writes them to a single audio stream.

> **See also**: [ffplay vs ffmpeg (examples)](#ffplay), [Combing video streams (example)](#hstack)

<a name="list"> </a>
<a name="list-filters"> </a>
## List available filters
```bash
ffmpeg -filters
```

This command lists all available [filters (example)](#filters-example) together with some information about each filter. The output provides various information about the filter,  `|->A` means a filter is an audio source, `A->A` means that the filter takes an audio stream as [input (example)](#audio-filter) and produces one as [output (example)](#audio-source). Similarly, `V` indicates [video input](#video-filter) or [video output (example)](#video-source).

Filters are listed in the filters [manual page](#man-pages): `man ffmpeg-filters`.

> **See also:** [About filters](#filters-abstract), [Show a filter's parameters (example)](#parameters)

<a name="parameters"> </a>
<a name="item-help"> </a>
<a name="filter-help"> </a>
## Show the parameters for a filter
```bash
ffmpeg --help filter=color
```

This command displays information about the [color (example)](#solid-color) such as which parameters can be used, their default values, [their order](#omit-names) and [what values can be given (example)](#expressions-example).

In the output, some parameters are marked with a `T`, which indicates that they can be [modified through commands (example)](#commands-example).

You can also get help for other objects such as a `decoder`, `encoder`, `demuxer`, `muxer`, `bsf` (bit stream filter), or `protocol`. See the [`man ffmpeg`](#man-pages) for details.

Unfortunately, some information about [filters (example)](#output-filter) such as which parameters can use [expressions (example)](#expression-example) and the variables which can be used in the expressions is not shown in this documentation, but must be found in [reference documentation](#documentation).

> **See also**: [An example of using parameters (example)](#parameters-example), [Expressions (example)](#expression-example), [FFmpeg documentation](#documentation)

<a name="audio-filter"> </a>
## Decrease the volume of an audio file
```bash
ffmpeg -filter_complex `sine=frequency=512:duration=10` sine.wav
ffmpeg -i sine.wav -af 'volume=volume=0.5' quiet-sine.wav
ffplay sine.wav
ffplay quiet-sine.wav
```

This creates a file containing a [sine wav (example)](#sine-play) and then uses the [`volume` filter](https://ffmpeg.org/ffmpeg-filters.html#volume) to decrease the volume to 50% of the original volume by using `volume=0.5`. Because we are using a filter that has precisely one input and output, we can use `-af` rather than `-filter_complex`, but `-filter_complex` would still work. We could use `-filter:a` instead of `-af` which is an identical option which may be easier to understand.

We then [play both the files](#play) the result wwith the `ffplay` program.

> **See also:** [ffplay vs ffmpeg (example)](#play), [Video filters (example)](#video-filter)

<a name="two-streams"> </a>
<a name="concat"> </a>
<a name="concat-stream"> </a>
## Concatenate two audio files together
```bash
ffmpeg -filter_complex 'sine=frequency=512:duration=2' sine1.wav
ffmpeg -filter_complex 'sine=frequency=256:duration=5' sine2.wav
ffmpeg -i sine1.wav -i sine2.wav -filter_complex '[0][1]concat=v=0:a=1'  -ac 1  concat.wav
ffplay concat.wav
```

We [create two sine waves](#write-sine) like in the previous recipes. We then combine together the first (`[0]`) and second (`[1]`) inputs with the `concat` filter. We must specify that concat outputs no video streams and one audio stream because these are not the defaults: concat works with one video stream by default.


Here is the documentation for `concat`:

```
> ffmpeg --help filter=concat
Filter concat
  Concatenate audio and video streams.
    Inputs:
        dynamic (depending on the options)
    Outputs:
        dynamic (depending on the options)
concat AVOptions:
   n                 <int>        ..FVA...... specify the number of segments (from 1 to INT_MAX) (default 2)
   v                 <int>        ..FV....... specify the number of video streams (from 0 to INT_MAX) (default 1)
   a                 <int>        ..F.A...... specify the number of audio streams (from 0 to INT_MAX) (default 0)
   unsafe            <boolean>    ..FVA...... enable unsafe mode (default false)
```

<a name="params"> </a>
<a name="solid-color"> </a>
## Play ten seconds of red
```bash
ffplay -f lavfi color=color=red:duration=10s
```

Here we use the [color filter (doc)](https://ffmpeg.org/ffmpeg-filters.html#allrgb_002c-allyuv_002c-color_002c-colorchart_002c-colorspectrum_002c-haldclutsrc_002c-nullsrc_002c-pal75bars_002c-pal100bars_002c-rgbtestsrc_002c-smptebars_002c-smptehdbars_002c-testsrc_002c-testsrc2_002c-yuvtestsrc) together with the color name of `red` and a `duration` of 10s.

You can also run this as `ffplay -f lavfi color=red:duration=10s` since color is a default parameter.

You can list the available colors with `ffmpeg -colors`.


> **See also**: [ffplay vs ffmpeg](#play)

<a name="image"> </a>
<a name="frames"> </a>
## Create an image consisting of solid red
```bash
ffmpeg -filter_complex 'color=color=red' -frames:v 1  red.png
ffplay red.png
```
Here we specify that we are reading one frame with `-frames:v 1`.

> **See also**: [Working with images](#images), [Extract a frame](#ts-frame)

<a name="video-source"> </a>
## Write ten seconds of red to a file
```bash
ffmpeg -filter_complex 'color=color=red:duration=10s' red.mp4
```

We use the [color filter (doc)](https://ffmpeg.org/ffmpeg-filters.html#allrgb_002c-allyuv_002c-color_002c-colorchart_002c-colorspectrum_002c-haldclutsrc_002c-nullsrc_002c-pal75bars_002c-pal100bars_002c-rgbtestsrc_002c-smptebars_002c-smptehdbars_002c-testsrc_002c-testsrc2_002c-yuvtestsrc) together with the duration parameter of ten seconds and the color parameter of red.

We use `-filter_complex` because there is no input file since `color` is a source filter. We specify the output files as `red.mp4`.

<a name="fade"> </a>
<a name="video-source-filter"> </a>
## Create a video that fades between two colours
```bash
ffplay -f lavfi 'color=color=red:duration=5s [red]; color=color=blue:duration=5s [blue]; [red][blue]xfade=duration=5s'
```

We create two streams, one consisting of solid red, the other of solid blue. We then feed both these streams into the `xfade` filter which "cross-fades" between the two over 5 seconds.

> **See also**: [ffplay vs ffmpeg](#play)

<a name="comma"> </a>
<a name="timestamp"> </a>
<a name="drawtext"> </a>
<a name="count"> </a>
## Create a video that counts up to 10
```bash
ffplay -f lavfi -i  'color=color=white, drawtext=text=%{pts}'
```

This uses the [drawtext filter's](https://ffmpeg.org/ffmpeg-filters.html#drawtext-1) expression language to show the current timestamp in each frame. `%{pts}` is [expanded](https://ffmpeg.org/ffmpeg-filters.html#Text-expansion) in each frame to the number of seconds into the frame.

See the [section on positioning text](#positioning)

<a name="middle"> </a>
<a name="variables-1"> </a>
<a name="expression-example"> </a>
## Place text in the middle of the screen
```bash
ffplay -f lavfi -i 'color=color=white, drawtext=text=hello:x=(main_w - text_w) / 2:y=(main_h-text_h) /2'
```

We [render text using the drawtext filter](#text) as we did in this previous recipe. But we also use the `x` and `y` parameters that accept [drawtext's own expression language(doc)](https://ffmpeg.org/ffmpeg-filters.html#Syntax).

Note that we the division operation, `/`, in the expression. `main_w` represents the video width, `main_h` its height, `text_w` the rendered text's width and `text_h` its height.

In the `x` parameter for `drawtext`, [expressions](#expressions-topic) are evaluated using [ffmpeg's own expression language(doc)](https://ffmpeg.org/ffmpeg-utils.html#toc-Expression-Evaluation) which provides various functions (`sin`, `cos`, etc) and operations (`*`, `/`, `-`, etc).

> **See also**: [ffplay vs ffmpeg](#play)


<a name="capture-screen"> </a>
<a name="x11grab"> </a>
## Capture the screen using Linux
```bash
ffplay  -f x11grab  -i ''
```

This example works on [Linux](#programming) but there are [analogs for other operating systems](https://trac.ffmpeg.org/wiki/Capture/Desktop)

This [section of the wiki of the ffmpeg wiki](https://trac.ffmpeg.org/wiki/Capture/Desktop) describes how to capture on different systems. You can also [capture a specific window](#capture-window), or a [specific region](#capture-region).

[x11grab](https://www.ffmpeg.org/ffmpeg-devices.html#x11grab) supports various options which can for example be used to capture a section of the screen.

> **See also**: [Recording your computer](#computer), [Record a window](#capture-window), [Record a region of your screen](#capture-region), [list devices](#devices), [ffplay vs ffmpeg](#play)

<a name="scale"> </a>
## Change the size of a video
```bash
ffmpeg -filter_complex 'color=color=white:size=1024x512:duration=5s, drawtext=text=%{pts}:fontsize=h' count.webm
ffmpeg -i count.webm -filter_complex 'scale=512:256' count-resized.webm
ffplay count-resized.webm
```

First we create an image with a known size of 1024x512 which [displays the time on the video (example)](#timestamp) using drawtext. We then use the [scale filter (doc)](https://ffmpeg.org/ffmpeg-filters.html#scale-1) to scale to a given size. Here we [omit the names (example)](#omit-name) of parameters of the `width` and `height` parameters of scale

You can replace one of the `width` or `height` parameters with `-1` to make FFmpeg calculate this parameter based on [aspect ratio](#video-glossary).

> **Reference**: The FFmpeg wiki has a [section on scaling](https://trac.ffmpeg.org/wiki/Scaling)

<a name="text"> </a>
<a name="input-stream"> </a>
## Create a video with the phrase "hello world" on a white background
```bash
ffmpeg -filter_complex  'color=color=white:duration=10s, drawtext=text=hello:fontsize=20' out.mp4
```

`drawtext` takes a video stream as input and draws text on top of it. Note the use of `,` to feed the output of one stream into the next.

We could, however, have used labels instead as we did before when [concatenating audio streams](#concat), like so:

```bash
ffmpeg -filter_complex  'color=color=white:duration=10s [one]; [one] drawtext=text=hello:fontsize=20' out.mp4
```

<a name="trim"> </a>
<a name="video-filter"> </a>
## Trim a video to five seconds
```bash
ffmpeg -filter_complex 'color=color=white, drawtext=text=%{pts}, trim=duration=10s' count-to-ten.mp4
ffmpeg -i count-to-ten.mp4 -vf 'trim=duration=5s' count-to-five.mp4
```

First we create a video which shows the timestamp each frame and is ten seconds long. We then apply the `trim` filter to limit this to ten seconds.

Here we can use the `-vf` because our filter has precisely one input and one output - but we could equally well use [-filter_complex (example)](#filter-complex) instead.

<a name="static-image"> </a>
## Turn a static image into a five second video of the image
```bash
ffmpeg -filter_complex 'color=color=white, drawtext=text=hello:fontsize=40' -v:frame 1 static.png
ffmpeg -i static.png  -filter_complex 'nullsrc=duration=5s [vid]; [vid][0]overlay' out.mp4
```

First we [create an image](#image) using the `-v:frame` option. We then create a 5 second empty video using the `nullsrc` filter and overlay the static image on it.

## Concatenate two videos together
```bash
ffmpeg -filter_complex 'color=color=white:duration=5s, drawtext=text=%{pts}' count-up.mp4
ffmpeg -filter_complex 'color=color=white:duration=5s, drawtext=text=%{pts}, reverse' count-down.mp4
ffmpeg -i count-up.mp4 -i count-down.mp4  -filter_complex '[0][1]concat' concat.mp4
```

First we create two videos one that [counts up to ten seconds](#timestamp), and then the reverse of this using the [reverse (doc)](https://ffmpeg.org/ffmpeg-filters.html#reverse) filter.

We then use the concat filter (as we did for [audio](#concat)) but this time we do not need to use concat - since the default output is a single video stream and no audio stream.

## Add a static introduction image to the beginning of a video
```bash
ffmpeg -filter_complex 'color=color=white, drawtext=text=hello:fontsize=40' -v:frame 1 static.png
ffmpeg -filter_complex 'color=color=red:duration=5s [red]; color=color=green:duration=5s [green]; [red][green]xfade=duration=5s' video.mp4
ffmpeg  -i static.png -i video.mp4 -filter_complex 'nullsrc=duration=2s, [0]overlay [intro]; [intro][1]concat' with-title.mp4
```

First we create a static image with the word hello and a video which fades from red to green. Then we combine the two [overlaying the static image](#static-image) on a 2 second video and call it intro. This is concatenated with a video.

## Add a logo to the corner of a picture
```bash
ffmpeg -filter_complex 'color=white:size=100x100, drawtext=text=L:fontsize=h' -frames:v 1 logo.png
ffmpeg -filter_complex 'color=red [red]; color=blue [blue]; [red][blue]xfade=duration=5s, trim=duration=5s' video.webm
ffmpeg -i video.webm -i logo.png -filter_complex '[0][1]overlay=x=W-w:y=H-h' final.webm
ffplay final.webm
```

We [create an image (example)](#image) consisting of the letter L, [scaled (example)](#scale-text) to the size of the image. We then create an [image fading between colours (example)](#fade). These act as input for our main filter.

We then input both these files into FFmpeg. We must use `-filter_complex` because we have two inputs.

We place the logo over the image using the overlay filter by specifying the `x` and `y` parameter. These are [expressions](#expressions) using the variables `W`, `w`, `H` and `h` for the weights and heights of the first and second images.

## Add music to a video
```
ffmpeg -filter_complex 'color=blue [blue]; color=yellow [yellow]; [blue][yellow]xfade=duration=10s, trim=duration=10s' fade.webm
ffmpeg -filter_complex 'sine=frequency=32+40*t:eval=true
```

## Make a black and white version of the screen
```bash
ffmpeg -f x11grab  -i '' -vf 'hue=s=0, format=yuv420p'  -f xv ''
```

Here we take the [input the screen](#xv)

<a name="commands-example"> </a>
<a name="command-example"> </a>
## Change color of a frame over time
The `sendcmd` filter allows you to change parameters at other filters at particular timestamps ([amongst other things (doc)](https://ffmpeg.org/ffmpeg-filters.html#sendcmd_002c-asendcmd))

```
ffplay -f lavfi -i color@x, sendcmd=commands=1 color@x color blue; 2 color@x color green; 3 color@x color purple
```

`color@x` creates a color filter with the name `color@x` which can be used to distinguish different filters.

For certain changes over time, particularly "continuous" changes you can use [expressions with the `t` parameter](#time-expressions) including using the [if expression](#if-expression) - but some commands do not support [expressions](#expressions-topic) and for discrete changes sendcmd can be more useful.

Note that [audio filters](#asendcmd) and video filters use a different filter to change commands.

> **See also**: [Section on commands](#commands-topic)

<a name="asendcmd"> </a>
<a name="escaping-audio"> </a>
## Change the frequency of an audio bandpass
```
ffplay -f lavfi -i  "aevalsrc=exprs=0.2 * gte(sin(128 * t * 2 * PI)\,0), bandpass=frequency=250, asendcmd=commmands='2 bandpass frequency 100'"
```

This example is similar to the [previous one](#command-example) but for an audio filter rather than a video filter.

We create a square wave because these [has a lot of overtones (wiki)](https://en.wikipedia.org/wiki/Subtractive_synthesis), using the [aevalsrc filter (example)](#aevalsrc) with [an expression](#expression) that allows one to specific a formula, that we send through a bandpass filter. This waveform is then sent into [asendcmd (doc)](https://ffmpeg.org/ffmpeg-filters.html#sendcmd_002c-asendcmd) which has a command to change the frequency of the bandpass signal after two seconds.

Unfortunately, FFmpeg requires you to use a different filter for sending commands to audio filters than video filters, so we use [asendcmd (doc)](https://ffmpeg.org/ffmpeg-filters.html#sendcmd_002c-asendcmd) rather than [sendcmd (example)](#command-example).

> **See also**: [Audio engineering](#audio)

<a name="devices"> </a>
## List available devices
```bash
ffmpeg -devices
```

`D` means an input device. (demuxer). `E` means an output device (muxer).

You can see more information about devices with [`man ffmpeg-devices`](#man-pages).

> **See also**: [xv device for video output](#xv), [x11grab device for video input](#x11grab)


## Create a grid of 16 frames taken from the video
```
ffmpeg  -filter_complex 'color=white, drawtext=text=%{pts}:x=w/2 + 0.6*w/2*sin((t / 8) * 2 * PI):y=h/2 + 0.6 * h/2*cos((t/8) * 2 * PI), drawbox=w=iw:h=ih:color=red, trim=duration=8s' orbit.mp4
ffmpeg -i orbit.mp4 -r 2 orbit-%02d.png
ls orbit-*.png  | sed 's/^/-i /' | xargs  ffmpeg -y -filter_complex '[0][1][2][3]hstack=4 [row1]; [4][5][6][7] hstack=4[row2]; [8][9][10][11]hstack=4[row3]; [12][13][14][15]hstack=4 [row4]; [row1][row2][row3][row4] vstack=4' -f apng  grid.png
ffplay grid.png
```
We first create an interesting (or sufficiently interesting for our cases here) image of the current timestamp moving in a circle. We then extract out two frames a second by setting the [rate parameter, -r, (doc)](https://ffmpeg.org/ffmpeg.html#toc-Video-Options) to 2 frames per second and outputting each frame to file by using a filename of `orbit-%02d.png` in the name. This is a format string for the [printf](#programming) function in the programming language. the %02d means that e.g. the 9th frame is wring to the file `orbit-09.png`.
In the next expression we use some command-line magic to run `ffmpeg -i orbit-01.png -i orbit-02.png ...` before horizontally stacking all of the images into rows with [hstack (example)](https://ffmpeg.org/ffmpeg-filters.html#hstack) and vertically stacking these rows with vstack. Each of these commands takes four [inputs (example)](#two-inputs). We specify the [default parameter (example)](#omit-names) `inputs` of each to 4.

<a name="topics"> </a>
## Finishing up
Hopefully this introduction has introduced to a broad range of how ffmpeg works and what it can do through examples. The rest of the guide covers topics in the style of a more traditional cookbook.

Here are some topics covered

* [Recording your computer](#recording)
* [Formatting text](#text)
* [Audio engineering](#audio)
* [Interesting outputs from FFmpeg](#text)
* [Controlling FFmpeg externally](#external)

We also cover some more topics of Thor internals of ffmpeg - while linking to examples.

* [Using commands](#commands)
* [Writing more readable filters](#readability)
* [Getting information about FFmpeg](#reflection)




<a name="recording"> </a>
<a name="computer"> </a>
# Recording your computer
<a name="capture-window"> </a>
<a name="capture-specific"> </a>
##  Record a specific window using Linux
```bash
xwininfo
ffplay -window_id 0x520003e -f x11grab -i ''
```

This example works on [Linux](#programming), but there are [similar filters for different operating systems](https://trac.ffmpeg.org/wiki/Capture/Desktop).

This uses the `-window_id` option which is an option for the x11grab [device](#devices) to record a specific window [on your screen](#capture-screen).

There are [various options](https://www.ffmpeg.org/ffmpeg-devices.html#toc-Options-20) that allow you to select a specific region (`grab_x`, `grab_y`, `video_size`), follow the mouse (`-fillow_mouse`), hide the mouse (`-draw_mouse`),  and other things (`framerate`, `show_region`, `framerate`).

## Record a specific region of the screen
```bash
ffmpeg -f x11grab -select_region 1 -i ''  region.mp4
# ffmpeg -f x11grab  -i ''  -select_region 1 region.mp4 - this does not work
ffplay region.mp4
```

This uses [x11grab](#capture-screen) to capture a region selected with the cursor when run (specified by [the option](https://ffmpeg.org/ffmpeg-devices.html#Options-20) `-select_region 1`). Note that `-select_region 1` must come before `-i`

> **See also**: [Record a specific window](#specific-window), [record the screen](#capture-screen), [List devices](#devices)

# Cutting, combining and formatting
## Combining video and audio
```
ffmpeg -filter_complex 'aevalsrc=pow(sin(128 * t * 2 * PI)\, sin(t) * sin(t)), atrim=duration=10s' warble.wav
ffmpeg -filter_complex 'color=white [white]; color=black [black]; [white][black]xfade=duration=10s, trim=duration=10s' transition.webm
ffmpeg -i warble.wav -i transition.webm combined.mp4
ffplay combined.webm
```

<a name="xstack"> </a>
<a name="xstack-zero-index"> </a>
## Combine two videos with different sizes
```
ffmpeg -filter_complex 'color=red:size=200x100, trim=duration=10s' red.mp4
ffmpeg -filter_complex 'color=blue:size=300x400, trim=duration=10s' blue.mp4
ffmpeg -i red.mp4 -i blue.mp4 -filter_complex '[0][1]xstack=layout=0_0|w0_0:fill=black' combined.mp4
```
Here we create two videos [consisting of solid colour (example)](#solid-color) which are ten seconds long but with different sizes. The first is red and 200 pixels wide and 100 pixels wide. The second is blue and 300 pixels wide and 400 pixels high.

We then combine these together with the [xstack (doc)](https://ffmpeg.org/ffmpeg-filters.html#xstack-1). `xstack` works by drawing videos into an imaginary canvas. The `layout` parameter specifies the top left coordinate of the rectangle for each source using the variables `w0`, `h0`, `w1`, `h1` ... etc for the width and heights of images. Somewhat confusingly the indexes start at 0 rather than 1 (this is referred to as [zero-based numbering](#zero-index)).

So this example the first image is display at `(0, 0)` and the second image is placed to the right of the image but at the same height at `(first_source_width, 0)`.

If we wanted to reverse this so that the second image is drawn first then the second image we could use the following expression:

`ffmpeg -i red.mp4 -i blue.mp4 -filter_complex '[0][1]xstack=layout=w1_0|0_0:fill=black' combined.mp4`

If images have matching dimensions you can use the [hstack (example)](#hstack) or [vstack (doc)](https://ffmpeg.org/ffmpeg-filters.html#vstack-1) filters for this.

> **Reference**: The FFmpeg wiki has [a section on xstack](https://trac.ffmpeg.org/wiki/Create%20a%20mosaic%20out%20of%20several%20input%20videos%20using%20xstack) together with diagrams.

> **See also**: [Pixel](#av-concepts), [coordinates](#av-concepts)

<a name="text"> </a>
# Formatting text
## Placing text in the middle of the screen
See the [earlier example](#middle)


<a name="positioning"> </a>
## Placing text at different positions
```bash
ffplay -f lavfi -i 'color=color=white, drawtext=text=bottom-right:x=main_w - text_w:y=(main_h-text_h), drawtext=text=top-left:x=0:y=0, drawtext=text=top-right:x=main_w-text_w:y=0, drawtext=text=bottom-left:x=0:y=main_h - text_h'
```

This recipe renders text in various positions. See the [previous recipe](#middle) for the meaning of expressions.

> **See also**: [ffplay vs ffmpeg](#play)

<a name="scale-text"> </a>
## Scaling text to the size of the video

```bash
ffplay -f lavfi 'color=color=white:size=500x100, drawtext=text=hello:fontsize=main_h'
```

Here we use an expression in `fontsize` using the `main_h` variable so the font is the size of the screen. We also use the `size` parameter in color so that the full string fits in the window.

<a name="bold"> </a>
## Draw bold text
[drawtext (doc)](https://ffmpeg.org/ffmpeg-filters.html#drawtext-1) offloads the styling of fonts to the [fontconfig library (doc)](https://www.freedesktop.org/wiki/Software/fontconfig/) or a font file that you give it. This means you need to understand ["Font Matching" and "Font Patterns"](https://fontconfig.pages.freedesktop.org/fontconfig/fontconfig-user.html) a little to use fonts.

`fc-match` and `fc-match -a` can be good to search fonts and show the font that will be applied.

```
ffplay -f lavfi 'color=white, drawtext=text=hello:fontfile=\\:style=Bold:fontsize=h'
```

This query uses a fontconfig generic query string `:style=Bold` in the `fontfile` field. We need to escape the colon with two backslashes. We use the draw text filter (example) as before


## Italic text
```
ffplay -f lavfi 'color=white, drawtext=text=hello:fontfile=\:style=Italic:fontsize=h'
```

This query uses a fontconfig generic query [like the bold examples (example)](#bold) together with the [drawtext filter to draw text (example)](#text) but the size it Italic

## Draw text in serif font
```
ffplay -f lavfi 'color=white, drawtext=text=hello:fontfile=Serif:fontsize=h'
```
Again we [use a fontconfig query (example)](#bold). This time we search for the font name.


# Working with images
FFmpeg provides filters to operate on video streams and audio streams and to combine these streams together into files. However, because an image is a video containing one frame FFmpeg can also operate on images.

Some of the image based functionality which FFmpeg provides is analogous to the [Imagemagick](https://imagemagick.org/script/command-line-processing.php). Imagemagick only operates on images, so if you learn FFmpeg then what you have learned will also apply to videos. However, imagemagick has more features. Animated GIFs and webm images are examples of some "video functionality" that imagemagick provides.

In general, if you want to output an image you can [use the `-frames:v 1` option (example)](#frames) to extract and image.

<a name="ts-frame"> </a>
<a name="trim-start"> </a>
## Extract a frame from a video
```
ffmpeg -filter_complex 'color=color=white, drawtext=text=%{pts}, trim=duration=10s' counting.mp4
ffmpeg -i counting.mp4 -vf 'trim=start=4s' -frames:v 1  four.png
```

First we [generate a video that counts up (example)](#count) to sample from. Then we use the [trim filter (example)](#trim) [(doc)](https://ffmpeg.org/ffmpeg-filters.html#trim) filter to skip to four seconds into the video (using the `start`) option. We use the `frames:v` option to [extract a single frame (example)](#frames) and save this in a png.



<a name="audio"> </a>
<a name="aevalsrc-sine"> </a>
<a name="variables-2"> </a>
# Audio engineering
FFmpeg is not explicitly designed for audio engineering and there a number of tools better suited for serious engineering. However, `ffmpeg` is performant, convenient for editing videos that need a little engineering and these examples are fun, interactive, and can demonstrate features
## Generating a sine wave using aevalsrc
[aevalsrc (doc)](https://ffmpeg.org/ffmpeg-filters.html#aevalsrc) allows you to use a function to express a sound. Here we reimplement the [sine wave recipe](#sine) using this.

```
ffplay -f lavfi -i 'aevalsrc=exprs=sin((128) *t*2*PI)'
```

Here we use an expression involving `t` which is the time in seconds. [ffmpeg's own expression language] (https://ffmpeg.org/ffmpeg-utils.html#toc-Expression-Evaluation) which provides various functions.

<a name="time-expressions"> </a>
<a name="aevalsrc"> </a>
## Generating some interesting sounds with aevalsrc
```
# Create a sine wave that slowly changes frequency
ffplay -f lavfi -i 'aevalsrc=exprs=sin((32 + 100* tanh(t/10) ) *t*2*PI)'

# Create a square wave
ffplay -f lavfi -i 'aevalsrc=exprs=gte(sin(250 * t * 2 * PI)\, 0)'
```

All of these examples use [aevalsrc (example)](#aevalsrc-sine) [(doc)](https://ffmpeg.org/ffmpeg-filters.html#aevalsrc).

In the second example we use the `sin` to give us something periodic with a known function and then use the greater than or equal function `gte` to turn this into a [square wave (wiki)](https://en.wikipedia.org/wiki/Subtractive_synthesis).  Note how we escape


<a name="outputs"> </a>
# Interesting outputs for ffmpeg
<a name="xv"> </a>
## Display output rather than write it to a file

You can use [ffplay (example)](#ffplay) to write to display a file out the output a filter. But this is a wrapper around features that `ffmpeg` provides.

```bash
ffmpeg -filter_complex 'color=red[red]; color=blue[blue]; [red][blue]xfade=duration=5s, trim=duration=5s' spectrum.webm
ffmpeg -re  -i spectrum.mp4  -vf 'format=yuv420p' -f xv title
```

`-re` tells `ffmpeg` to read the file at normal speed (otherwise ffmpeg would read as fast as possible). `format=yuv420p` converts to a format that `xv` accepts. `-re` does not work when you use [source filters (example)](#source) for these you have to use the [real-time filter(example)](#realtime)

<a name="realtime"> </a>
## Limit the speed when using xv with filters
The `xv` video allows ffmpeg to create real time video output. If you do not use the [`-re` filter (example)](#xv) or use [source filters (example)](#video-source-filters) then by default ffmpeg will run as fast as possible - even when writing to `xv`. You will not be able to watch the output and this will use 100% of the CPU. You can fix this use the [realtime (doc)](https://ffmpeg.org/ffmpeg-filters.html#realtime_002c-arealtime) filter.

```
ffmpeg -filter_complex 'color=white, drawtext=text=%{pts}:fontsize=h, realtime' -f xv ''
# This would run too fast:
# ffmpeg -filter_complex 'color=white, drawtext=text=%{pts}:fontsize=h -f xv ''

ffmpeg -filter_complex 'color=white, drawtext=text=%{pts}:fontsize=h, trim=duration=10s' video.mp4
ffmpeg -i  video.mp4 -vf 'realtime' -f xv ''
```

The first example creates a stream counting up using [drawtext (example)](#drawtext). and then [sends this xv (example)](#xv). We specify `realtime` as the final filter to play at normal speed.

> **See also**: [Play media with ffplay (example)](#ffplay)
<a name="external"> </a>

## Playing videos on the terminal with kitty
There [terminal emulator kitty](https://github.com/kovidgoyal/kitty) extended the the protocol that terminals used that it can render pixels. Some other terminal emulators have adopted this standard. You can use this to display the output from ffmpeg - including videos - directly in the terminal.

This can be used to display images directly in the terminal. This is partly a neat trick - but it can have some benefits in terms of productivity and avoiding "paper cuts" while working because you does not have to move windows around. For many use cases one might prefer to use streaming to send output to a consistent window.

The utility [timg](https://github.com/hzeller/timg) works well here.

```
sudo apt-get install timg
sudo apt-get install kitty
kitty
ffmpeg -loglevel 16 -filter_complex 'color=blue, drawtext=text=hi:x=w/3 + w/3*sin(t):y=h/3 + h/3*cos(t):fontsize=h/5' -f webm - | timg -V -
```

First, we install `timg` for play videos in a terminal and `kitty` for a terminal that supports images. We then creating an interesting video using ffmpeg to [render some text (example)](#text) which is animated using an [expression in time (example)](#time-expression) to [change the position of our text (example)](#positioning).

The output is written to [standard output](#programming), as specified by  `-`,  and then read from standard in by `timg`. Because there is no file name we must specify the format of the output stream using `-f`.  We use `-loglevel 16` to hide logging about FFmpeg's normal operation which would otherwise get in the way of the video playing in your terminal.

## Controlling FFmpeg externally
If you are using realtime outputs such as [X11 video output](#xv) or streaming it may make sense to control FF mpeg externally. You can do this using [commands (example)](#command-example) which can be [controlled from the command line or programs via zmq](#zmq).

<a name="readability"> </a>
# Writing more readable filters
The readability of code can be a bit of a trade off. What is easier to read for an expert may use concepts that the new user finds difficult to grasp. Code has an audience like prose, and the features used should cater for this audience. There are various approaches to change filters are written which may improve readability.

<a name="hstack"> </a>
## Using source filters as input
In some examples, we use complex filters with `-filter_complex` to support [source filters (example)](#source-filters). The output of source filters will often be [labelled (example)](#labels). You may be able to simplify such filters by moving parts of the filter out of the main filter and into an input source [of type lavfi (example)](#lavfi).

Here is an example:

```bash
ffmpeg -f lavfi -i 'color=red' -f lavfi -i 'color=green' -f lavfi -i 'color=blue' -filter_complex '[0][1][2]hstack=inputs=3, trim=duration=5s' stacked.mp4
```

This command uses [`-f lavfi` (example)](#lavfi) to create three inputs consisting of solid red, green, and blue. We then combine this three inputs together with [hstack](https://ffmpeg.org/ffmpeg-filters.html#hstack-1) to produce a "flag" video. The argument of `-filter_complex` is definitely shorter as a result.

> **See also**: [Combining video streams of different size(example)](#xstack), Combining audio filters [simulateneous (example)](#amix), [sequentially (example)](#concat-stream)

## Creating a filter graph in a file
With `-filter_complex_script`, you can read a filter specification ("filter graph") from a file. Sometimes command can be more readable it details are separated out into a file.

```bash
echo "color=red:duration=5s" > filter.lavfi
ffmpeg -filter_complex_script filter.lavfi red.mp4
ffplay red.mp4
```
<a name="omit-name"> </a>
## Omitting parameter names
Parameters have an order that can you can view with `ffmpeg --help filter=color`. If you provide parameters in this order then you can omit the parameter name.

Here is an example:

```bash
ffmpeg --help filter=color
ffplay -f lavfi -i 'color=white:100x100'
```

We output a video with solid colour with a colour of `white` and `size` of `100x100`. We can omit both parameter names since the first parameter is `color` and the second is `size`.

Here is some of the output from `ffmpeg --help filter=color`. Notice the order of parameters and the fact that some parameters have aliases which do not form part of the parameter order.

```
color AVOptions:
   color             <color>      ..FV.....T. set color (default "black")
   c                 <color>      ..FV.....T. set color (default "black")
   size              <image_size> ..FV....... set video size (default "320x240")
   s                 <image_size> ..FV....... set video size (default "320x240")
   rate              <video_rate> ..FV....... set video rate (default "25")
   r                 <video_rate> ..FV....... set video rate (default "25")
   duration          <duration>   ..FV....... set video duration (default -0.000001)
   d                 <duration>   ..FV....... set video duration (default -0.000001)
   sar               <rational>   ..FV....... set video sample aspect ratio (from 0 to INT_MAX) (default 1/1)
```

## Using short or long flags
Many of the flags for FFmpeg can long and short versions. Using long versions may make is easier to work out what an option is doing from context. Using short options can make the option short and reduce unnecessary reading.

`-filter:a` can be written as `-af` and creates an [audio filter](#audio-filter).
`-filter:v` can be written as `-vf` and creates a [video filter](#video-filter).


## Using flags or filters
Some of the functionality of filters can be implemented instead as command-line flags.
Using flags can pull complexity out of filters as a form of modularity. Converting flags into filters may result in a command that is easier to read for unfamiliar uses and more suitable for search. There may be features available for flags which are unavailable for features.

`-ss` can be used to seek to a particular position to start with so that some material is skipped You can use the [`trim` filter (doc)](https://ffmpeg.org/ffmpeg-filters.html#trim) instead of `-ss` with the [start option (example)](#trim-start).

`-sseof` is analogus to `-ss` but skips material. Again the one can use the
[`trim` filter (doc)](https://ffmpeg.org/ffmpeg-filters.html#trim) instead of this with the end paramater.



<a name="reflection"> </a>
# Getting information about FFmpeg
FFmpeg has various functions to provide documentation and information about how to run FFmpeg by calling FFmpeg itself.
## Listing objects
FFmpeg has various types of objects that form part of commands. These objects can be listed like so:

```bash
ffmpeg -colors
ffmpeg -filters
ffmpeg -devices
ffmpeg -devices
ffmpeg -codecs
ffmpeg -decoders
ffmpeg -encoders
ffmpeg -protocols
ffmpeg -filters
ffmpeg -pix_fmts
ffmpeg -sample_fmts
ffmpeg -sinks pulse
```

See [`man ffmpeg`](#man-pages) for details.

> **See also**: [Listing ffmpeg filters (example)](#list-filters) , [FFmpeg documentation](#documentation)

<a name="commands"> </a>
<a name="commands-topic"> </a>
# Using commands
[Commands (doc](https://ffmpeg.org/ffmpeg-filters.html#sendcmd_002c-asendcmd) ([example](#command-example)) are a general mechanism to modify the behaviour of filters while they are running such as by changing the value of [parameters (example)](#parameter-example).

> **See also**: [Expressions](#expressions-topic)

## Check that parameter can be updated by a command
The [help for a filter (example)](#parameters) indicates whether a filter can be modified by a command with the letter `T`

```
ffmpeg --help filter=drawtext
```

Here, in the output of `ffmpeg --help filter=drawtext`. We can see see that its text can be updated by commands, but that `fontfile` and `textfile` (if used) cannot be.

```
Filter drawtext
  Draw text on top of video frames using [D[C`libfreetype` library.
    Inputs:
       #0: default (video)
    Outputs:
       #0: default (video)
drawtext AVOptions:
   fontfile          <string>     ..FV....... set font file
   text              <string>     ..FV.....T. set text
   textfile          <string>     ..FV....... set text file
```

Many commands such as `sine` have parameters that cannot be updated even though you might expect them to be mutable.

<a name="zmq"> </a>
## Update commands from the command line with zmq
The [zmq filter (doc)](https://ffmpeg.org/ffmpeg-filters.html#zmq_002c-azmq) allows one to read commands to control filters from an external source. `zmq` is a popular protocol used for light-weight messages buses. There is command-line tooling for zmq.

```
pipx install zeroless-tools
ffplay -f lavfi -i  'color, zmq' &
echo 'color color orange' | zeroclient 5555 req
echo 'color color purple | zeroclient 5555 req
kill %
```

This example installs tooling for use on the command-line using pipx. We the start playing a filter that outputs solid color ans uses zmq in the filter chain allowing this color to be modified. We send a message to the color filter that its color should be changed to orange, then purple. By default the zmq protocol listens on port 5555.

`zmq` supports different modes of message passing - we seem to need to use the request protocol, indicated by `req` when we use zeroclient.

## Create a remote control for a video being played

```
ffplay -f lavfi -i 'sine=volume=sin(t)'
```

<a name="features"> </a>
# Index of language features
At its core, FFmpeg applies filters on input, feeds the result into other filters, and then produces output.
There are more filters available than this cookbook could cover. You can [list filters](#list) from the command line.

Filters take a number of inputs and outputs. [Source filters](#source) have no inputs or outputs. Filters take [parameters](#params) which are expressed in the form [`key=value`](#params) and [separated by colon (`:`) characters](#colons). Parameters have an order and you can [omit the parameter name](#omit-name) if parameters are given in this order. For some filters, for some parameters you can use [expressions](#expressions-topic) to calculate a value based on things like the width and height of the frame and the timestamp.

You can connect up filters that take one input stream and have one output stream [using ,](#comma). Alternatively you can create a named stream and write to it by putting this is [square brackets after an expression](#output-filter).

You can then run separate filters in parallel by [separating them with a ;](#parallel-stream). When doing so, it is often necessary to label streams, both for [output](#output-stream) and [input](#input-stream). Some filters such as [concat](#concat-stream) take multiple inputs and can produce multiple output.

# Filters
<a name="filters-abstract"> </a>
There are more filters than we can complete document here and there is already reference document. You can see documentation for filters with `man ffmpeg-filters` or [via the ffmpeg website](https://ffmpeg.org/ffmpeg-filters.html). There is a [syntax for expression parameters to filters](https://ffmpeg.org/ffmpeg-filters.html).

You can [show help for a filter](#filter-help) and [list as filters](#list-filters).

<a name="expressions-topic"> </a>
# Expressions
Some filters support [programmatic expression(doc)](https://ffmpeg.org/ffmpeg-utils.html#toc-Expression-Evaluation) in some of their parameters. Filters tend to decide for themselves how they handle expressions, for example [drawtext has a different expression language (example)](#drawtext) using a different syntax, but most of the time if a filter's parameter use expressions it is using this language - but with a different set of variables ([ex1](#variables-1), [ex2](#variables-2)).

You can see the documentation for the shared parts of the expression language (such as functions) with [`man ffmpeg-utils`](#man-pages)

Expressions are useful for "continuous data", but you can add discrete behaviour using the `if` function. For discrete behaviour, [commands](#commands-examples) can be more useful. Just as a filter decides for itself if an expression can be used for a parameter, it decides which of its parameters can be updated with commands..

Examples of parameters supporting expressions include [drawtext's x](#drawtext) and [aevalsrc's expr](#aevalsrc)


It may be useful to understand some [programming concepts](#programming) to use expressions , such as *escaping*, *variables* and *functions*.


<a name="documentation"> </a>
# Documentation
<a name="man-pages"> </a>
## Manual pages
Manual pages run through the command `man` from the [command-line](#programming) are a commonly used way to provided easy to access documentation from the command-line.

FFmpeg provides various manual pages. You can show these with the command `apropos ffmpeg`. There is online documentation that corresponds to each man page which we link to here:

* [ffmpeg (doc)](https://ffmpeg.org/ffmpeg.html)
* [ffmpeg-all (doc)](https://ffmpeg.org/ffmpeg-all.html)
* [ffmpeg-bitstream-filters (doc)](https://ffmpeg.org/ffmpeg-bitstream-filters.html)
* [ffmpeg-codecs (doc)](https://ffmpeg.org/ffmpeg-doecs.html)
* [ffmpeg-devices (doc)](https://ffmpeg.org/ffmpeg-devices.html)
* [ffmpeg-filters (doc)](https://ffmpeg.org/ffmpeg-filters.html)
* [ffmpeg-formats (doc)](https://ffmpeg.org/ffmpeg-formats.html)
* [ffmpeg-protocols (doc)](https://ffmpeg.org/ffmpeg-protocols.html)
* [ffmpeg-resampler (doc)](https://ffmpeg.org/ffmpeg-resampler.html)
* [ffmpeg-scaler (doc)](https://ffmpeg.org/ffmpeg-scaler.html)
* [ffmpeg-utils (doc)](https://ffmpeg.org/ffmpeg-utils.html)

## Web pages
FFmpeg provides [reference documentation online](https://ffmpeg.org/ffmpeg.html).

> See [alternatives](#alternatives) for documentation similar to this.

<a name="zero-index"> </a>
<a name="programming" ></a>
# General programming and command-line knowledge
FFmpeg is a command-line tool written for the kind of user that uses command-line tools. This has certain affordances and norms which FFmpeg will assume and make use. Completely covering concepts related to general computer use would distract from the article and take time. However, I can provide links to certain that someone more interested in FFmpeg from an artistic or practical angle might not know.

Once you have derived some value from this guide, you might like to review all these concepts to speed up your understanding of the rest of the material.

* [Escaping](https://en.wikipedia.org/wiki/Escape_character). Used in [audio filters](#audio-filters) and as part of the [expression language (example)](#expression-example) [(topic)](#expressions-topic).
* FFmpeg's [expression language (example)](#expression-example) makes use of [variables](https://en.wikipedia.org/wiki/Variable_\(computer_science\)) and [functions](https://en.wikipedia.org/wiki/Function_\(computer_programming\))
* FFmpeg provides a [command-line interface](https://en.wikipedia.org/wiki/Command-line_interface) which is often run from a [shell](https://en.wikipedia.org/wiki/Unix_shell), commonly [bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)).
* [Linux](https://en.wikipedia.org/wiki/Linux) is an operating system that you can use on your system which is often easier to program for but may lack commercial support
* [man](https://en.wikipedia.org/wiki/Man_page) is a command line tool available on many operating systems like Linux and Mac that can be used to quickly obtain documentation from the command-line
* [Zero-based based numbering](https://en.wikipedia.org/wiki/Zero-based_numbering). Some filters such as [xstack] start counting from 0 for some values.
* [printf](https://en.wikipedia.org/wiki/Printf) is a way of formatting data into text used in C as well as other languages.

<a name="av-concepts"> </a>
# General audio visual concepts
This guide assumes understanding of certain concepts related to images and videos. We will note cover this material but this provides a glossary.

* [Aspect ratios (wiki)](https://en.wikipedia.org/wiki/Aspect_ratio_(image)) describes the ratio of the width of an image or video to its height. When [scaling (example)](#scale) you may wish to maintain the aspect ratio.
* [Pixels](https://en.wikipedia.org/wiki/Pixel) an image consists of a grid of pixels each of which is a certain colour.
* [Coordinate](https://en.wikipedia.org/wiki/Coordinate_system) pixels in an image can be labelled with horizontal distance from the left edge and the vertical image from the top of the image.

# Support
If you found this guide helpful you can incentivize me to work it and similar things by [donating to my ko-fi](https://ko-fi.com/c/91cb0010ae).

You could also [book an online tutorial on this material](https://ko-fi.com/c/6ee072d36c).

Alternatively, simply reading [other documentation I have written](https://readwithai.substack.com/p/my-command-line-documentation) supports my (varied) work.

# About me
I am **@readwithai**. I make tools for research, reading and productivity - sometimes with [Obsidian](https://readwithai.substack.com/p/what-exactly-is-obsidian).

I am interested in `FFmpeg` because I am [a fan of the command-line](https://readwithai.substack.com/p/my-productivity-tools) and occasionally do image editing and video-editing.

If you liked this this guide you might like to:

1. Have a look at some of [my productivity and command line tools](https://readwithai.substack.com/p/my-productivity-tools);
2. Read [my review of note taking in Obsidian](https://readwithai.substack.com/p/note-taking-with-obsidian-much-of); or
3. Read my [cookbook for the Obsidian note taking tool](https://open.substack.com/pub/readwithai/p/obsidian-plugin-repl-cookbook) and my open source Emacs inspired automation plugin [Plugin REPL](https://github.com/talwrii/plugin-repl).

You can follow me on [X](https://x.com/readwithai) where I write about productivity tools like this besides other things, or my [blog](https://readwithai.substack.com/) where I write more about reading,  research, note taking and Obsidian.
