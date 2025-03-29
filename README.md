# ffmpeg cookbook
*@readwithai - [X](https://x.com/readwithai) - [blog](https://readwithai.substack.com/)*

This is a cookbook for the audio-video command-line tool, [ffmpeg](https://www.ffmpeg.org/).
I created it because I found working out what ffmpeg was doing from snippets slightly too difficult.

This cookbook tries to build up an understanding of ffmpeg filters through examples.

I'm trying to do a few things with this cookbook:

1. Every example can be run. Examples are neither abstract, nor does you need you to provide your own data
1. Examples link to earlier examples which do similar (but simpler) things. If the you cannot adapt an example to a use case, you can find something more basic which you can start working on
1. Where possible, examples link to documentation
1. Examples are, initially, ordered in terms of complexity rather than based on the task you are trying to perform

There is an [index of features](#features) which goes through different ffmpeg features linked to recipes. `ffmpeg` has more features than a cookbook can cover completely but provides the ability to [list filters](#list) and [display their parameters](#parameters) as well as providing [terse but useable reference documentation](https://ffmpeg.org/ffmpeg-filters.html).

# Attribution
Thanks to [Ventz Petkov](https://github.com/ventz) for providing me with [various additional recipes](https://github.com/talwrii/ffmpeg-cookbook/issues/1). This is particularly valuable as I am not (at the time of writing) an expert in ffmpeg.

If you think a particularly recipe belongs in this guide feel free to [add it as an issue](https://github.com/talwrii/ffmpeg-cookbook/issues). It may take me a little time to add recipes as I want to keep the guide accessible to beginners, so bear this in mind when considering the whether submitting me recipes is worth your time!

# Alternatives and prior work
There are a few other cookbooks. [ffmprovisr](https://amiaopensource.github.io/ffmprovisr/) is quite complete, but doesn't feel like it starts at the beginning. haileys has [the beginnings of a cookbook](https://github.com/haileys/ffmpeg-cookbook) but with only a couple of examples. There is [quite complete documentation on filters](https://ffmpeg.org/ffmpeg-filters.html#Filtering-Introduction).

# Introduction
This introduction tries to cover a broad range of tasks and features building up an understanding of ffmpeg it is not complete.

## Create 10 seconds of silence
<a name="source"> </a>
<a name="output-filter"> </a>

```
ffmpeg -filter_complex 'anullsrc=duration=10s [out]' -map '[out]' -ac 1 silence.wav
```

You can play the silence you have created with `ffplay silence.wav`.

This uses the [source filter](https://ffmpeg.org/ffmpeg-filters.html#toc-Filtering-Introduction), [anullsrc](https://ffmpeg.org/ffmpeg-filters.html#anullsrc) which generates ten seconds of silence (specified through the `duration` parameter) and then writes this to a stream labelled '[out]'. The `-map` command specifies stream labelled `[out]` is used in the output.

You can also run this as: `ffmpeg -filter_complex 'anullsrc=duration=10s' -ac 1 silence.wav` and an the output of the filter is ipplicitly used as the stream for output. But you cannot use `-af` because these filters must have precisely one input and one output and this has no input.

## Play a sine wav at 256 hertz for ten seconds
<a name="play"> </a>
<a name="sine-play"> </a>
```
ffplay -f lavfi 'sine=frequency=256:duration=10'
```

This command specifies that the format of the input is a `libavformat` filter ("LibAVFIlter"). `ffplay` does not have a `-filter_complex` argument but you can use this instead for complex filters.

The parameters for the source filter, [`sine`](https://ffmpeg.org/ffmpeg-filters.html#sine), are specified by `=` and separted by `:`.

See also: [ffplay vs ffmpeg](#play)
## Write a sine wave to a file
<a name="write_sine"></a>
```
ffmpeg -filter_complex 'sine=frequency=256:duration=10' sine.wav
```

Note how this compares to the previous example. This shows how you can convert between filters that play media and those that create a file containing the media.

## Combine two sine waves into a chord.
```
ffplay -f lavfi 'sine=frequency=256:duration=10 [one]; sine=frequency=512:duration=5 [two]; [one][two]amix'
```
This creates two streams, `[one]` and `[two]`, which are then mixed together into the output stream by `amix`. `amix` which takes two streams as input and mixes them together.

## List available filters
<a name="list"> </a>

```
ffmpeg -filters
```

This will list all available filters. The output indicates what type of filter a filter is. For examples `|->A` means a filter is an audio source, `A->A` means that the filter takes audio as input and writes it to output etc.

## Show the parameters for a filter
<a name="parameters"> </a>

```
ffmpeg --help filter=name
```

## Decrease the volume of an audio file
```
ffmpeg -filter_complex `sine=frequency=512:duration=10` sine.wav
ffmpeg -i sine.wav -af 'volume=volume=0.5' quiet-sine.wav
```

This creates a file containing a sine wav and then use the `volume` filter to decrease the volume to 50% of the original volumen. Because we are using a filter that has precisely one input and output, we can use `-af` rather than `-filter_complex`, but `-filter_complex` would still work.

## Concatenate two audio files together
<a name="concat"> </a>

```
ffmpeg -filter_complex 'sine=frequency=512:duration=2' sine1.wav
ffmpeg -filter_complex 'sine=frequency=256:duration=5' sine2.wav
ffmpeg -i sine1.wav -i sine2.wav -filter_complex '[0][1]concat=v=0:a=1'  -ac 1  concat.wav
```

We [create two sine waves](#write-sine) like in the prevoius recipes. We then combine together the first (`[0]`) and second (`[1]`) inputs with the `concat` filter. We must specify that concat outputs no video streams and one audio stream because these are not the defaults: concat works with one video stream by default.


Here is the documentation for concat.

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

## Play ten seconds of red
```
ffplay -f lavfi color=color=red:duration=10s
```

See also: [ffplay vs ffmpeg](#play)

## Create an image consisting of solid red
<a name="image"> </a>

```
ffmpeg -filter_complex 'color=color=red' -frames:v 1  red.png
```

Here we specify that we are reading one frame with `-frames:v 1`.

## Write ten seconds of red to a file
```
ffmpeg -filter_complex 'color=color=red:duration=10s'  red.mp4
```

## Create a video that fades between two colours
```
ffplay -f lavfi 'color=color=red:duration=5s [red]; color=color=blue:duration=5s [blue]; [red][blue]xfade=duration=5s'
```

We create two streams, one consisting of solid red, the other of solid blue. We then feed both these staerms into the `xfade` filter which "cross-fades" between the two over 5 seconds.

See also: [ffplay vs ffmpeg](#play)

## Create a video that counts up to 10
<a name="comma"> </a>
<a name="timestamp"> </a>

```
ffplay -f lavfi -i  'color=color=white, drawtext=text=%{pts}'
```

This uses the [drawtext filter's](https://ffmpeg.org/ffmpeg-filters.html#drawtext-1) expression language to show the current timestamp in each frame. `%{pts}` is [expanded](https://ffmpeg.org/ffmpeg-filters.html#Text-expansion) in each frame to the number of seconds into the frame.

See the [section on positioning text](#positioning)

## Create a video with the phrase "hello world" on a white background
<a name="text"> </a>

```
ffmpeg -filter_complex  'color=color=white:duration=10s, drawtext=text=hello:fontsize=20' out.mp4
```

`drawtext` takes a video stream as input and draws text on top of it. Note the use of `,` to feed the output of one stream into the next.

We could, however, have used labels instead as we did before when [concatenating audio streams](#concat), like so:

```
ffmpeg -filter_complex  'color=color=white:duration=10s [one]; [one] drawtext=text=hello:fontsize=20' out.mp4
```

## Trim a video to five seconds duration
```
ffmpeg -filter_complex 'color=color=white, drawtext=text=%{pts}, trim=duration=10s' count-to-ten.mp4
ffmpeg -i count-to-ten.mp4 -vf 'trim=duration=5s' count-to-five.mp4
```

First we create a video which shows the timestamp each frame and is ten seconds long. We then apply the `trim` filter to limit this to ten seconds.

## Turn a static image into a five second video of the image
<a name="static-image"> </a>

```
ffmpeg -filter_complex 'color=color=white, drawtext=text=hello:fontsize=40' -v:frame 1 static.png
ffmpeg -i static.png  -filter_complex 'nullsrc=duration=5s [vid]; [vid][0]overlay' out.mp4
```

First we [create an image](#image) using the `-vframe` option. We then create a 5 second empty video using the `nullsrc` filter and overlay the static image on it.

## Concatenate two videos together
```
ffmpeg -filter_complex 'color=color=white:duration=5s, drawtext=text=%{pts}' count-up.mp4
ffmpeg -filter_complex 'color=color=white:duration=5s, drawtext=text=%{pts}, reverse' count-down.mp4
ffmpeg -i count-up.mp4 -i count-down.mp4  -filter_complex '[0][1]concat' concat.mp4
```

First we create two videos one that [counts up to ten seconds](#timestamp), and then the reverse of this using the [reverse](https://ffmpeg.org/ffmpeg-filters.html#reverse) filter.

We then use the concat filter (as we did for [audio](#concat)) but this time we do not need to use concat - since the default output is a single video stream and no audio stream.

## Add a static introduction image to the beginning of a video
```
ffmpeg -filter_complex 'color=color=white, drawtext=text=hello:fontsize=40' -v:frame 1 static.png
ffmpeg -filter_complex 'color=color=red:duration=5s [red]; color=color=green:duration=5s [green]; [red][green]xfade=duration=5s' video.mp4
ffmpeg  -i static.png -i video.mp4 -filter_complex 'nullsrc=duration=2s, [0]overlay [intro]; [intro][1]concat' with-title.mp4
```

First we create a static image with the word hello and a video which fades from red to green. Then we combine the two [overlaying the static image](#static-image) on a 2 second video and call it intro. This is concatenated with a video.


# Positioning text
<a name="positioning"> </a>

## Placing text in the middle of the screen
<a name="middle"> </a>

```
ffplay -f lavfi -i 'color=color=white, drawtext=text=hello:x=(main_w - text_w) / 2:y=(main_h-text_h) /2'
```

We [render text using the drawtext filter](#text) as we did in this previous recipe. But we also use the `x` and `y` paramaters that accept [drawtext's own expression language](https://ffmpeg.org/ffmpeg-filters.html#Syntax).

Note that we the division operation, `/`, in the expression. `main_w` represents the video width, `main_h` its height, `text_w` the rendered text's width and `text_h` its height.

See also: [ffplay vs ffmpeg](#play)

## Placing text at difference positions
```
ffplay -f lavfi -i 'color=color=white, drawtext=text=bottom-right:x=main_w - text_w:y=(main_h-text_h), drawtext=text=top-left:x=0:y=0, drawtext=text=top-right:x=main_w-text_w:y=0, drawtext=text=bottom-left:x=0:y=main_h - text_h'
```

This recipe renders text in various positions. See the [previous recipe](#middle) for the meaning of expressions.

See also: [ffplay vs ffmpeg](#play)
# Index of language features
<a name="features"> </a>

At its core, ffmpeg applies filters of input and output in a pipeline of filters.
There are more filters than this cookbook will cover, you can [list filters](#list) from the commad line.

Filters take a number of inputs and outputs. [Source filters](#source) have no inputs or outputs.

You can connect up filters that take one input stream and have one output stream [using ,](#comma). Alternatively you can create a named stream and write to it by putting this is [square brackets after an expression](#output-filter).

You can then run separate filters in parallel by [separating them with a ;](#parallel-stream).


# About me
I am @readwithai. I make tools for research, reading and productivity - sometimes with [Obsidian](https://readwithai.substack.com/p/what-exactly-is-obsidian).

If you liked this this guide you might like to:

1. Have a look at some of [my productivity and command line tools](https://readwithai.substack.com/p/my-productivity-tools); 
2. Read [my review of note taking in Obsidian](https://readwithai.substack.com/p/note-taking-with-obsidian-much-of); or
3. Check out my [cookbook for the Obsidian notetaking tool](https://open.substack.com/pub/readwithai/p/obsidian-plugin-repl-cookbook) and my open source Emacs inspired automation plugin [Plugin REPL](https://github.com/talwrii/plugin-repl).

You can follow me on [X](https://x.com/readwithai) where I write about productivity tools like this besides other things, or my [blog](https://readwithai.substack.com/) where I write more about reading,  research, note taking and Obsidian. 
