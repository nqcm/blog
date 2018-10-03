---
title: "⚡️ Building Twitter Bot"
date: 2018-10-02T17:05:33+02:00
---

I built a Twitter bot that tweets [Tajweed](https://en.wikipedia.org/wiki/Tajwid) rules every eight hours using Python and AWS Lambda.

<br>

<div class="center">
    <blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">The best of you is he who learns the Quran and teaches it. ~ Prophet Muhammad SAW<br> Today&#39;s <a href="https://twitter.com/hashtag/tajweed?src=hash&amp;ref_src=twsrc%5Etfw">#tajweed</a> rule... <a href="https://t.co/SMhyE2Jgi9">pic.twitter.com/SMhyE2Jgi9</a></p>&mdash; TajweedBot (@tajweedbot) <a href="https://twitter.com/tajweedbot/status/1047116062532096001?ref_src=twsrc%5Etfw">October 2, 2018</a></blockquote>
    <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

<br>

I have [previously written](https://blog.naveeraashraf.com/posts/making-tajweed-bot/) about the writing and designing part of the project and in this part outlines the technical details on how the bot was made and works.

<br>

<a href="https://twitter.com/tajweedbot?ref_src=twsrc%5Etfw" class="twitter-follow-button" data-size="large" data-show-count="false">Follow @tajweedbot</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

You can check out the completed code [here](https://github.com/nqcm/tajweedbot)

## Using S3

I decided on using an Amazon S3 bucket to store the image cards as it has a nice free tier. Since I was planning on hosting the bot on AWS Lambda, the integration between different AWS services is seamless.

S3 has a nice web based uploader to upload stuff but if you have huge amount of files to transfer, using their CLI would be the way to go. For my purpose however, I just used the website's uploader.

![S3 Uploader](/img/aws-s3-upload.jpg)


## Testing Boto3

My first step was to test the usage of Amazon's SDK for Python which is called [Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html). Strange name, I know. But it is [named after a dolphin](https://github.com/boto/boto3/issues/1023) 'Boto' which navigates the Amazon rainforest's eco system. Cool! 

So after installing Boto3 in my virtual environment, I wrote a simple script to test if I could download a file using Boto3. Turned out I could.

```Python
    import boto3
    import botocore

    BUCKET_NAME = 'tajweedbot'
    KEY = 'round-zero.jpg'

    s3 = boto3.resource('s3')

    try:
        s3.Bucket(BUCKET_NAME).download_file(KEY, 'local.jpg')
    except botocore.exceptions.ClientError as e:
        if e.response['Error']['Code'] == "404":
            print("The object does not exist.")
        else:
            raise   
```

## Choosing A Random Image

My next step was to grab a random image from the bucket with every execution. Using the `bucket.objects.all()` method I could count all the objects in that bucket and feeding and feeding it to a random int function, I was able to get a random number between 1 and the total number of files in that bucket. Adding it with the '.png' extension, I assigned it to the KEY variable replacing the hard-coded KEY.

```Python
    obj = str(random.randint(1, sum(1 for _ in bucket.objects.all())))
    KEY = obj + '.png'
```

But this is not an elegant solution. For one, my file names had to be numbers like '1.png' etc. Secondly there must be a dedicated bucket for the images being used by this bot. 

An ideal solution would be to make a JSON file containing the names of the images and then I could get a random file name from the JSON document with a simple script like:

```Python
    s3 = boto3.resource('s3')
    bucket= s3.Bucket(BUCKET_NAME)
    obj = s3.Object(BUCKET_NAME, "meta.json")
    data = obj.get()
    js = json.loads(data['Body'].read().decode('utf-8'))
    pic = str(random.randint(1, len(js))) + '.png'
```
So in my next iteration over the project, I may implement this method. But for now I am sticking with my not-so-elegant solution.

## Tweeting the Image

Next step was to make my bot tweet this downloaded image. That was easy using [Twython](https://twython.readthedocs.io/en/latest/). First I registered an application at <https://apps.twitter.com>, grabed my applications Consumer Key and Consumer Secret. Sending a tweet can be easily done through the following lines of code.

```Python
    twit_resp = twitter.upload_media(media=img)
    twitter.update_status(status="Some status", media_ids=twit_resp['media_id'])
    print("image tweeted")
```





<br>
<br>

