# MikeBot (3000)

MikeBot can be described as a boiling pot of different new AI models, loosely held together by a basic Python script, that can be used to generate short videos of a person (in this case, Michael Pound explaining a computing topic).

To warn you, the results shown in the video are **cherry picked**, the majority of the content produced by MikeBot is nonsense, mainly because (at the time of writing) most of these AI models cannot reliably produce coherent content of every potential video topic. However, these technologies are still very powerful, interesting and can generate amazing results given some oversight!

In this repo, I will briefly be explaining how each of the models works, how they function together and how you can get started using this yourself. I will **not** be providing the LORA or TTS weights for Michael Pound (for obvious reasons), but I will provide links to how you can train your own LoRA and finetuned TTS model. All models are open source and should work with a medium to high end graphics card (NVIDIA 30 series or higher).

**Code to be released soon (needs tidying up first)**

# Technologies

## ComfyUI (AI framework)

Almost all of this project was developed using ComfyUI, a fantastic framework for running AI models, that is extremely user friendly and requires little knowledge of machine learning. You can run these models in a *workflow*, which defines how different models should link together to produce a desired output. All workflows used in the project are stored in the *workflows* directory, which can run the different technologies below in ComfyUI. These workflows are very easy to run, all you need to do is download the required addons and click execute!

To download ComfyUI, you will need to visit their repository: https://github.com/comfyanonymous/ComfyUI

There is information on how to get this installed on their README page that I recommend following.

Tip: Set up a *virtual environment* and install the required package into this environment, this will save you a lot of trouble with installing the required models. You can find how to do this from the *manual-install* section of the README.

Once ComfyUI is configured, I highly recommend you install the **package manager**. This will allow you to install new external packages for ComfyUI without needing to manually download the code from GitHub and install the required python packages- this was a life saver. This can be downloaded here: https://github.com/Comfy-Org/ComfyUI-Manager. When executing a lot of the provided workflows, you will most likely encounter errors, as different packages will need to be installed. I recommend using the package manager to easily download these!

To install a new model in ComfyUI, you will almost always need to download different network weights. These network weights will need to be put in the *<your ComfyUI>/models/<model type>* directory. I will include links to explanations of how to do this for each model.

We used ComfyUI version **0.3.26**, although the newer versions *should* work fine with the provided workflows.

## Llama 3 (Large Language Model)

Large Language Models are used for generating content based on some input text string. For MikeBot, we wanted this to generate the required script, based on the type of video that the user requested. This script would contain a set of img prompts, audio prompts and video prompts, which we can then use to query the other models.

Llama3 was the LLM of choice for generating the content for each video. This can run using the Ollama software, which can be downloaded from their website: https://ollama.com/library/llama3. We used the 8B parameter model, as we found that this was more than good enough for generating the script for each video. By following the installation instructions on the website, you should be able to get this running.  You can check that Ollama is running by entering the following URL: http://127.0.0.1:11434/. 

You can then connect to Ollama by executing the Llama3 workflow: ```llm/generate_script_using_llm.json```. 

Llama3 accepts two different inputs: 
1) User Prompt- this is *what* you want Llama to generate (e.g. "Create a video explaining Or Gates")
2) System Prompt- this is *how* you want Llama to format the output (e.g. "Outputs should be only contain 2 sentences per scene and not contain too much technical language") 

Since the outputs need to be consistently good in order to get MikeBot to work, we had to use a complex system prompt to get the desired text for a variety of potential required videos.

## Flux (Image generator)

Image generators use diffusion, along with other optimisation techniques, to construct a image from noise. We chose to use Flux, as this model can produce a wide range of realistic images from a user prompt. Flux Dev was the model that we used, as it is free and produced good results.

To download and use Flux, I recommend having a look at the ComfyUI example page: https://comfyanonymous.github.io/ComfyUI_examples/flux/

Essentially, you will need to download the model weights, text encoders and VAE, and copy these to the respective directories in the ComfyUI directory. It is as easy as that to get this working! 

## Kohya_SS (LoRA)

