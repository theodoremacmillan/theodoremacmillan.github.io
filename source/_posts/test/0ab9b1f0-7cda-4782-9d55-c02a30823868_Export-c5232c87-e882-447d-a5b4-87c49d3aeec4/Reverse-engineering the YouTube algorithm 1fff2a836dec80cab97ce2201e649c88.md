# Reverse-engineering the YouTube algorithm

## Motivation

A few years ago, the Wall Street journal published a piece on reverse-engineering the recommendation algorithm of TikTok. They initialized ~100 bots, each with their own hidden interests (one would be interested by forestry, another by astrology) and sent their bots into the TikTok wild, having them scroll past videos they were disinterested by and pause and watch videos that aligned with their interests. They found that across the board their bots were pushed towards more niche content, even bots initialized with a diverse set of interests. You can watch the whole video below.

[https://www.youtube.com/watch?v=nfczi2cI6Cs&t=324s](https://www.youtube.com/watch?v=nfczi2cI6Cs&t=324s)

Its definitely an interesting approach towards studying the interactions between users and a content-recommendation algorithm. However, a question that jumped out to me was how exactly were they deciding on what videos to watch or to scroll past? The problem they are facing is essentially to decide whether or not a person-resembling bot will like a video or not - but this 'inverse problem' is as difficult as the ‘forward’ content recommendation problem that large scale content systems have tried to solve over the past decades of recommender algorithms. So is it surprising that a bot even with a diverse set of interests will be pigeonholed by a recommender algorithm when the bot can’t express those interests in a way that humans would? From the WSJ video, they mention that their bot will look at images and hashtags associated with videos.

So, I decided to take a shot at making my own content-watching bots. After opening the TikTok website and being hit instantly with a captcha, I turned my attention to YouTube Shorts. They seem to welcome any sort of engagement, bot or otherwise.

## Preference Engine

First step: build a preference engine for our bot. To do this, we’ll have to find what a realistic set of interests look like for a bot. And after that’s done, find out how to express these interests algorithmically. I tried an implementation based on video descriptions and their associated hashtags, but it didn’t do great. Videos only have informative descriptions if their uploaders decide to make them descriptive, but as a user I would never watch a video based on its description. Instead, I found the video thumbnail to be a much better predictor of whether or not I would watch a video.

This poses a challenge, since the preferences of our bot are best *described* in natural language but best *expressed* by observing images. To handle this duality, I’ll co-embed our text-based preferences and the image thumbnails using OpenAI’s [CLIP](https://openai.com/index/clip/). Once we’ve embedded both sides of the modality coin, we can compute distances between video thumbnails and interest categories.

When you actually compute a cosine distance between an image embedding and a text vector you’ll typically get a score around 0.25 for a good match and 0.10 for a poor one. I’d rather not have to fine-tune a threshold to decide whether or not I’m watching a video, so instead I took the 4000 text tags included in Youtube’s [Youtube8M](https://research.google.com/youtube8m/) dataset and embedded these using CLIP. Then I clustered these into 100 categories, which will be our representative interest categories. Below you can see the UMAP representation of the 512-dim text vectors.

![UMAP visualization of the text embedding’s that I’ll use for classification](Reverse-engineering%20the%20YouTube%20algorithm%201fff2a836dec80cab97ce2201e649c88/blog_img1.png)

UMAP visualization of the text embedding’s that I’ll use for classification

![A zoomed in region depicting various video games](Reverse-engineering%20the%20YouTube%20algorithm%201fff2a836dec80cab97ce2201e649c88/blog_img2.png)

A zoomed in region depicting various video games

![A zoomed in region comprised of animals + food](Reverse-engineering%20the%20YouTube%20algorithm%201fff2a836dec80cab97ce2201e649c88/blog_img3.png)

A zoomed in region comprised of animals + food

I’d rather not have to enumerate which of the 4000 tags our bot is interested in, so instead I had ChatGPT summarize each cluster by feeding it a list of the tags. The logic of our agent will be as follows:

1. Grab the video thumbnail and embed it with CLIP
2. Find the nearest text tag and classify the image according to its parent cluster
3. If the parent cluster belongs to the list of preferences we’ve given our agent, watch the video. Otherwise, keep scrolling.

## Experiment #1

For the following experiments, I’ll be working with a few different personas. Each persona will have a list of clusters assigned to it, which themselves have a list of text tags that they want to watch. Here are the five personas that I (and 4o) came up with. Each persona or character has between five and ten clusters they’ll watch, which you can view by clicking on them.

- **🎣 Outdoor Life Enthusiast**
    - 🌾 7: Farming, nature, and outdoor life
    - 🏍️ 23: Motorcycles and off-road vehicles
    - 🏁 26: Motorsports and vehicle customization
    - 🛻 57: Pickup trucks and off-road SUVs
    - 🛠️ 66: Tools, machinery, and equipment
    - 🎣 72: Fishing, boating, and water sports
    - 🛹 58: Skating, boarding, and snow sports
    - 🚙 99: SUVs and off-roaders
- **💄 Beauty and Fashion Lover**
    - 📱 14: Smartphones and mobile devices
    - 👘 27: Japanese culture and fashion
    - 🍣 44: Global culture, cuisine, and fashion
    - 💍 31: Decorations, jewelry, and small objects
    - 👕 82: Clothing, textiles, and accessories
    - 💇‍♀️ 89: Haircare and beauty styles
    - 🍔 91: Everyday and fast food
    - 📺 76: Pop culture and miscellaneous media
- **🎮 Gamer**
    - 🧙‍♂️ 2: Fantasy MMORPG and magic games
    - 🔫 10: Shooters and combat-themed games
    - 🧩 11: Online games and RPG strategy
    - 🏹 21: Assassin's Creed and Prince of Persia
    - 🌟 33: Final Fantasy and Kingdom Hearts
    - 🏎️ 50: Racing and driving simulation games
    - 🦔 51: Sonic the Hedgehog games
    - 🎖️ 63: Modern shooters and war games
    - 🌍 79: Simulation and sandbox games
- **🎸 Music Head**
    - 🎸 12: Musical instruments and furniture
    - 🎸 16: Electric and acoustic guitars
    - 🎺 25: Brass and woodwind instruments
    - 🥁 47: Rock music and percussion instruments
    - 🎸 59: Guitar Hero music games
    - 🎶 67: Music rhythm and dance games
- **🐾 Animal Devotee**
    - 🐶 0: Dogs, horses, and working animals
    - 🌾 7: Farming, nature, and outdoor life
    - 🦓 64: Animals, pets, and wild species
    - 🐕 84: Guard dog breeds
    - 🦎 85: Pet geckos and reptiles
    - 🐕‍🦺 96: Shepherd and companion dog breeds
    - 🐾 98: Domesticated and wild animals

Using Playwright, a web automation package for Python, we can spin up an instance of Chromium and open YouTube Shorts. Our bot starts scrolling. Once it gets to a video, we grab the thumbnail and run it through our “should we watch this video”-logic. If we decide not to watch the video, the bot will stay on the video for a random time (about 2 seconds) so it doesn’t look too much like a bot. Below I included the thumbnail history for our music head bot, with a thumbnail highlighted green if we watched the video and red otherwise.

- Click me for an example scroll!
    
    ![alt_scroll.png](Reverse-engineering%20the%20YouTube%20algorithm%201fff2a836dec80cab97ce2201e649c88/alt_scroll.png)
    

It’s pretty obvious that YouTube’s algorithm quickly starts to serve up music-related content. Are we being pigeonholed? Well, I initialized the bots five times and measured the fraction of videos being served that it watches (a measure of how well Youtube’s algorithm is discovering our preferences) along with the average number of views that a video has. I don’t know how to quantify ‘pigeonholeness’, but both plots tell the same story: we quickly get more niche and specific content to match what our bot will decide to watch.

![(Left) The agent receives more niche content. (Right) The agent gets more relevant content.](Reverse-engineering%20the%20YouTube%20algorithm%201fff2a836dec80cab97ce2201e649c88/blog_img6.png)

(Left) The agent receives more niche content. (Right) The agent gets more relevant content.

To better visualize what’s going on, I wanted to show the videos in the embedding space that the text tags already live in. In the interest of reducing clutter, I transformed all of the points belonging to clusters that we’re interested in into a kernel density plot.

![(Left) Original text embeddings with the tags corresponding to our cluster highlighted in blue. (Right) Same plot but with the highlighted points converted into a kernel density plot.](Reverse-engineering%20the%20YouTube%20algorithm%201fff2a836dec80cab97ce2201e649c88/kernel_plot_transformation.png)

(Left) Original text embeddings with the tags corresponding to our cluster highlighted in blue. (Right) Same plot but with the highlighted points converted into a kernel density plot.

Now as we watch videos, I plot the video embedding (rather, its UMAP reduced embedding) as a red square if we don’t watch it, and a green square if we do watch it.

![A scrolling sessions for our music guy. Red squares are videos we didn’t watch. Green squares are video we did watch.](Reverse-engineering%20the%20YouTube%20algorithm%201fff2a836dec80cab97ce2201e649c88/agent_path_colored5.gif)

A scrolling sessions for our music guy. Red squares are videos we didn’t watch. Green squares are video we did watch.

This ends up being a pretty nice way to visualize our browsing history! Below I have a different character - the outdoor enthusiast — and two different runs: on the left we have a run where Youtube never figures us out, and on the right we are ‘algorithmically captured’.

![Bot scrolls for hours with no YouTube response.](Reverse-engineering%20the%20YouTube%20algorithm%201fff2a836dec80cab97ce2201e649c88/agent_path_colored2-ezgif.com-cut.gif)

Bot scrolls for hours with no YouTube response.

![Bot scrolls for a reasonable time and is soon rewarded with engaging content.](Reverse-engineering%20the%20YouTube%20algorithm%201fff2a836dec80cab97ce2201e649c88/agent_path_colored3-ezgif.com-cut_(1).gif)

Bot scrolls for a reasonable time and is soon rewarded with engaging content.

# Experiment #2

My next idea was to scale up the YouTube Shorts bot so that I could run a bunch of them simultaneously, getting more data in the process. Because the CLIP model I’m using is actually pretty small, I ended up creating a Google Cloud project and running a headless version of the bot on a tiny e2 VM. I wrote a script to take in a character and launch a VM, scroll on YouTube for a few hours, write its logs to cloud storage, and delete itself. A sad life indeed for our little researchers.

I envisioned myself launching 1000 of these and seeing what they found after scrolling. But, in the interest of not exploding my cloud bill and invoking the rage of YouTube, I decided to run a more limited experiment of 5 runs for each of my 5 characters, the bots running for 2 hours a piece. Even at this limited number, you run up pretty quickly against google cloud vCPU and IP address quotas. I managed to get my project upgraded to a higher vCPU usage and spread out the bots over different zones to get around the IP address issue. This is also where my roommate accused me of having too much free time. At least I’m not the one scrolling through YouTube, its the bots.

My final experiment consists of five runs for each of the five characters I defined. Results below.

![Our experimental runs for each of the five characters.](Reverse-engineering%20the%20YouTube%20algorithm%201fff2a836dec80cab97ce2201e649c88/blog_img10.png)

Our experimental runs for each of the five characters.

A few things that I noticed:

1. **Some categories are much more easily found than others.** Music head struggles a lot more than our fashionista. Makes sense.
2. **Randomly my bots will get stuck on generic content.** I’m not sure why this is happening. The animal lover, for example, finds animals to love exceedingly fast (unsurprisingly) but sometimes won’t at all. This is probably an error on my end.
3. **Even after two hours the algorithm is still learning.** This is also expected, but after an initial ‘capturing phase’ where the amount of relevant content we get served increases rapidly, we enter another slow ‘learning phase’ where our relevant content still increases, just at a decreased rate.

# Experiment #3

One interesting thing I wanted to study with these bots is algorithm inertia. There’s an interesting interplay between the preferences a content recommendation algorithm tries to extract from the user and the preferences that it ends up enforcing. What if I change my preferences quickly — will the content recommender be sensitive to that?

Here’s my experiment: I initialized the bots all as our outdoors guy and had them watch 300 videos. After the 300th video, they all switch their preferences to one of the other four characters. The results below show again the fraction of video’s that the bots are watching.

![blog_img11.png](Reverse-engineering%20the%20YouTube%20algorithm%201fff2a836dec80cab97ce2201e649c88/blog_img11.png)

After the vertical dashed line, the bots switch their characters and so stop watching all of the outdoors content that Youtube is serving them. Soon enough, Youtube realizes that something is wrong and they’re back to watching content that their new preferences align with.

If we define a ‘captured’ bot as a bot that watches more than 1/5 of the content they’re served (which conveniently aligns with what all of my data did), we can compare the V2C (number of videos until capture) of the bots starting fresh, or starting from the outdoor persona. The table below shows these numbers.

|  | Music Head  | Gamer | Beauty and Fashion | Animal Devotee |
| --- | --- | --- | --- | --- |
| Fresh | 265 | **290** | 61 | 98 |
| Non-fresh | **301** | 242 | **134** | **151** |

In most cases we had to wait longer for the non fresh-start bots for Youtube to find their preferences. But overall I was impressed at how quickly we could switch around and get entirely new content served at us.

# Take-away thoughts

Ultimately this experiment was limited more by lack of questions than difficulty acquiring data. I still don’t know what a good question you might have about a content recommendation system that could be answered with bots like this. 

After seeing how strangely the youtube algorithm will behave if you have a really poor bot on your end, I don’t know if I trust any conclusions WSJ came to after doing their Tik Tok analysis. In any case, they would need more trials and transparency of their methods to have any meaningful answers.

Finally I think it would be cool to see how far you could push the preference engine of one of these scrollers. I don’t know exactly what benchmark you’d be testing against — maybe some sort of consistency with the content recommendation algorithm (reminds me of model distillation). If the latency was low enough, you could feed key frames into a multi-modal LLM to decide what to watch or to scroll past.