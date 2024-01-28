---
title: "Autocut"
date: 2023-04-30T13:00:46-07:00
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
As remote learning continues to be the norm, teachers and professors are relying more heavily on recorded lectures to deliver course material to their students. However, recorded lectures can be time-consuming to edit, and cleaning up audio files can be a particular challenge.

That's where Autocut comes in. Autocut is an open-source tool designed to automatically clean up recorded lectures, saving teachers and professors valuable time and effort. It's designed to allow a user to upload their lecture video into the tool and Autocut will proceed to edit out silence, dead air, and noise artifacts from audio files and their accompanying video (if applicable).

## **How Autocut Works**

Autocut uses pre-trained deep learning models to perform various labeling tasks such as Silence Removal, Automatic Speech Recognition, Forced alignment, and Voice Activity Detection. These models analyze the recorded lecture, identifying and removing unwanted sounds, such as background noise, coughs, and sneezes. The result is a cleaner, more professional-sounding recording that is easier for students to follow.

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1682868048308/cd013aa9-0244-40ae-81a0-b87bb665db4a.png" >}}

## **Project Components**

Autocut consists of four main components:

### **1\. Label Studio** Front-End

The [Label Studio](https://labelstud.io/) (LS) front-end is mostly unchanged, but we had to connect it to our custom LS backend API and set up the XML interface to visualize everything the user might be interested in.

{{< figure src="https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fdocs.primehub.io%2Fdocs%2Fassets%2Fapp_tutorial_labelstudio_login_page.png&f=1&nofb=1&ipt=066c2fc7d67847d053295854b910e0cd25c240a6d5db3cd53e30833f17717e1f&ipo=images" >}}

### **2\. Label Studio Backend**

The Label Studio Backend implements a required LS class so that the LS front-end knows how to retrieve predictions. The main logic goes in the "predict" method. In the latest iteration of the product, there are several pre-trained DL models being used for various labeling tasks such as Silence Removal, Automatic Speech Recognition, Forced alignment, and Voice Activity Detection.

### **3\. Flask API**

The Flask API responds to the submit button from the LS front-end and calls the appropriate video editing scripts. Additionally, this API accepts the uploaded file from the landing page UI and stores it so that it can make edits to the file later on (for which we use [ffmpeg](https://ffmpeg.org/)). The Flask API also sends emails to users, including verification codes and project links upon inference completion.

### **4\. Landing Page**

The landing page is responsible for accepting uploaded files from users and presenting them with a user-friendly interface for labeling the file. For this, we used a bare-bones React app to enable user management and file upload before proceeding to route them to LS.

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1682867114694/d0c040ad-0c06-42c7-aea3-eec19749d924.png" >}}

## **Cluster Compute**

All inference in Autocut is done on the WWU cluster computer. The machine learning backend API will scp the current video file. The ML backend calls the inference script over SSH and then remains idle, leaving the LS front-end waiting until the ML finishes. The outputs labels are a synthesis of the various model outputs that we can translate into information about which areas of the video/audio file to delete, speed up, mute, etc. Finally, the cluster "cleans up" by deleting the video and label files because all data management should happen on the home server side.

## Installation

**Instructions for installing label-studio as a dev:**

1. Make sure you are inside of the ml\_autocut21 repo
    
2. Clone the fork of label-studio
    

`git clone` [`https://github.com/TrevORtega/label-studio.git`](https://github.com/TrevORtega/label-studio.git)

`cd label-studio`

^This will create a git submodule inside of the repository. Use git status to check which repo you are in if it is confusing.

1. Install required dependencies and run ( if there are no front-end changes )
    

```bash
# Install all package dependencies
pip install -e .
# Run database migrations
python label_studio/manage.py migrate
# Start the server in development mode at http://localhost:8080 
python label_studio/manage.py runserver
```

1. This is how we apply and view front-end changes if we make them
    

```bash
cd label_studio/frontend/
npm ci
npx webpack
cd ../..
python label_studio/manage.py collectstatic --no-input
```

To use our specific models in Label Studio:

1. (If first time) Follow the directions in the Label Studio ML backend repository to install as dev: [https://github.com/heartexlabs/label-studio-ml-backend](https://github.com/heartexlabs/label-studio-ml-backend)
    
2. run:
    
    ```bash
    label-studio-ml init dafdl_backend --script src/LS-audio-segment-model.py
    label-studio-ml start dafdl_backend
    ```
    
3. Create a new project in label studio (should still be running on [http://localhost:8080](http://localhost:8080))
    
4. In project settings add a new ML Backend with [http://localhost:9090](http://localhost:9090) as url.
    
    * Turn on automatic annotations
        
    * Use custom labeling config:
        
        ```xml
            <View>
                <Labels name="labels" toName="audio">
                  <Label value="Speaker 1" />
                  <Label value="Speaker 2" />
                </Labels>
                <AudioPlus name="audio" value="$audio"/>
        
                <View visibleWhen="region-selected">
                  <Header value="Provide Transcription" />
                </View>
        
                <TextArea name="transcription" toName="audio"
                          rows="2" editable="true"
                          perRegion="true" required="true" />
                <TextArea name="transcription2" toName="audio"
                          rows="2" editable="true"
                          required="true" />
          </View>
        ```
        
5. Import data and view the automatic predictions!
    

## FAQs

### **Why Label Studio?**

We chose Label Studio because it had a built-out front-end and labeling capabilities already set. We considered other styles of software to build off of such as open-source video editing, but ultimately decided against this because these projects are massive and too complex for what we were trying to accomplish. Competing tools at this task were either lacking in ML support or were too rudimentary to be built into a software product at the scale we were interested in.

### **Why not passwords?**

We chose not to implement a password system as we didn't want to be responsible for user password storage. Instead, we felt that the code verification system was sufficient.

### **Why Autocut?**

Autocut is an open-source tool that can help teachers and professors save time and effort in editing their recorded lectures. By using pre-trained deep learning models to clean up recorded lectures, Autocut makes it easier for teachers and professors to provide high-quality course material to their students. Whether you're a teacher or professor looking to improve your recorded lectures, Autocut is an excellent tool to help you achieve your goals.

### Can I see the code?

Yes! However, since it was a school project we had to make the repo private. Get in touch with me and I can get you access.

## **Future Work and Ideas**

While Autocut is a powerful tool for automatically cleaning up recorded lectures, there is still room for improvement and expansion. Here are some potential areas for future work and ideas for Autocut:

### **1\. Speech Enhancement**

In addition to removing unwanted sounds, Autocut could be expanded to enhance speech quality. For example, it could adjust the volume of the speaker's voice to make it more consistent or apply filters to improve clarity.

### **2\. Transcription**

Autocut could also be expanded to provide automatic transcription of recorded lectures. By combining speech recognition with labeling tasks, Autocut could generate a transcript of the lecture, making it easier for students to follow along and review the material.

### **3\. Integration with Learning Management Systems**

To make it even easier for teachers and professors to deliver high-quality course material, Autocut could be integrated with popular learning management systems like Blackboard, Canvas, or Moodle. This would allow teachers and professors to upload and clean up recorded lectures directly from their LMS, without the need for additional steps.

### **4\. Integration with Video Editing Software**

While Autocut is designed to be a standalone tool, it could also be integrated with popular video editing software like Adobe Premiere or Final Cut Pro. This would allow teachers and professors to clean up recorded lectures and make additional edits in the same software, streamlining the editing process even further.

Overall, there are many potential areas for future work and ideas for Autocut. As remote learning continues to be the norm, Autocut has the potential to become an essential tool for teachers and professors looking to deliver high-quality course material to their students.
