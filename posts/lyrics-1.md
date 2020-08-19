> :Hero src=/img/lyrtube.jpg,
>       leak=96px

> :Title shadow=0 0 8px black, color=white
>
> Playing .lrc files in the browser

> :Author src=github

<br>

In this post I'm reminiscing about an old project. In fact, I believe it was the first project I completed after I graduated from high-school.

Some people like to see the finished product first. If this is you, you can see this project in action at [lyrtube.joshwalsh.me](https://lyrtube.joshwalsh.me/). If you'd prefer some suspense, read on, I've included another link near the end of the post.

_**SYNDICATED CONTENT:** This post originally appeared on blog.joshwalsh.me. You can find the original [here](https://blog.joshwalsh.me/lyrics-1/). Some formatting and content has been removed in order to fit the `coding.blog` platform._

---

# Motivation

I find music inspiring. Music moves me in a way that other forms of art are unable to. Poetry has never interested me, but if you add in some instrumentation then suddenly it attains great significance, at least in my view.

I'm also rarely content to merely consume, I always want to contribute in some way. Unfortunately for me I am completely devoid of musical talent, so I find myself frustrated at not being able to participate in something that's so meaningful & important to me. This hasn't stopped me from trying, but I'm sure anyone who's heard me sing would rather it had.

In my final year of high school I was starting to collect a few FLACs and Windows Media Player just wasn't cutting it anymore, so I switched to [foobar2000](http://www.foobar2000.org/). I had a lot of fun customising it to look just how I wanted.

![My foobar2000 theme](/img/fb2k.png)

While setting up my fb2k theme I discovered a component called Lyrics Show Panel 3. This component automatically searches the internet for lyrics to songs and displays them. But of interest to me was the fact that it supported timestamped lyrics, lyrics that are synchronised to the song. I fell in love with this functionality, and pretty soon I was creating my own timestamped lyrics files for whatever mildly-obscure songs took my fancy. This scratched the itch that I had to contribute, I felt that by syncing lyrics I was enhancing the songs.

But the true reward of creating things is in sharing those things with others, and it was difficult to convince friends that they should install fb2k and LSP3 in order to view my lyrics. I needed an easy way to share my creations. I wanted to be able to just send people links and not require them to do anything but click on those links.

# Proof-of-concept

Timestamped lyrics are (usually) stored in the [LRC file format](https://en.wikipedia.org/wiki/LRC_(file_format)). This is a text-based format, so it's pretty easy to parse. I threw together a proof of concept that displayed timestamped lyrics within the browser.

The logic I used for parsing the LRC files was simple. It splits the file into lines, and then it uses Regular Expressions to work out what each line contains. If the line contains a metadata tag, the metadata tag is read and the value is stored. If it contains one or more timestamps, the rest of the line is treated as lyrics and saved at the given timestamps. Otherwise, the line is ignored.

At the time I was obsessed with Canvas, (which had only recently started being supported in browsers) so I used that to draw the lyrics. I mimicked the style of Lyrics Show Panel 3: the lyrics are shown line-by-line with the currently active line vertically-centred and highlighted.

Since I'm not interested in being sued for distribution of copyrighted content, I chose not to host any of the music myself. Instead I embedded a YouTube upload of the song on the page. This also saved me having to make my own transport controls.

Something I noticed while setting this up was that the `getCurrentTime()` function exposed by the YouTube iframe Player API is only accurate if you call it infrequently. If you call it once per second it returns a result accurate to within a few milliseconds. But if you call it 60 times per second you get results that are all over the place. This initially caused the lyrics to jump and stutter. I resolved this by implementing my own timer and periodically updating it based on the time reported by YouTube. Occassionally these updates cause a small visible jump in the lyrics, but the result still looks much better than it did at first. It would be possible to eliminate these jumps by applying some smoothing to the updates, but I deemed that the issue was minor enough to ignore.

# Abandoned dreams

I was very proud of this proof-of-concept and thought it was going to be the Next Big Thing. This was around the time that Rap Genius (now Genius.com) had just exploded onto the internet (shortly before they got [penalised by Google for using blackhat SEO techniques](https://en.wikipedia.org/wiki/Genius_(website)#Google_search_penalty)) so perhaps that's why I believed I had a chance at viral success with this project. I developed a simple PHP+MySQL backend for storing the lyrics in and launched LyrTube.com. I planned to turn the site into a "social network for timestamped lyrics" where users could share their own .lrc files and suggest changes to those submitted by others. I put the project on hold due to not having a suitable authentication system for it (a recurring problem for me, maybe I'll write a blog post about it at some point) and never came back to it.

Now, in 2019, I've grown tired of wasting money hosting this dead site. It's still something that I'm proud of (and nostalgic about) so I don't want to just delete it. Instead I've removed all the PHP code and converted the site into something static that I can just host from AWS S3. [lyrtube.joshwalsh.me](https://lyrtube.joshwalsh.me/) is the final resting place of LyrTube, a website I once had grand plans for. I intend to let the old lyrtube.com domain registration lapse peacefully.

# Conclusion

It may seem like the story ends here, with me giving up on the LyrSync project. But that's not the case! Just because I've decided not to build a social network for lyrics, doesn't mean my obsession with them has diminished.

You can read about the next manifestation of my fixation in [part 2](/lyrics-2/) of this series of posts.

---

<br>

> :Author src=github

> :ToCPrevNext
