# Open-LLM-VTuber

> :warning: This project is in its early stages and is currently under **active development**. Features are unstable, code is messy, and breaking changes will occur. The main goal of this stage is to build a minimum viable prototype using technologies that are easy to integrate.

Open-LLM-VTuber allows you to talk to any LLM by voice locally with a Live2D talking face. The LLM inference backend, speech recognition, and text synthesizer are all designed to be swappable. This project can be configured to run offline (with a small exception...) on macOS, Linux, and Windows. 

Long-term memory with MemGPT can be configured to achieve perpetual chat, infinite* context length, and external data source.

This project started as an attempt to recreate the closed-source AI VTuber `neuro-sama` with open-source alternatives that can run offline on platforms other than Windows.


<img width="500" alt="demo-image" src="https://github.com/t41372/Open-LLM-VTuber/assets/36402030/fa363492-7c01-47d8-915f-f12a3a95942c"/>





https://github.com/t41372/Open-LLM-VTuber/assets/36402030/e8931736-fb0b-4cab-a63a-eea5694cbb83
- The demo video uses llama3 (Q4_0) with Ollama, edgeTTS, and WhisperCPP running the coreML version of small model.





### Why this project and not other similar projects on GitHub?
- It works on macOS
  - Many existing solutions display Live2D models using VTube Studio and achieve lip sync by routing desktop audio into VTube Studio. They let VTuber Studio listen to the desktop's internal audio and control the lips with that. On macOS, however, there is no easy way to let VTuber Studio listen to internal audio on the desktop.
