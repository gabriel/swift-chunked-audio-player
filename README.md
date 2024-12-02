
# ChunkedAudioPlayer

Simple audio player for sync / async chunked audio streams.

[![CI](https://github.com/mihai8804858/swift-chunked-audio-player/actions/workflows/ci.yml/badge.svg)](https://github.com/mihai8804858/swift-chunked-audio-player/actions/workflows/ci.yml) [![](https://img.shields.io/endpoint?url=https%3A%2F%2Fswiftpackageindex.com%2Fapi%2Fpackages%2Fmihai8804858%2Fswift-chunked-audio-player%2Fbadge%3Ftype%3Dswift-versions)](https://swiftpackageindex.com/mihai8804858/swift-chunked-audio-player) [![](https://img.shields.io/endpoint?url=https%3A%2F%2Fswiftpackageindex.com%2Fapi%2Fpackages%2Fmihai8804858%2Fswift-chunked-audio-player%2Fbadge%3Ftype%3Dplatforms)](https://swiftpackageindex.com/mihai8804858/swift-chunked-audio-player)


## Installation

You can add `swift-chunked-audio-player` to an Xcode project by adding it to your project as a package.

> https://github.com/mihai8804858/swift-chunked-audio-player

If you want to use `swift-chunked-audio-player` in a [SwiftPM](https://swift.org/package-manager/) project, it's as
simple as adding it to your `Package.swift`:

``` swift
dependencies: [
  .package(url: "https://github.com/mihai8804858/swift-chunked-audio-player", from: "1.0.0")
]
```

And then adding the product to any target that needs access to the library:

```swift
.product(name: "ChunkedAudioPlayer", package: "swift-chunked-audio-player"),
```

## Overview

`ChunkedAudioPlayer` uses the following approach to stream real time audio:
  * Parse [`AudioStreamBasicDescription`](https://developer.apple.com/documentation/coreaudiotypes/audiostreambasicdescription) and split data chunks into audio packets using [`AudioFileStreamOpen(_)`](https://developer.apple.com/documentation/audiotoolbox/1391498-audiofilestreamopen) and [`AudioFileStreamParseBytes(_)`](https://developer.apple.com/documentation/audiotoolbox/1391492-audiofilestreamparsebytes)
  * Convert audio packets into [`CMSampleBuffer`](https://developer.apple.com/documentation/coremedia/cmsamplebuffer)
  * Enqueue and play the sample buffers using [`AVSampleBufferAudioRenderer`](https://developer.apple.com/documentation/avfoundation/avsamplebufferaudiorenderer) and [`AVSampleBufferRenderSynchronizer`](https://developer.apple.com/documentation/avfoundation/avsamplebufferrendersynchronizer)

## Quick Start

* Create an instance of `AudioPlayer`:
```swift
private let player = AudioPlayer()
```
* Get the audio data stream (can be either `AsyncThrowingStream` or `AnyPublisher`):
```swift
let stream = AsyncThrowableStream<Data, Error> = ...
```

* Start playing the audio stream:

```
// type parameter is optional, but recommended (if the stream type is known)
player.start(stream, type: kAudioFileMP3Type)
```

* Listen for changes:

```swift
player.$currentState.sink { state in
  // handle player state
}.store(in: &bag)

player.$currentRate.sink { rate in
  // handle player rate
}.store(in: &bag)

player.$currentDuration.sink { duration in
  // handle player duration
}.store(in: &bag)

player.$currentTime.sink { time in
  // handle player time
}.store(in: &bag)

player.$currentError.sink { error in
  if let error {
    // handle player error
  }
}.store(in: &bag)
```

* Control playback:

```swift
// Set stream volume
player.volume = 0.5

// Set muted
player.isMuted = true

// Set stream rate
player.rate = 0.5

// Pause current stream
player.pause()

// Resume current stream
player.resume()

// Stop current stream
player.stop()

// Rewind 5 seconds
player.rewind(CMTime(seconds: 5.0, preferredTimescale: 1000))

// Forward 5 seconds
player.forward(CMTime(seconds: 5.0, preferredTimescale: 1000))

// Seek to specific time
player.seek(to: CMTime(seconds: 60, preferredTimescale: 1000))
```

* SwiftUI Support

`AudioPlayer` conforms to `ObservableObject` so it can be easily integrated into SwiftUI `View` and automatically update the UI when properties change:
```swift
struct ContentView: View {
  @ObservedObject private var player = AudioPlayer()

  var body: some View {
    Text("State \(player.currentState)")
    Text("Rate \(player.currentRate)")
    Text("Time \(player.currentTime)")
    Text("Duration \(player.currentDuration)")
    if let error = player.currentError {
        Text("Error \(error)")
    }
  }
}
```

## License

This library is released under the MIT license. See [LICENSE](LICENSE) for details.
