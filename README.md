# ffmpeg cookbook
This is a cookbook for the audio-video command line tool, [ffmpeg](https://www.ffmpeg.org/), because I found it difficult to work out how ffmpeg worked when I first used it.

This cookbook tries to build up an understanding of ffmpeg filters through examples. If you can't adapt an example you may be able to start learning on an earlier example.

# Alternatives and prior works
There are a few other cookbooks. [ffmprovisr](https://amiaopensource.github.io/ffmprovisr/) is quite complete, but I want to build up from more basic resources. There is [quite complete documentation on filters](https://ffmpeg.org/ffmpeg-filters.html#Filtering-Introduction) haileys has [the beginnings of a cookbook](https://github.com/haileys/ffmpeg-cookbook) but with only a couple of examples.

## Create 10 seconds of silence
```
ffmpeg -filter_complex 'anullsrc=duration=10s [out]' -map '[out]' -ac 1 silence.wav
```

You can play then silence with `ffplay silence.wav`

This uses a source filter [anullsrc](https://ffmpeg.org/ffmpeg-filters.html#anullsrc) which generates ten seconds of silence (specified through the duration parameter) and then writes this to a stream called '[out]'. The `-map` command specifies stream called `[out]` is used in the output.


You can also run this as: `ffmpeg -filter_complex 'anullsrc=duration=10s' -ac 1 silence.wav` and an the output of the filter is ipplicitly used as the stream for output. But you cannot use `-af` because these filters must have precisely one input and one output and this has no input.

## Play a sine wav at 256 hertz
```
ffplay -f lavfi 'sine=frequency=256:duration=10'
```

This specifies that the format of the input is a `libavformat` filter ("LibAVFIlter"). `ffplay` does not have a `-filter_complex` argument.

Notice how parameters for the source filter `sine` are specified by `=` and then separted by `=`

## Write a sine wave to a file
<a name="write_sine"></a>
```
ffmpeg -filter_complex 'sine=frequency=256:duration=10' sine.wav
```

Note how this compares to the previous example. This shows how you can convert between filters that play a sounds and those that create a file.

## Combine two sine waves into a chord.
```
ffplay -f lavfi 'sine=frequency=256:duration=10 [one]; sine=frequency=512:duration=5 [two]; [one][two]amix'
```
This creates two streams `[one]` and `[two]` which are then mixed together into the output stream by `amix` which takes two inputs and mixes them together.

## List available filters
```
ffmpeg -filters
```

## Show the parameters for a filter
```
ffmpeg --help filter=name
```

## Decrease the volume of an audio file
```
ffmpeg -filter_complex `sine=frequency=512:duration=10` sine.wav
ffmpeg -i sine.wav -af 'volume=volume=0.5' quiet-sine.wav
```

This creates a file with a sine wav and then use the volume filter (which has one input and one output) to decrease the volume to 50%. Because we are using a filter that has precisely one input and output we can use `-af` rather than `-filter_complex`. But we could also use filter_complex.


## Concatenate two files.
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
