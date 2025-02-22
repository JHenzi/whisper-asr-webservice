# Whisper ASR Webservice

![Whisper ASR Webservice](https://github.com/ahmetoner/whisper-asr-webservice/blob/main/docs/assets/img/banner.png?raw=true)
![Release](https://img.shields.io/github/v/release/ahmetoner/whisper-asr-webservice.svg)
![Docker Pulls](https://img.shields.io/docker/pulls/onerahmet/openai-whisper-asr-webservice.svg)
![Build](https://img.shields.io/github/actions/workflow/status/ahmetoner/whisper-asr-webservice/docker-publish.yml.svg)
![Licence](https://img.shields.io/github/license/ahmetoner/whisper-asr-webservice.svg)

Whisper is a general-purpose speech recognition model. It is trained on a large dataset of diverse audio and is also a multi-task model that can perform multilingual speech recognition as well as speech translation and language identification. For more details: [github.com/openai/whisper](https://github.com/openai/whisper/)

## Notice

This repository is a **fork** of [github.com/ahmetoner/whisper-asr-webservice](https://github.com/ahmetoner/whisper-asr-webservice) and is will face a revamp in 2023. The original repository stands as a proof of concept for automatic speech recognition. Our repository will focus on collecting and documenting **utterances** by a user, writing them to a database and providing output in formats that can be read by other applications (e.g. *markdown, docx, pdf, etc*).

### General Roadmap

The project should be able to:

- [ ] Accept audio files from a user
- [ ] Transcribe the audio files and write them to a database
- [ ] Add new transcriptions to a pipeline for topic modeling
- [ ] Update database with topic modeling results

In addition, the project needs to consider or create a user interface for browswing, searching and reading the utterances in the database. The user interface should be able to:

- [ ] Search for utterances by date, time, topic, etc.
- [ ] Give a user a list of topics to choose from
- [ ] Given a selected topic the user should be able to see a list of related utterances (*or see related sub-topics*)

### Updates Needed

- [ ] Add support for MongoDB via PyMongo
- [ ] Create docker-compose file for building Docker images from source(s)
- [ ] Add topic modeling job to the pipeline

### Potential Divergence from Original Repository

- [ ] Drop support for non-English languages
- [ ] Optimize for CPU inference
- [ ] Optimize for VM usage/VPS hosting
- [ ] *General removal of various extraneous endpoints*

## Usage [Pre-Fork]

Whisper ASR Webservice now available on Docker Hub. You can find the latest version of this repository on docker hub for CPU and GPU.

Docker Hub: <https://hub.docker.com/r/onerahmet/openai-whisper-asr-webservice>

For CPU:

```sh
docker run -d -p 9000:9000 -e ASR_MODEL=base onerahmet/openai-whisper-asr-webservice:latest
```

For GPU:

```sh
docker run -d --gpus all -p 9000:9000 -e ASR_MODEL=base onerahmet/openai-whisper-asr-webservice:latest-gpu
```

For MacOS (CPU only):

GPU passthrough does not work on macOS due to fundamental design limitations of Docker. Docker actually runs containers within a LinuxVM on macOS. If you wish to run GPU-accelerated containers, I'm afraid Linux is your only option.

The `:latest` image tag provides both amd64 and arm64 architectures:

```sh
docker run -d -p 9000:9000 -e ASR_MODEL=base onerahmet/openai-whisper-asr-webservice:latest
```

```sh
# Interactive Swagger API documentation is available at http://localhost:9000/docs
```
![Swagger UI](https://github.com/ahmetoner/whisper-asr-webservice/blob/main/docs/assets/img/swagger-ui.png?raw=true)

Available ASR_MODELs are `tiny`, `base`, `small`, `medium`, `large`, `large-v1` and `large-v2`. Please note that `large` and `large-v2` are the same model.

For English-only applications, the `.en` models tend to perform better, especially for the `tiny.en` and `base.en` models. We observed that the difference becomes less significant for the `small.en` and `medium.en` models.

## Run (Development Environment) [Pre-Fork]

Install poetry with following command:

```sh
pip3 install poetry
```

Install torch with following command:

```sh
# just for GPU:
pip3 install torch==1.13.0+cu117 -f https://download.pytorch.org/whl/torch
```

Install packages:

```sh
poetry install
```

Starting the Webservice:

```sh
poetry run gunicorn --bind 0.0.0.0:9000 --workers 1 --timeout 0 app.webservice:app -k uvicorn.workers.UvicornWorker
```

## Quick start [Pre-Fork]

After running the docker image interactive Swagger API documentation is available at [localhost:9000/docs](http://localhost:9000/docs)

There are 2 endpoints available:

- /asr (TXT, VTT, SRT, TSV, JSON)
- /detect-language

## Automatic Speech recognition service /asr [Pre-Fork]

If you choose the **transcribe** task, transcribes the uploaded file. Both audio and video files are supported (as long as ffmpeg supports it).

Note that you can also upload video formats directly as long as they are supported by ffmpeg.

You can get TXT, VTT, SRT, TSV and JSON output as a file from /asr endpoint.

You can provide the language or it will be automatically recognized.

If you choose the **translate** task it will provide an English transcript no matter which language was spoken.

Returns a json with following fields:

- **text**: Contains the full transcript
- **segments**: Contains an entry per segment. Each entry  provides time stamps, transcript, token ids and other metadata
- **language**: Detected or provided language (as a language code)

## Language detection service /detect-language [Pre-Fork]

Detects the language spoken in the uploaded file. For longer files it only processes first 30 seconds.

Returns a json with following fields:

- **detected_language**
- **language_code**

## Build  [Pre-Fork]

Build .whl package

```sh
poetry build
```

Configuring the Model

```sh
export ASR_MODEL=base
```

## Docker Build [Pre-Fork]

### For CPU [Pre-Fork]

```sh
# Build Image
docker build -t whisper-asr-webservice .

# Run Container
docker run -d -p 9000:9000 whisper-asr-webservice
# or
docker run -d -p 9001:9000 -e ASR_MODEL=base whisper-asr-webservice3
```

### For GPU [Pre-Fork]

```sh
# Build Image
docker build -f Dockerfile.gpu -t whisper-asr-webservice-gpu .

# Run Container
docker run -d --gpus all -p 9000:9000 whisper-asr-webservice-gpu
# or
docker run -d --gpus all -p 9000:9000 -e ASR_MODEL=base whisper-asr-webservice-gpu
```

## Cache [Pre-Fork]
The ASR model is downloaded each time you start the container, using the large model this can take some time. If you want to decrease the time it takes to start your container by skipping the download, you can store the cache directory (/root/.cache/whisper) to an persistent storage. Next time you start your container the ASR Model will be taken from the cache instead of being downloaded again.

**Important this will prevent you from receiving any updates to the models.**
 
```sh
docker run -d -p 9000:9000 -e ASR_MODEL=large -v //c/tmp/whisper:/root/.cache/whisper onerahmet/openai-whisper-asr-webservice:latest
```


## TODO [Pre-Fork]

- Unit tests
- Recognize from path
- Batch recognition from given path/folder
- Live recognition support with HLS
- gRPC support
