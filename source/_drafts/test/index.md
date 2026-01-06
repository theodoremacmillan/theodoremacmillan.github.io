---
title: Reproducing WSJ Tik Tok bots (title in progress)
author: Theo MacMillan
date: 2025-04-26 18:00:00
---

# Background: 

Something about history viewing youtube shorts and instagram. On the first day of CHristmas my true love gave to me a partridge in a pear tree.

Inspired by Wall Street Journal Tik Tok analysis. Here's the gist: they initiated ~100 bots each with their own hidden interests. One was interested by forestry, another astrology. Then they send their bots into Tik Tok, having them scroll past videos they were disinterested by and pause and watch videos that aligned with their interests. They found that across the board their bots were pushed towards more niche content, even bots initialized with a diverse set of interests.

After seeing their analysis, I was equal parts intrigued and disapointed. It seems like a fantastic approach towards auditing the behavior of a live recommendation system, but their analysis lacked some amount of scientific rigor.

The question that jumped out to me was how exactly were they deciding on what videos to watch or to scroll past? The problem they are facing is essentially to decide whether or not a person-resembling bot will like a video or not -- but this 'inverse problem' is as difficult as the content recommendation problem that large scale content systems have tried to solve over the past decades of recommender algorithms. So is it surprising that a bot even with a diverse set of interests will be pigeonholed by a recommender algorithm when it can't express those interests in a way that humans would? From the WSJ video, they mention that their bot will look at images and hashtags associated with videos.

So, I decided to take a shot at making my own content-watching bots. After opening the Tik Tok website and being hit instantly with a capcha, I turned my attention to youtube shorts. They seem to welcome any short of engagement, bot or otherwise.

First, what does a realistic set of interests look like for a bot? How can we express these interests algorithmically? I tried an implementation based on video descriptions and their associated hashtags, but this approach seemed doomed from the start. Videos only have informative descriptions if their uploaders decide to make them descriptive, but as a user I would never watch a video based on its description (example of video that someone might watch with an uninformative description). Instead, I found the video thumbnail to be a much better predictor of whether or not I would watch a video, but there are still a lot of counterexamples.

This makes the classification problem harder, but very tractable with today's ML toolkit. I used CLIP, which is an OpenAI model that can place both images and text in the same embedding space. From there, we can compute distances between video thumbnails and interest categories.

<pre> ```python def hello(): print("Hello, world!") ``` </pre>


So what should the interest categories be? For this, I grabbed video tags from Youtube8M and embedded them using CLIP. In that high dimensional vector space (512 dimensions) I ran k-means with 100 clusters to get groups of tags, and UMAP to visualize these embeddings/clusters in a 2D space.

Image:

{% asset_img test.png Alt Text %}


\begin{equation}
\mathcal{O}(n)=
\end{equation}

<details><summary>Does not work</summary>
[hi](https://hello.ca)
</details>


<details><summary>Does work</summary>

[hi 📖](https://hello.ca)

</details>


# Our personas

<details>
<summary>🎣 Outdoor Life Enthusiast</summary>

- 🌾 7: Farming, nature, and outdoor life
- 🏍️ 23: Motorcycles and off-road vehicles
- 🏁 26: Motorsports and vehicle customization
- 🛻 57: Pickup trucks and off-road SUVs
- 🛠️ 66: Tools, machinery, and equipment
- 🎣 72: Fishing, boating, and water sports
- 🛹 58: Skating, boarding, and snow sports
- 🚙 99: SUVs and off-roaders

</details>

<details>
<summary>💄 Beauty and Fashion Lover</summary>

- 📱 14: Smartphones and mobile devices
- 👘 27: Japanese culture and fashion
- 🍣 44: Global culture, cuisine, and fashion
- 💍 31: Decorations, jewelry, and small objects
- 👕 82: Clothing, textiles, and accessories
- 💇‍♀️ 89: Haircare and beauty styles
- 🍔 91: Everyday and fast food
- 📺 76: Pop culture and miscellaneous media

</details>

<details>
<summary>🎮 Gamer</summary>

- 🧙‍♂️ 2: Fantasy MMORPG and magic games
- 🔫 10: Shooters and combat-themed games
- 🧩 11: Online games and RPG strategy
- 🏹 21: Assassin's Creed and Prince of Persia
- 🌟 33: Final Fantasy and Kingdom Hearts
- 🏎️ 50: Racing and driving simulation games
- 🦔 51: Sonic the Hedgehog games
- 🎖️ 63: Modern shooters and war games
- 🌍 79: Simulation and sandbox games

</details>

<details>
<summary>🎸 Music Head</summary>

- 🎸 12: Musical instruments and furniture
- 🎸 16: Electric and acoustic guitars
- 🎺 25: Brass and woodwind instruments
- 🥁 47: Rock music and percussion instruments
- 🎸 59: Guitar Hero music games
- 🎶 67: Music rhythm and dance games

</details>

<details>
<summary>🐾 Animal Devotee</summary>

- 🐶 0: Dogs, horses, and working animals
- 🌾 7: Farming, nature, and outdoor life
- 🦓 64: Animals, pets, and wild species
- 🐕 84: Guard dog breeds
- 🦎 85: Pet geckos and reptiles
- 🐕‍🦺 96: Shepherd and companion dog breeds
- 🐾 98: Domesticated and wild animals

</details>

On the first day of Christmas my true love gave to me a partridge in a pear tree. On the second day of Christmas my true love gave to me two turtle doves and a partridge in a pear tree. On the third day of Christmas my true love gave to me three french hens, two turtle doves, and a partridge in a pear tree. Let's watch our outdoor enthusiast take a scroll through Youtube Shorts land...

<div style="text-align: center;">

  # Outdoor enthusiast reels history
  
<div style="height:600px; width:400px; overflow-y:auto; border:1px solid #ccc; margin: 0 auto;">
  <img src="/images/testtest.svg" alt="Long Image">
</div>


</div>



Early on, we're getting generic youtube shorts content. But as our bot exhibits its preferences, youtube shorts starts getting the hint and serving it more outdoorsy content. Again, not every video is properly classified (although what "properly" here means is not entirely clear), but the CLIP recommender does a pretty good job altogether.

<div style="display: flex; justify-content: center; gap: 20px;">

  <img src="/images/agent_path_colored2-ezgif.com-cut.gif" alt="GIF 1" style="width:45%; height:auto;">
  <img src="/images/agent_path_colored3-ezgif.com-cut (1).gif" alt="GIF 2" style="width:45%; height:auto;">

</div>

Here we see what two different paths through embedding space look like. On the left is an example of a bot that wasn't captured by the algorithm after 2 hours of scrolling. As a counterpoint, the agent on the right with the same preference vector is captured. Should show how these kernel density plots are generated...

![Test Image](/images/blog_img6.png)

Now we're going to repeat the same analysis for each of our 5 personas. CHeck it out below. More random text: The question that jumped out to me was how exactly were they deciding on what videos to watch or to scroll past? The problem they are facing is essentially to decide whether or not a person-resembling bot will like a video or not -- but this 'inverse problem' is as difficult as the content recommendation problem that large scale content systems have tried to solve over the past decades of recommender algorithms. So is it surprising that a bot even with a diverse set of interests will be pigeonholed by a recommender algorithm when it can't express those interests in a way that humans would? From the WSJ video, they mention that their bot will look at images and hashtags associated with videos.


![Test Image](/images/blog_img7.png)

![Test Image](/images/blog_img8.png)

![Test Image](/images/blog_img9.png)