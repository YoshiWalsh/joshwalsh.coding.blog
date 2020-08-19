> :Hero src=/img/etoile-banner.jpg,
>       leak=96px

> :Title shadow=0 0 8px black, color=white
>
> One-day build: Immersive animated lyrics webapp

> :Author src=github

<br>

In this post I'll be talking about a project I challenged myself to complete within a single day.

This post is part of a 3 part series. It will make more sense if you read [the first post](/lyrics-1/) first.

Some people like to see the finished product first. If this is you, you can see this project in action at [sandbox.ymindustries.com/etoile-et-toi](http://sandbox.ymindustries.com/etoile-et-toi/). If you prefer the suspense, read on, I've put another link near the end of the post.

_**SYNDICATED CONTENT:** This post originally appeared on blog.joshwalsh.me. You can find the original [here](https://blog.joshwalsh.me/lyrics-2/). Some formatting and content has been removed in order to fit the `coding.blog` platform._

---

# Reincarnation

It turns out that my fascination for timestamped lyrics has not decreased. I discovered this after I finished watching *Kizumonogatari Ⅲ: Reiketsu-hen* (Wound Story 3: Cold Blood) and the end credits started rolling.

![Kizumonogatari Ⅲ Credits](/img/tactical-etoile.gif)

During the credits a song called *étoile et toi (edition le blanc)* plays. The lyrics are in French, but Tactical's fansubs kindly provide a translation. The song was very fitting for the movie, but I'm sure that without the translation it wouldn't have resonated with me to such a degree. I found myself listening to the song repeatedly for the next few days, but each time I would have to skip to the end of the movie so I could enjoy the translated lyrics at the same time. I decided I wanted to be able to view these lyrics wherever I was, and so began a new project.

The obvious choice would've been to just make an LRC file for the song and add it to LyrTube, but I had a few specific goals which ruled out this approach:

 * I wanted to roughly imitate the style Tactical used for the lyrics. It's minimalism & elegance suited the song.
 * I wanted to only display one line of lyrics at a time, to prevent reading ahead. I think this increases enjoyment of the flow of the song.
 * I wanted to synchronise every word, instead of every line. This was because the song is quite slow-paced, so every line would be a bit boring.
 * I wanted to display the original French lyrics and the English translation simultaneously.

I had learned from my mistakes of aiming too big, so I decided that instead of trying to build a platform for lyrics I would just try to build a Single-Serving Site for this one song. This meant I was prioritising development speed over flexibility/maintainability. I challenged myself to develop the site as quickly as I could, preferably within 1 day.

# Development process

## Planning

I still intended to use the LRC file format, but this time I'd be using the [enhanced format](https://en.wikipedia.org/wiki/LRC_(file_format)#Enhanced_format) which supports per-word syncing.

I'd need to implement a way to include both French & English lyrics. The uncommon [simple format extended](https://en.wikipedia.org/wiki/LRC_(file_format)#Simple_format_extended) version of LRC supports multiple voices, but is quite limited. You can only tell the voices apart by gender (not language) and it's not designed for the voices to sing simultaneously. I decided instead to add a new (non-standard) tag to specify voices: `[voice:french]`.

Since one *line* in the song would now have multiple *lines* in the file (one for French and one for English) I decided to change the terminology a little to avoid ambiguity. A *line* in the song would now be referred to as a *card*, since all those words would be displayed fullscreen at once. "Line Time Tags" would now be called "Card Time Tags".

I decided not to use Canvas this time around for several reasons. Using Canvas meant I had to do a lot of text rendering myself. The original LyrTube didn't even support line wrapping, and this time I wanted to be sure my site would work on mobile devices. The web is a different place now to how it was in 2013, and mobile devices are now a very important consideration. By instead using DOM elements, the browser would do all the hard layout work for me.

I sketched out how the JavaScript would work. There would be four phases:

1. Parsing - this would read the LRC file and translate it into a JavaScript object.
2. Rendering - this would read the JavaScript object and create DOM elements from it.
3. Layout - this would perform any expensive calculations about position and size, anything requiring a reflow. It would be re-run whenever the window was resized.
4. Draw - this would run every frame and update the positions/opacitites of all the elements.

The planning didn't take very long, I only spent about 10 minutes on it.

## Syncing

To sync lyrics in the past I'd always used the simple tools provided by Lyrics Show Panel 3. These only support per-line syncing, but now I needed to sync each word. I decided I wasn't going to spend any time looking for other tools, I was going to stick with what I knew. So I needed to find a way to use this tool to do per-word syncing. Here's what I decided on:

1. Write the lyrics in Notepad++ using Unix (LF) line endings.
2. Use find/replace to replace all newline characters '\n' with a colon and a space '; '. This is so we can remember where the line breaks are supposed to be.
3. Use find/replace to replace all spaces ' ' with newlines '\n'. Now every word is on its own line.
4. Convert line endings to Windows (CR LF) since this is what Lyrics Show Panel 3 understands. Copy/paste the lyrics into the Lyrics Show Panel 3 editor.
5. Synchronise the lyrics. Copy/paste the result back into Notepad++.
6. Convert line endings back to the superior Unix (LF) format.
7. Use find/replace to replace all '[' with '<' and all ']' with '>'. Also replace all newlines '\n' with spaces ' '. Now everything's a word instead of a line.
8. Use find/replace to replace any colon and space '; ' with a newline '\n'. Now all the line breaks are back.
9. Use a RegEx find/replace to add a Card Time Tag to the start of each line based on the first Word Time Tag on that line. For readability, the Card Time Tag can go on a different line to card's contents. Replace '`regex›\n<(\d{2}\:\d{2}\.\d{2})>`' with '`regex›\n[\1]\n<\1>`'.
10. Manually touch-up anything that doesn't quite look right.

This worked perfectly, so now the only thing left to do was to add the translation. To do this I duplicated each line of lyrics, prefixing the original with `[voice:french]` and the copy with `[voice:english]`. Then I read through a translation of the lyrics and replaced the French words on the English line with English words, while preserving the word timing tags.

I spent about 1.5 hours between buying the song, downloading the song, syncing the lyrics, and adding the translation. 

## Parsing

Initially I thought of modifying the LyrTube parsing code to handle Word Time Tags, but I quickly realised a problem with this approach. The old parsing code worked by splitting the lyrics file into lines and parsing each line separately, but now a single card might span multiple lines.

I don't really want to talk much about the parsing code since it's a hastily-written hack. It starts out fairly sane (feed the file character-by-character into a Finite State Machine) but then certain states of the FSM read characters on their own, which is pretty gross. If I was writing this again I would be consistent about every single character being fed to the FSM and the FSM not reading anything from the file itself.

But I managed to write the parsing algorithm in just a bit more than 2 hours, so from a development-speed perspective it was a success. (Or at least I was happy with it, since I really have no clue what I'm doing when it comes to parsing.)

## Rendering

The rendering algorithm is very straightforward, it just loops through the parsed object (henceforth called the Abstract Syntax Tree or AST) and creates DOM elements for everything.

But I committed another sin here: I took advantage of JavaScript's looseness and stored data on each created element. Card elements contain a reference to the card AST object, and word elements contain a reference to the word AST object. I stored a few other bits of miscellaneous data too. This data is used later on in the Draw phase. If I was writing this again, I would either mutate the AST to contain references to the elements, or I would have the render function output a new data structure which has references to both the elements and their corresponding AST objects.

This part took about 15 minutes.

## Layout

The layout function serves a couple of purposes. The first is to make sure that on each card, all the voices are the same height. The lines might naturally be different heights if the text wraps in one language but doesn't in another, but this would cause the card to no longer look vertically centred. So the layout function unsets the height on each voice (allowing them to take their natural height), measures the height of each voice, then sets the height of all voices on the card to be the same as the height of the tallest voice.

The other purpose of the layout function is to measure the width of each voice and store it. This information is used to determine the width of the separator line that is displayed between voices.

Writing this function took about 15 minutes.

## Draw

The draw function performs three main duties:

1. Loop through every card and set their opacity, as well as determining whether they are (at least partially) visible.
2. Loop through the words on each visible card and set their position and opacity.
3. Loop through the separators on all visible cards and update their width.

This is quite simple, but I spent quite a lot of time tweaking this to make sure everything looked as nice as possible. I spent about 2 hours in total on this part.

# Results

And that was it, done! All-in-all it took slightly more than 6 hours. You can view the result [here](http://sandbox.ymindustries.com/etoile-et-toi/). It was fun to work on a small project with a tight timeline, I guess that's the appeal of hackathons.

![Result](/img/oneday-etoile.gif)

But that's not where the story ends. My timestamping itch hadn't been scratched, instead my passion for synchronised lyrics had been reignited. These two posts so far have just been providing background information for part 3 (coming soon to coding.blog, or you can read it in advance [on my own blog](https://blog.joshwalsh.me)), where things get a little bit crazy.

---

<br>

> :Author src=github

> :ToCPrevNext