- It supports [MemGPT](https://github.com/cpacker/MemGPT) for perpetual chat. The chatbot remembers you and what you've said.
- No data leaves your computer if you wish to
  - Once the live2D model is loaded, you can unplug the cable and enjoy total privacy. You can choose local solutions for LLM/voice recognition/speech synthesis. In addition, everything works on macOS.






### Goal
- [x] Chat with LLM by voice
- [x] Choose your own LLM backend
- [x] Choose your own Speech Recognition & Text to Speech provider
- [x] Long-term memory
- [x] Live2D frontend

### Target Platform
- macOS
- Windows
- Linux



## Implemented Features

- Talk to LLM with voice. Offline.
- RAG on chat history *(needs to be rewritten to be compatible with many other features, so turn this off for now)*

Currently supported LLM backend
- Ollama
- Any OpenAI-API-compatible backend, such as Groq, LM Studio, OpenAI, and more.
- MemGPT (setup required)

Currently supported Speech recognition backend
- [Faster-Whisper](https://github.com/SYSTRAN/faster-whisper) (Local)
- [Whisper-CPP](https://github.com/ggerganov/whisper.cpp) using the python binding [pywhispercpp](https://github.com/abdeladim-s/pywhispercpp) (Local, mac GPU acceleration can be configured)
- [Whisper](https://github.com/openai/whisper) (local)
- [Azure Speech Recognition](https://azure.microsoft.com/en-us/products/ai-services/speech-to-text) (API Key required)
- The microphone will be listening in the terminal by default. You can change the settings in the `conf.yaml` to move the microphone (and vad) to the browser (at the cost of latency, for now). Microphone listening in the terminal means the program can't hear you if you open the Live2D front-end website on a different device or run this project inside a VM or docker. Set `MIC_IN_BROWSER` to `True` in this case.

Currently supported Text to Speech backend
- [py3-tts](https://github.com/thevickypedia/py3-tts) (Local, it uses your system's default TTS engine)
- [bark](https://github.com/suno-ai/bark) (Local, very resource-consuming)
- [Edge TTS](https://github.com/rany2/edge-tts) (online, no API key required)
- [Azure Text-to-Speech](https://azure.microsoft.com/en-us/products/ai-services/text-to-speech) (online, API Key required)

Fast Text Synthesis
- Synthesize sentences as soon as they arrive, so there is no need to wait for the entire LLM response.
- Synthesize text during audio playback. It's synthesizing new lines while speaking.

Live2D Talking face
- Change Live2D model with `config.yaml` (model needs to be listed in model_dict.json)
- Load local Live2D models. Check `doc/live2d.md` for documentation.
- Uses expression keywords in LLM response to control facial expression, so there is no additional model for emotion detection. The expression keywords are automatically loaded into the system prompt and excluded from the speech synthesis output.

live2d technical details
- Uses [guansss/pixi-live2d-display](https://github.com/guansss/pixi-live2d-display) to display live2d models in *browser*
- Uses WebSocket to control facial expressions and talking state between the server and the front end
- The Live2D implementation in this project is currently in its early stages. It currently requires an internet connection to load the required front-end packages from CDN. Once the page is loaded, you can disconnect the internet. Some Live2D models need to be fetched from CDN; some don't. Read `doc/live2d.md` for documentation on loading your live2D model from local.
- Run the `server.js` to run the WebSocket communication server, open the `index.html` in the `./static` folder to open the front end, and run `launch.py` to run the backend for LLM/ASR/TTS processing.

## Install & Usage

Install FFmpeg on your computer.

Clone this repository.

You need to have [Ollama](https://github.com/jmorganca/ollama) or any other OpenAI-API-Compatible backend ready and running. If you want to use MemGPT as your backend, scroll down to the MemGPT section.

Prepare the LLM of your choice. Edit the BASE_URL and MODEL in the project directory's `conf.yaml`.


This project was developed using Python `3.10.13`. I strongly recommend creating a virtual Python environment like conda for this project. 

Run the following in the terminal to install the dependencies.

~~~shell
pip install -r requirements.txt # Run this in the project directory
# Install Speech recognition dependencies and text-to-speech dependencies according to the instructions below
~~~

This project, by default, launches the audio interaction mode, meaning you can talk to the LLM by voice, and the LLM will talk back to you by voice.

Edit the `conf.yaml` for configurations. You may want to set the speech recognition to faster-whisper, text-to-speech to edgeTTS, live2d to True, and Rag to off to achieve a similar effect to the demo. I recommend turning Rag off because it's incompatible with many new features at the moment.

If you want to use live2d, run `server.py` to launch the WebSocket communication server and open the URL you set in `conf.yaml` (`http://HOST:PORT`). By default, go to `http://localhost:8000`.

Run `launch.py` with python. Some models will be downloaded during your first launch, which may take a while.

Also, the live2D models have to be fetched through the internet, so you'll have to keep your internet connected before the `index.html` is fully loaded with your desired live2D model.



### Update
Back up the configuration files `conf.yaml` and `memgpt_config.yaml` if you've edited them and do `git fetch` and `git pull`.
Or just clone the repo again and make sure to either transfer your config files or set those configurations again.




## Install Speech Recognition
Edit the STT_MODEL settings in the `conf.yaml` to change the provider.

Here are the options you have for speech recognition:

`Faster-Whisper` (local)
- Whisper, but faster. On macOS, it runs on CPU only, which is not so fast, but it's easy.

`WhisperCPP` (local) (runs super fast on a Mac if configured correctly)
- Install the package by running `pip install pywhispercpp`
- The whisper cpp python binding. It can run on coreML with configuration, which makes it very fast on macOS.
- On CPU or Nvidia GPU, it's probably slower than Faster-Whisper

WhisperCPP coreML configuration:
- Uninstall the original `pywhispercpp` if you have already installed it. We are building the package.
- Run `install_coreml_whisper.py` with Python to automatically clone and build the coreML-supported `pywhispercpp` for you.
- Prepare the appropriate coreML models.
  - You can either convert models to coreml according to the documentation on Whisper.cpp repo
  - ...or you can find some [magical huggingface repo](https://huggingface.co/chidiwilliams/whisper.cpp-coreml/tree/main) that happens to have those converted models. Just remember to decompress them. If the program fails to load the model, it will produce a segmentation fault.
  - You don't need to include those weird prefixes in the model name in the `conf.yaml`. For example, if the coreml model's name looks like `ggml-base-encoder.mlmodelc`, just put `base` into the `model_name` under `WhisperCPP` settings in the `conf.yaml`.

`Whisper` (local)
- Original Whisper from OpenAI. Install it with `pip install -U openai-whisper`
- The slowest of all. Added as an experiment to see if it can utilize macOS GPU. It didn't.

`AzureSTT` (online, API Key required)
- Azure Speech Recognition. Install with `pip install azure-cognitiveservices-speech`.
- API key and internet connection are required.

## Install Speech Synthesis (text to speech)
Install the respective package and turn it on using the `TTS_MODEL` option in `conf.yaml`.

`pyttsx3TTS` (local, fast)
- Install with the command `pip install py3-tts`.
- This package will use the default TTS engine on your system. It uses `sapi5` on Windows, `nsss` on Mac, and `espeak` on other platforms.
- `py3-tts` is used instead of the more famous `pyttsx3` because `pyttsx3` seems unmaintained, and I couldn't get the latest version of `pyttsx3` working.



`barkTTS` (local, slow)
- Install the pip package with this command `pip install git+https://github.com/suno-ai/bark.git` and turn it on in `conf.yaml`.
- The required models will be downloaded on the first launch.

`edgeTTS` (online, no API key required)
- Install the pip package with this command `pip install edge-tts` and turn it on in `conf.yaml`.
- Sounds pretty good. Runs pretty fast.
- Remember to connect to the internet when using edge tts.

`AzureTTS` (online, API key required)
- See below

### Azure API for Speech Recognition and Speech to Text, API key needed

Create a file named `api_keys.py` in the project directory, paste the following text into the file, and fill in the API keys and region you gathered from your Azure account.

~~~python
# Azure API key
AZURE_API_Key="YOUR-API-KEY-GOES-HERE"

# Azure region
AZURE_REGION="YOUR-REGION"

# Choose the Text to speech model you want to use
AZURE_VOICE="en-US-AshleyNeural"
~~~



If you're using macOS, you need to enable the microphone permission of your terminal emulator (you run this program inside your terminal, right? Enable the microphone permission for your terminal). If you fail to do so, the speech recognition will not be able to hear you because it does not have permission to use your microphone.


## MemGPT
> MemGPT integration is very experimental and requires quite a lot of setup. In addition, MemGPT requires a powerful LLM (larger than 7b and quantization above Q5) with a lot of token footprint, which means it's a lot slower.
> MemGPT does have its own LLM endpoint for free, though. You can test things with it. Check their docs.

This project can use [MemGPT](https://github.com/cpacker/MemGPT) as its LLM backend. MemGPT enables LLM with long-term memory.

To use MemGPT, you need to have the MemGPT server configured and running. You can install it using `pip` or `docker` or run it on a different machine. Check their [GitHub repo](https://github.com/cpacker/MemGPT) and [official documentation](https://memgpt.readme.io/docs/index).

> :warning:
> I recommend you install MemGPT either in a separate Python virtual environment or in docker because there is currently a dependency conflict between this project and MemGPT (on fast API, it seems). You can check this issue [Can you please upgrade typer version in your dependancies #1382](https://github.com/cpacker/MemGPT/issues/1382).


Here is a checklist:
- Install memgpt
- Configure memgpt
- Run `memgpt` using `memgpt server` command. Remember to have the server running before launching Open-LLM-VTuber.
- Set up an agent either through its cli or web UI. Add your system prompt with the Live2D Expression Prompt and the expression keywords you want to use (find them in `model_dict.json`) into MemGPT
- Copy the `server admin password` and the `Agent id` into `./llm/memgpt_config.yaml`
- Set the `LLM_PROVIDER` to `memgpt` in `conf.yaml`. 
- Remember, if you use `memgpt`, all LLM-related configurations in `conf.yaml` will be ignored because `memgpt` doesn't work that way.



# Development
(this project is in the active prototyping stage, so many things will change)

### [Outdated] How to add support for new TTS provider
1. Create the new class `TTSEngine` in a new py file in the `./tts` directory
2. In the class, expose a speak function: `speak` and `speak_stream` functions. Read the `pyttsx3TTS.py` for reference.
3. Add your new tts module into the `tts_module_name` dictionary, which is currently hard-coded in the `main.py` and `launch.py` (I plan to ditch the main.py in the future). The dictionary key is the name of the TTS provider. The value is the module path of your module.
4. Now you should be able to switch to the tts provider of your choice by editing the `conf.yaml`
5. Create a pull request

### [Outdated] How to add support for new Speech Recognition (or speech-to-text, STT) provider
1. Create the new class `VoiceRecognition` in a new py file in the `./speech2text` directory
2. In the class, expose a function: `transcribe_once(self)` that starts the voice recognition service. This function should be able to keep listening in the background, and when the user speaks something and finishes, the function should return the recognized text.
3. Add your new stt module into the `stt_module_name` dictionary, which is currently hard-coded in the `main.py` and `launch.py` (I plan to ditch the main.py in the future). The key of the dictionary is the name of the speech recognition provider. The value is the module path of your module.
4. Now you should be able to switch to the stt provider of your choice by editing the `conf.yaml`
5. Create a pull request

### Add support for new LLM provider
Todo...

# Acknowledgement
Awesome projects I learned from

- https://github.com/dnhkng/GlaDOS
- https://github.com/SchwabischesBauernbrot/unsuperior-ai-waifu
- https://codepen.io/guansss/pen/oNzoNoz
- https://github.com/Ikaros-521/AI-Vtuber
- https://github.com/zixiiu/Digital_Life_Server