The issue with standard image generators is that they cannot generate everything! Michael Pound, while very famous, is underrepresented in the training set. This means that Flux cannot generate a good image of Michael Pound, as it does not know how to create an image of him. Hence, we need to use a LoRA (Low Ranked Adapter). This model generates its own set of weights that can encode new styles or entities into the Flux network, meaning we can generate new concepts that are not known to the default Flux network. 

### Dataset Preperation

Before you can construct a new LoRA, you will need to generate a dataset to train this new model. For MikeBot, we used roughly 20 images of Mike, captured from various different ComputerPhile videos. Our images had a 512x512 resolution, but this can be increased if your graphics card can handle it!

Tip: I recommend using images that show the object/person in a variety of different poses and environments (which was tough for us since Mike films in his office most of the time)

For each image, you will need to generate a caption that describes what is in the image. You can use **Florence 2**, or some other image captioning model, to generate these captions for you (if you do not want to caption it yourself). You can use Florence 2 executing the following workflow: ```img/florence2_img_captioning.json```. This model can be downloaded here: https://github.com/kijai/ComfyUI-Florence2

For your new object/person, you will need to assign a new *token* (character string) that the network can learn to associate with the new concept in the images. For MikeBot, we used the token ```MichaelPound```. What this means is that when we use the token *Michael Pound* in the image prompt, the Flux model (with the LoRA) knows that it needs to generate a new image of Mike. We chose to concatenate the two words together, since it makes it easier to train the LoRA since *MichaelPound* is a unique string. 

So, to put this all together, your dataset should consist of a series of images and captions in the following format:

```
+--- Dataset
+------ 0.png
+------ 0.txt
+------ 1.png
+------ 1.txt
...
+------ 20.png
+------ 20.txt
```

Each caption should be stored as a text file with the same name as the image. An example of a caption that we used is:

```
The image is a screenshot from a video interview. It shows MichaelPound sitting in a chair in an office setting. He is wearing a blue t-shirt and has short dark hair. 
He appears to be in his late 30s and is looking directly at the camera with a serious expression on his face. His hands are gesturing with his fingers as if he is explaining something. 
The background shows a window with blinds and a desk with a computer monitor. 
```

### Training

Now that you have your dataset, you will need to train the LoRA to learn what your new object/person is. We used **Kohya_SS**, which is a great framework for executing various diffusion models. You will need to download this framework and setup a new virtual environment to train your LoRA. 

This can be downloaded here: https://github.com/bmaltais/kohya_ss?tab=readme-ov-file#lora. I highly recommend reading their LoRA training guide and options. Here is another reddit post that I found helpful: https://www.reddit.com/r/StableDiffusion/comments/1h1hk0o/sharing_my_flux_lora_training_configuration_for/ 

Training a new LoRA took around 6 hours on my PC, and when completed, will output a set of network weights which you can then use in ComfyUI. You can use the LoRA with Flux by executing the included Flux Dev workflow: ```img/flux_dev.json```. You will need to change the *directory path* in the LoRA node to the network weights of your new trained LoRA. Now, by including your token in the prompt, you will be able to generate new images of your object/person/style (as well as all other image types that have been trained by Flux).

Tip: You will need to increase/decrease the strength of the LoRA until you are happy with the image results.

## WAN 2.1 (Video)

Now that you can generate new images of your object/person, we require an img-to-video generator that can take the image, as well as a caption, and animate it correctly. We experimented with various different video models, and we found that **WAN 2.1** produced the best results (at the time). 

To download and use WAN 2.1, I recommend having a look at the ComfyUI example page: https://comfyanonymous.github.io/ComfyUI_examples/wan/

Similarly to FLUX, all you need to do is the model weights, text encoders and VAE, and copy these to the respective directories in the ComfyUI directory.

## FILM (Frame interpolation) and 4xLSDIR (Upscaling)

While these open source video models are good, the output video can be lacking in quality, with low frame rate and resolution. Hence, we used frame interpolation and upscaling models to achieve better quality. These models are:

