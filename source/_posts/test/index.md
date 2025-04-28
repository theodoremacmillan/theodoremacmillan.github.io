---
title: Reproducing WSJ Tik Tok bots
date: 2025-04-26 18:00:00
---


## Content watching bots

Something about history viewing youtube shorts and instagram.

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

![Test Image](/images/test.png)

![An Imgur Image](https://i.imgur.com/ucHPX7L.gif "An Imgur GIF")


\begin{equation}
\mathcal{O}(n)=
\end{equation}

```markdown
<details>
<summary>Click to see Python code</summary>

```python
def hello():
    print("hi")