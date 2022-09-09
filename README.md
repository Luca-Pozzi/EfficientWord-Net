# EfficientWord-Net
![Versions : 3.6 ,3.7,3.8,3.9](https://camo.githubusercontent.com/a7b5b417de938c1faf3602c7f48f26fde8761a977be85390fd6c0d191e210ba8/68747470733a2f2f696d672e736869656c64732e696f2f707970692f707976657273696f6e732f74656e736f72666c6f772e7376673f7374796c653d706c6173746963)

## Hotword detection based on one-shot learning

Home assistants require special phrases called hotwords to get activated (eg:"ok google")

EfficientWord-Net is an hotword detection engine based on one-shot
learning inspired from FaceNet's Siamese Network Architecture.
Works very similar to face recognition , just requires a few samples of your own custom hotword to get going. 
**No extra training or huge datasets required!!**
This will allow developers to add custom hotwords to their programs without a sweat or any extra charges.
Just like google assistant's hotword detector, the engine performs the best when 3-4 hotword samples are collected directly from the user
This repository is an official implemenation of EfficientWord-Net as
a python library from the authors.

The library is purely written with python and uses Google's Tflite
implemenation for faster realtime inference.

### Demo of EfficientWord-Net in Pi

https://user-images.githubusercontent.com/44740048/139785995-3330d65a-cfe1-4e92-8769-ee389a122acc.mp4

## Access paper

[Click here](https://worldscientific.com/doi/10.1142/S0219649222500599) to access the research paper.
<br>

## Python Version Requirements

This Library works between python versions:
    `3.6 to 3.9`
<br>

## Dependencies Installation
Before running the pip installation command for the library, few dependencies need to be installed manually.

* [PyAudio (depends on PortAudio)](https://abhgog.gitbooks.io/pyaudio-manual/content/installation.html)
* [Tflite (tensorflow lightweight binaries)](https://www.tensorflow.org/lite/guide/python#install_tensorflow_lite_for_python)
* [Librosa (Binaries might not be available for certain systems)](https://github.com/librosa/librosa)
Mac OS M* and Raspberry Pi users might have to compile these dependecies.

***tflite*** package cannot be listed in requirements.txt hence will be automatically installed when the package is initialized in the system.

***librosa*** package is not required for inference only cases , however when generate_reference is called , will be automatically installed.

<br>

## Package Installation
Run the following pip command

```
pip install EfficientWord-Net
```

and to import running

```
import eff_word_net
```
<br>

## Demo
After installing the packages, you can run the Demo
script inbuilt with library (ensure you have a working mic).

Accesss Documentation from : https://ant-brain.github.io/EfficientWord-Net/

Command to run demo
```
python -m eff_word_net.engine
```
<br>

## Generating Custom Wakewords

For any new hotword, the library needs information about the hotword, this
information is obtained from a file called `{wakeword}_ref.json`. 
Eg: For the wakeword 'alexa', the library would need the file called `alexa_ref.json`

These files can be generated with the following procedure:

One needs to collect few 4 to 10 uniquely sounding pronunciations
of a given wakeword. Then put them into a seperate folder, which doesnt contain 
anything else.

Or one could use the following command to generate audio files for a given word, uses ibm neural tts demo api, Kindly dont over use it for our sake (lol)

```bash
python -m eff_word_net.ibm_generate
```

Finally run this command, it will ask for the input folder's location 
(containing the audio files) and the output folder (where _ref.json file will be stored).
```
python -m eff_word_net.generate_reference
```

The pathname of the generated wakeword needs to passed to the HotwordDetector detector instance.

```python
HotwordDetector(
        hotword="hello",
        reference_file = "/full/path/name/of/hello_ref.json"),
        threshold=0.9, #min confidence required to consider a trigger
        relaxation_time = 0.8 #default value ,in seconds
)
```
relaxation time parameter is used to determine the min time between any 2 triggers, any potential triggers before the relaxation_time will be cancelled

The detector operates on a sliding widow approach resulting in multiple triggers for single utterance of a hotword, the relaxation_time parameter can used to control the multiple triggers, in most cases 0.8sec(default) will do 

<br>

## Out of the box sample hotwords
Few wakewords such as **Mycroft**, **Google**, **Firefox**, **Alexa**, **Mobile**, **Siri** the library has predefined embeddings readily available in the library installation directory, its path is readily available in the following variable

```python
from eff_word_net import samples_loc
```

<br>

## Try your first single hotword detection script

```python
import os
from eff_word_net.streams import SimpleMicStream
from eff_word_net.engine import HotwordDetector
from eff_word_net import samples_loc

mycroft_hw = HotwordDetector(
        hotword="Mycroft",
        reference_file = os.path.join(samples_loc,"mycroft_ref.json"),
    )

mic_stream = SimpleMicStream()
mic_stream.start_stream()

print("Say Mycroft ")
while True :
    frame = mic_stream.getFrame()
    result = mycroft_hw.scoreFrame(frame)
    if result==None :
        #no voice activity
        continue
    if(result["match"]):
        print("Wakeword uttered",result["confidence"])

```
<br>


## Detecting Mulitple Hotwords from audio streams

The library provides a computation friendly way 
to detect multiple hotwords from a given stream, instead of running `scoreFrame()` of each wakeword individually

```python
import os
from eff_word_net.streams import SimpleMicStream
from eff_word_net import samples_loc
print(samples_loc)

alexa_hw = HotwordDetector(
        hotword="Alexa",
        reference_file = os.path.join(samples_loc,"alexa_ref.json"),
    )

siri_hw = HotwordDetector(
        hotword="Siri",
        reference_file = os.path.join(samples_loc,"siri_ref.json"),
    )

mycroft_hw = HotwordDetector(
        hotword="mycroft",
        reference_file = os.path.join(samples_loc,"mycroft_ref.json"),
        activation_count=3
    )

multi_hw_engine = MultiHotwordDetector(
        detector_collection = [
            alexa_hw,
            siri_hw,
            mycroft_hw,
        ],
    )

mic_stream = SimpleMicStream()
mic_stream.start_stream()

print("Say Mycroft / Alexa / Siri")

while True :
    frame = mic_stream.getFrame()
    result = multi_hw_engine.findBestMatch(frame)
    if(None not in result):
        print(result[0],f",Confidence {result[1]:0.4f}")

```
<br>

Access documentation of the library from here : https://ant-brain.github.io/EfficientWord-Net/


## Change notes from v0.1.1 to 0.2.2
major changes to replace complex friking logic of handling poly triggers per utterance into more simpler logic and more simpler api for programmers

Introduces breaking changes

## Limitations in Current model
- trained on single words , hence may result in bizare behaviour on using phrases like "Hey xxx"
- audio processing window limited to 1 sec. Hence will not work effectively for longer hotwords
 
## FAQ :
* **Hotword Perfomance is bad** : if you are having some issue like this , feel to ask the same in [discussions](https://github.com/Ant-Brain/EfficientWord-Net/discussions/4)

## CONTRIBUTION:
* If you have an ideas to make the project better, feel free to ping us in [discussions](https://github.com/Ant-Brain/EfficientWord-Net/discussions/3)
* The current [logmelcalc.tflite](/eff_word_net/logmelcalc.tflite) graph can convert only 1 audio frame to Log Mel Spectrogram at a time. It will be of a great help if tensorflow guru's outthere help us out with this.

## TODO :

* Add audio file handler in streams. PR's are welcome.
* Remove librosa requirement to encourage generating reference files directly in edge devices
* Add more detailed documentation explaining slider window concept

## SUPPORT US:
Our hotword detector's performance is notably low when compared to Porcupine. We have thought about better NN architectures for the engine and hope to outperform Porcupine. This has been our undergrad project. Hence your support and encouragement will motivate us to develop the engine. If you loved this project recommend this to your peers, give us a 🌟 in Github and a clap 👏 in [medium](https://link.medium.com/yMBmWGM03kb).

## LICENCSE : [Apache License 2.0](/LICENSE.md)
