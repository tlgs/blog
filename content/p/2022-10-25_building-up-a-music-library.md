+++
title = 'Slowly building up a music library, concurrently'
customcss = 'building-up-a-music-library.css'
+++

[youtube-dl](http://ytdl-org.github.io/youtube-dl/) is one of my favorite pieces
of software; It allows you to download videos from Youtube and
[many, many other sites](http://ytdl-org.github.io/youtube-dl/supportedsites.html).
One of the things you can do with it is extract the audio portion of a video and encode
it in a desired format.

Nowadays most artists upload complete albums as playlists to Youtube,
no doubt in order to stay relevant in the post-CD era of the music industry.
You can probably tell where I'm going with this:
you can have a music album ready to go with as little as `youtube-dl -x <PLAYLIST_LINK>`.

This approach downloads and post-processes videos one-by-one, which can take
**a long time**. youtube-dl doesn't support concurrent downloads out-of-the-box,
but it's trivial to write a little wrapper that does this;
I've had something like this in my `.bashrc` for sometime:

```bash
ytdl-playlist() {
  options=(--no-progress -x --audio-format flac --add-metadata --)
  youtube-dl --get-id "$1" | xargs -n 1 -P "${2:-5}" youtube-dl "${options[@]}"
}
```

That's probably the most useful Bash function I've written yet.
It's very powerful, but still has a couple of annoying shortcomings:
the output of each process is printed sequentially, leaving the user's
screen filled with jumbled up messages, and thus making it difficult to
have a good idea of the overall progress.
More importantly, youtube-dl will sometimes fail with `HTTP Error 403: Forbidden`,
forcing the user to manually re-download the missing track[^1].
This happens once per album on average and I'm too lazy to track down the
misbehaved webpage and re-run the command, so I thought I'd spend
a [disproportionate amount of time](https://xkcd.com/1205/) coming up with
something better™ and writing this up.

[^1]: This is obviously also the case when using `youtube-dl` directly,
      though it will abort on the first encountered error unless `-i` passed as an option.

## Cue the gophers

Whenever concurrency is at the heart of a problem, Go is a good candidate
when it comes to a programming language choice;
It's always a lot of fun to work out a solution using its concurrency primitives.
(Of course I could stick with Bash and hack out some uncomprehensible and
unmaintanable script from hell, but this time I was looking to sharpen my Go hammer.)
I'll continue by sharing a couple of relevant notes on design and programming techniques.

A watered-down requirement specification, along with some design thoughts follows:

- **Concurrent** - should perform downloads concurrently.
  Let's use goroutines to spawn and manage youtube-dl processes, and channels
  to communicate with the main control loop;
  The [Worker Pool](https://gobyexample.com/worker-pools) pattern provides a
  simple approach to model task queues like this.
  The number of spawned worker goroutines should be configurable.
- **Fault-tolerant** - should handle errors gracefully and retry failed tasks.
  User should be able to configure the number of allowed retries and duration
  between attempts.
- **Intelligible** - the user should be able to effortlessly understand
  the overall status and current progress.
  Displaying the status of each individual task and relaying the last line of
  output of the youtube-dl process (or any internal error)
  should be more than enough.

As stated above, the Worker Pool pattern is sufficiently powerful
to handle our concurency needs, so we're mainly concerned with coming up with
_something_ that defines a task's status and issues control actions accordingly.
[State machines](https://en.wikipedia.org/wiki/Finite-state_machine)
are nifty abstract computational models that fit this problem nicely -
it's simple to lay out the possible task states and map them to actions
(queue task, wait for a task to finish running, re-queue a task if it failed, etc.):

```goat
    +---------+     +--------+     +---------+     +-----------+
--->| Pending +---->| Queued +---->| Running +---->| Succeeded |
    +---------+     +--------+     +----+----+     +-----------+
         ^                              |
         |                              |
         |          +--------+          |
         o----------+ Failed |<--------'
                    +--------+
```

[Enumerated constants](https://go.dev/doc/effective_go#constants)
are very effective constructs when dealing with state machines in Go;
By defining a type, we attach contextual meaning to fields and variables, and
ensure comparisons and transitions are performed _safely_.
As an added bonus, it's easy to map the above states to quick-to-interpret
values to display in our user interface:

```go
type Status uint8

const (
	StatusPending Status = iota
	StatusQueued
	StatusRunning
	StatusSucceeded
	StatusFailed
)

func (s Status) String() string {
	switch s {
	case StatusPending:
		return "💤"
	case StatusQueued:
		return "📬"
	case StatusRunning:
		return "🚀"
	case StatusSucceeded:
		return "💯"
	case StatusFailed:
		return "💣"
	default:
		return ""
	}
}
```

By using well-defined state transitions[^2],
the main control loop ends up being rather simple:

```go
for completed := 0; completed < len(tracks); {
    update := <-updates
    id := update.Id

    job := jobs[id]
    job.Status = update.Status
    job.Message = update.Message

    switch update.Status {
    case StatusSucceeded:
        completed++

    case StatusFailed:
        failures[id]++
        if failures[id] > *nRetries {
            completed++
            break
        }

        job.Status = StatusPending
        time.AfterFunc(
            time.Duration(*retryInterval)*time.Second,
            func() {
                queue <- id
                // send update setting job status to Queued
            },
        )
    }

    // re-draw user interface
}
```

[^2]: Most transitions like `Queued ⇒ Running`, and `Running ⇒ Succeeded` are
      handled by the worker goroutine not seen here.

### The rest of the owl

There isn't much else to talk about, it's meant as a _thin wrapper_ after all.
You're welcome to take a peek at the 
[end result](https://github.com/tlgs/dotfiles/blob/master/bin/ytdl-playlist.go) -
it ends up being about 50 times longer than the original,
but satisfies (at least to an acceptable level) the requirements set above.

Here's a snapshot of what the terminal screen looks like as _someone_
downloads Red Hot Chili Pepper's
[Return of the Dream Canteen](https://www.youtube.com/playlist?list=OLAK5uy_ke4zvQhGW2BDith3tH_fh_uMaoGunxkHo):

```console
$ ytdl-playlist -jobs 8 OLAK5uy_ke4zvQhGW2BDith3tH_fh_uMaoGunxkHo
Completed: 7/17
💯 [ffmpeg] Adding metadata to 'Red Hot Chili Peppers - Tippa My Tongue (Officia…
🚀 [download] Destination: Red Hot Chili Peppers - Peace & Love (Official Audio)…
💯 [ffmpeg] Adding metadata to 'Red Hot Chili Peppers - Reach Out (Official Audi…
🚀 [download] Destination: Red Hot Chili Peppers - Eddie (Official Audio)-pXMEXC…
📬 exit status 1
💯 [ffmpeg] Adding metadata to 'Red Hot Chili Peppers - Bella (Official Audio)-3…
💯 [ffmpeg] Adding metadata to 'Red Hot Chili Peppers - Roulette (Official Audio…
💯 [ffmpeg] Adding metadata to 'Red Hot Chili Peppers - My Cigarette (Official A…
💯 [ffmpeg] Adding metadata to 'Red Hot Chili Peppers - Afterlife (Official Audi…
💯 [ffmpeg] Adding metadata to 'Red Hot Chili Peppers - Shoot Me A Smile (Offici…
🚀 [download] Destination: Red Hot Chili Peppers - Handful (Official Audio)-XGc5…
🚀 [download] Destination: Red Hot Chili Peppers - The Drummer (Official Music V…
🚀 [download] Destination: Red Hot Chili Peppers - Bag of Grins (Official Audio)…
🚀 [download] Destination: Red Hot Chili Peppers - La La La La La La La La (Offi…
🚀 [download] Destination: Red Hot Chili Peppers - Copperbelly (Official Audio)-…
🚀 [download] Destination: Red Hot Chili Peppers - Carry Me Home (Official Audio…
💤 exit status 1
```

<img id="gopher"
  src="/images/gopher.svg"
  alt="A gopher wearing a pirate hat and holding a wrench"
  width="150"
  height="150"
  title="You Wouldn't Download a Car">