1) **Film_net** accepts a video and doubles the frame rate: https://huggingface.co/nguu/film-pytorch/blob/887b2c42bebcb323baf6c3b6d59304135699b575/film_net_fp32.pt

2) **4xLSDIR** accepts a low quality image and quadruples the resolution: https://openmodeldb.info/models/4x-LSDIR

I **highly recommend** checking out this post for more information: https://civitai.com/articles/12072

You can run WAN 2.1 with frame interpolation and upscaling by executing the following workflow:  ```video/wan_video_with_upscale_and_frameInterpolation.json```

## F5-TTS (Audio)

###  Generating Audio

To produce effective videos of Mike explaining a video topic, we need audio that sounds realistic and similar to Mike. We chose to use **F5-TTS**, an extremely powerful model that can generate audio with only 10 seconds of initial audio! To get started with this model, you will need roughly 10 seconds of audio of the person who you are trying to clone, and a text that matches this audio. For Mike, we used a clip of him explaining matrices, with the following text caption: 

```If you think that, one of these networks might have thousands and thousands of very large matrix multiplications, where each of your matrices is two or four thousand by four thousand```

Tip: Pick audio that has little background noise and where the person speaking has a tempo/pitch that best matches their normal voice.

To get started with using this, I recommend checking out this **fantastic** reddit post: https://www.reddit.com/r/StableDiffusion/comments/1id8spa/effortlessly_clone_your_own_voice_by_using/

### Fine-tuning

While results from this model are very impressive, it may struggle with pronouncing certain words/phrases that sound like person you are trying to clone. This is simply because you do not have enough audio. Hence, we will need to finetune the model on more audio to get consistently good results.

We fine-tuned the F5-TTS base model on roughly 1.5 hours of Michael Pound talking. This was pretty tedious to acquire, as I had to make sure that no background noise was included in the audio. All of this audio was concatenated into a large .mp3 file, which can then be used for fine-tuning.

F5-TTS offer a Gradio app for achieving this, I recommend checking out their official documentation which explains how you can fine-tune your model: https://github.com/SWivid/F5-TTS/tree/main/src/f5_tts/train

Also, I found the video tutorial on this post to be **very helpful**: https://github.com/SWivid/F5-TTS/discussions/143

Tip: Do not train for the entire number of epochs that it recommends — we found we got good results after around 1500 epochs (but this will vary depending on the amount of audio you want to train on)

To run F5-TTS audio, you can execute the workflow: ```video/f5-tts.json```. You will need to provide 10 seconds of audio and the associated text. To use your fine-tuned model, you will need to change *model* in the *F5TTSAudioInputs* node to your *.pt* file

## LatentSync (Lip Syncing)

Now that we have audio of Mike, we need the lips of our Michael Pound video to match the audio. To achieve this, we used **LatentSync**, an open source lip syncing model, which produced excellent results! 

To get this working in ComfyUI, download the following repo into your ComfyUI custom node: https://github.com/ShmuelRonen/ComfyUI-LatentSyncWrapper

I recommend watching this video to get started: https://www.youtube.com/watch?v=roINgIOhybA

You can run Latent Sync with frame interpolation by executing the following workflow: ```video/latentsync_with_frame_interpolation.json```

# How to Run MikeBot

**Code to be released soon (needs tidying up first)**

# Closed Source 

While these open source models are excellent, you may find that alternative closed source models might be better suited for what you require. Hence, here is a list of closed source models that you may find useful:

ChatGPT 4o (Text to Image with LLM support): https://openai.com/index/hello-gpt-4o/

Veo 3 (Text to Video/Audio): https://deepmind.google/models/veo/

Kling (Text to Video): https://www.klingai.com/global/

Hedra Character-3 (Image/Text/Audio to full video): https://www.hedra.com/

Udio (Text to Music): https://www.udio.com/

# Disclaimer

Hopefully I have credited all the technologies that were used for the creation of MikeBot, but if I haven't, then please let me know!

**Please use this responsibly.** While these new AI models are impressive and fun to experiment with, they can cause harm if misused. Avoid using the technologies demonstrated in this video or repository for any harmful or unethical purposes.