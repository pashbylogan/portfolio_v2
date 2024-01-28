---
title: "Fictionary"
date: 2023-04-29T13:00:46-07:00
author: "Me"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://cdn.hashnode.com/res/hashnode/image/upload/v1682860188524/b1370476-d015-4cb4-90c9-c2a40a0b4fb7.png" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
---

## Introduction

In this blog post, I'll discuss our award-winning hackathon project from the 2021 Western Washington University Hackathon. Our team developed a web application called Fictionary that generates fictitious definitions for coherent words. We'll guide you through the process of building this app, which consists of a large language model, a website, and an API that passes information between the two. We'll also share code examples, images, and insights into our project's development.

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1682860188524/b1370476-d015-4cb4-90c9-c2a40a0b4fb7.png" alt="fictionary screenshot" >}}

## **Language Model: Fine-Tuning GPT-2**

To create Fictionary, we fine-tuned the GPT-2 language model (which we got from [hugging face](https://huggingface.co/) 🤗) on dictionary data to train it to generate new definitions for given words. We compared the results between fine-tuning on Urban Dictionary and the Online Plain Text English Dictionary. Ultimately, we decided to go with Urban Dictionary as the dataset was significantly larger and provided more entertaining definitions.

### **Data Cleaning and Profanity Check**

To get the data into a usable format, we downloaded all of Urban Dictionary to a CSV and used pandas to remove duplicates, extremely vulgar definitions, etc. We applied a profanity check in two places to the data to make the definitions generated from Urban Dictionary less vulgar. The aforementioned data-cleaning phase as well as in the API itself to check the actual generated text. This pared the dataset down to around 2 million word-definition pairs. Here's a snippet of our profanity check code:

```python
try:
    definition = models.define(model, word, num_return=1)[0]
    definition = definition
                    .replace(']', '')
                    .replace('[', '')\
                    .replace('f***', 'duck')
                    .replace('c***', 'trunk')\
                    .replace('sex', 'love')
    if definition[-1] != '.':
        definition += '.'
except Exception as e:
    print(e)
    definition = "This word is undefinable. Good job..."
```

*Please note that the model still may generate explicit material occasionally, so use it at your own discretion.*

## **Building the Website**

We built the website using Node.js and Express for the webserver backend, Handlebars for HTML templating, and Sass for styling. Below is a simplified example of our server setup:

```javascript
javascriptCopy codeconst express = require('express');
const app = express();
var indexRouter = require('./routes/index');

app.set('view engine', 'hbs');
app.use(express.static('public'));

app.use('/', indexRouter);

app.listen(3000, () => {
  console.log('Fictionary app listening on port 3000!');
});
```

### **Twitter and Zazzle API Integration**

We also integrated the Twitter API for users to tweet about their Fictionary words and the Zazzle API to create a custom mug featuring their words and definition.

{{< figure src="https://rlv.zcache.com/fictionary_mug-r1fee51c5c3254ce9b1b2344860f0c7e8_kfpv9_1024.jpg?rlvnet=1&t_text_txt=hello%20from%20my%20blog%0AA%20way%20of%20saying%20hello%20in%20a%20non-sensical%20manner.%20Often%20used%20by%20people%20who%20have%20never%20heard%20the%20term%2C%20and%20are%20too%20lazy%20to%20type%20%27Hello%27%20or%20say%20it%20correctly%20because%20they%27re%20bored%20with%20typing%20them%20out%20on%20their%20cell%20phone%20(or%20any%20other%20device" >}}

## **Creating the API with Flask**

Lastly, we developed an API using the Flask Python framework to communicate between the language model and the website. Here's a basic example of our Flask API setup:

```python
from flask import Flask, request, jsonify
import sys

sys.path.append('..')
import models.train as models

app = Flask(__name__)

# ML model definition
loaded_models = {
    '1': 'data/dict-short.bin',
    '2': 'data/dict-long.bin',
    '3': 'data/urbandict.bin',
    '4': 'data/urbandict-long.bin',
}
model = input(f"{loaded_models}\nEnter model number from above: ")
weights = loaded_models[model]
model = models.get_model_for_api(weights_path=weights)

@app.route('/word', methods=["POST"])
def word():
    data = request.json
    word = data['word']
    definition = models.define(model, word, num_return=1)[0]

    resp_data = json.dumps(
            {'definition': definition}
        )
    response = Response(resp_data)
    return response

@app.route('/', methods=["GET"])
def index():
    return redirect(f'http://hackathon.chrisdaw.net:3030', 301)

if __name__ == '__main__':
    """
    # Load ML model
    for k, weights_path in loaded_models.items():
        loaded_models[k] = models.get_model(weights_path)
    """
    app.debug=True
    app.run(host='0.0.0.0')  # run our Flask app
```

## **Fictionary Now Available in React**

Since its release, I have [forked the repo](https://github.com/pashbylogan/Fictionary) and made various improvements including:

1. Ported the app over to React
    
2. Set up permanent hosting on my website for everyone to enjoy
    
3. Added additional security to the Flask API like authentication
    

### **Call to Action: Try Fictionary Now!**

We invite you to visit [**loganpashby.com/fictionary**](http://loganpashby.com/fictionary) to try Fictionary for yourself. Generate your own unique and entertaining definitions for any words you can imagine and share them with your friends. You can also create a custom mug featuring your word and definition, or tweet your creations to your followers. Have fun, and remember to use Fictionary responsibly!

Thank you for reading our blog post about our award-winning hackathon project. We hope you found it insightful and enjoyable. If you have any questions or suggestions, please feel free to leave a comment below. Happy defining!
