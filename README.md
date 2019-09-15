# Install and Use CTRL on Google Compute Engine

A script + guide on how to set up a machine on Google Compute Engine capable of running and use CTRL to generate conditional text.

## Machine Setup Instructions

This machine is the minimum configuration powerful enough to run CTRL without going out-of-memory (P100 GPU, 8 vCPU, 30 GB RAM, preemtible). With this configuration, having the VM up will cost **$0.51/hr**.

1. Make sure `gcloud` is set up on your local computer and up-to-date (can update via `gcloud components update`).
2. Make sure your Google Compute Engine project tied to your local computer's `gcloud` has enough quota in the `us-central-1` region (8 CPUs and 1 P100; these should be available by default, but request more quota if they aren't)
3. On your local computer, run this `gcloud` command which creates a VM with the specs noted above:

```sh
gcloud compute instances create ctrl \
  --zone us-central1-c \
  --boot-disk-size 45GB \
  --image-family tf-latest-gpu \
  --image-project deeplearning-platform-release \
  --maintenance-policy TERMINATE \
  --machine-type n1-standard-8 \
  --metadata install-nvidia-driver=True \
  --preemptible \
  --accelerator='type=nvidia-tesla-p100,count=1'
  
```

You can view the created instance, turn it on/off, and delete it, in the [Google Compute Engine](https://console.cloud.google.com/compute/instances) dashboard.

Once created (after waiting a bit for the GPU drivers to install), SSH into the instance. (recommended way to do is via the gcloud command created from the SSH dropdown, which will look like:

```sh
gcloud beta compute --project "<PROJECT ID>" ssh --zone "us-central1-c" "ctrl"
```

Download and run the `install_gce.sh` script from this repo via:

```sh
curl -O -s https://raw.githubusercontent.com/minimaxir/ctrl-gce/master/install_gce.sh
sudo sh install_gce.sh
```

You can see what the script does by looking at it in this repo.

You're done!

## Using CTRL

For basic usage, running the command below will load the model and eventually start an interactive prompt.

```sh
sudo python generation.py --model_dir seqlen256_v1.ckpt/
```

While generating, you can specify a KeyboardInterrupt to stop generation (Ctrl+C on macOS). It's also recommended to clear the terminal (CMD+K on macOS) occasionally.

You **must** include a control code with each interactive prompt. You can see how the control codes are used in the original paper, or refer to the following examples:

### Links

The `Links` control code allows you to specify a URL and generate text from it. Examples:

`Links https://www.buzzfeednews.com/article/annehelenpetersen/jeremy-renner-app-trolling-career-hurt-locker-avengers`

```
The actor who played the villain Loki in Avengers: Infinity War has been accused of being a troll. 
 
Actor Jeremy Renner was criticized for his role as Loki, an antagonist from Marvel's upcoming Avengers movie, after he tweeted that fans should "stop trying to make me look bad." 
 
In response to Renner's tweet, many people on Twitter said they were offended by what they perceived as trolling.
```

`Links https://www.nytimes.com/2020/08/29/us/politics/skynet-ai-online-norad.html`

```
Site Mobile Navigation 
 
A New Online Course for the Super-Smart 
 
The University of Washington is offering a new online course on artificial intelligence that will be taught by an expert in machine learning and neural networks. 
 
The course, called “Skynet,” was developed with help from Google, Facebook, Microsoft and other companies. It has been designed to teach students how to use deep learning, which uses computer programs modeled after neurons to analyze large amounts of data — like images or videos — using only basic rules about what each pixel represents. 
 
In addition to teaching them how to build their own programs, it also teaches them how to apply those algorithms to real-world problems. 
```

### Questions

The `Questions` control code allows you to specify a question; CTRL will attempt to follow up from that question (your mileage will vary). Examples:

`Questions Q: Who is the Zodiac Killer?`

```
A: charles manson 
```

`Questions Q: What is the last digit of pi?`

```
A: 0
```

### Reddit Subreddits

Each Reddit subreddit has a different control code, see the bottom of the paper for a list. You can optionally specify the Title or the self-post Text body. Examples:

`Legal Title: [FL] Is it legal to get into legal trouble?`

```
Text: I was driving home from work and saw a police car pull up behind me. The officer asked me for my license and registration. He then proceeded to ask if i had any drugs in the car. 
 
My question is this: 
 
1. Can you be arrested without probable cause or even being charged with anything at all? 
 
2. If so, what are your chances of getting out of this situation? 
 
3. What happens when they arrest you? 
 
4. How do you go about proving that there were no drugs? 
 
5. Do you have an attorney who can help me? 
 
Thanks 
 
Edit: 
 
Thank you all so much for your responses. This has been very helpful as I'm trying to figure out how best to proceed. 
```

`Opinion Text: Recycling is bad for the world.`

```
It takes a lot of energy to recycle and it creates more waste than it saves. 
 
I think that recycling should be banned because we are destroying our planet with it. 
 
Edit: Thanks everyone who has given me some good arguments against recycling. 
 
Edit 2: I'm not saying that people shouldn't recycle; it's just that they need to do so in an environmentally friendly way.
```

### Amazon Reviews

The `Reviews` tag can be used to generate Amazon reviews at an optional specified rating or starting text.

`Reviews Rating: 1.0`

```
I bought this book because it was recommended by a friend. It is not worth the money. The author has no credentials and his writing style is very poor.
```

`Reviews Rating: 5.0\n\nI died`

```
a little inside when I saw the first page of this book. It was so beautiful and it made me feel like I could do anything. But then I read on to see what happened next. And there were no more pages. The book just stopped. No epilogue, nothing. Just an abrupt ending. I'm not sure if it's because there's another one coming out or what, but that's how I feel. It's almost as though she got tired of writing about her life in New York City and decided that she'd write something else instead.
```

## Notes

* Since the model is huge, generation is *very* slow: about 2 tokens per second with the configuration above. (therefore, it takes about 2 minutes for a full generation with default parameters)
* The BPEs CTRL uses are "longer" that those used in GPT-2. As a result, a 256-token generation in CTRL is about the same decoded length as a 1024-token generation in GPT-2.
* When using the `Links` control code, keep in mind that code is conditioned on OpenWebText, which is conditioned on Reddit data. Therefore, there's a bias toward English websites and Reddit-friendly content. [Here's a quick spreadsheet](https://docs.google.com/spreadsheets/d/1O3_1RTnGn_ozcvf6eDLpsmf8ElxIqHg0FKbVfDxGS88/edit?usp=sharing) of the most popular TLDs on Reddit, sans some obvious image-oriented websites.
* If CTRL gets confused by the `Links` URL, it tends to fall back to a more general news-oriented output.
* It is recommended to use Google Compute Engine (even if you aren't following this guide) as the model itself is hosted in Google Cloud Storage and thus it's relatively fast to transfer to a VM (>100 Mb/s), and also lowers the cost for Salesforce.

## TODO

* Support/Test the 512-length CTRL model.
* Support/Test domain detection.

## Maintainer/Creator

Max Woolf ([@minimaxir](https://minimaxir.com))

*Max's open-source projects are supported by his [Patreon](https://www.patreon.com/minimaxir). If you found this project helpful, any monetary contributions to the Patreon are appreciated and will be put to good creative use.*

## Special Thanks

[Adam King](https://twitter.com/AdamDanielKing) for identifying a working implementation of loading the model after unexplained setbacks.

## License

MIT

## Disclaimer

This repo has no affiliation or relationship with the CTRL team and/or Salesforce.