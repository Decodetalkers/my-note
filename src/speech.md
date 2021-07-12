# 记捣鼓python 语音api的过程

想做一个游戏王的app啥的，flutter上有个讯飞api，但是并不能用

python是有非常丰富的api，比如speechrecognition。

## 安装
```bash
cd ~/pythonproject
mkdir voice
cd voicd
python -m venv snake
source snake/bin/activate
```
这部分生成一个新的虚拟环境，然后容器里的pip可能版本低，可以upgrade一下

之后有需要用到一个调用麦克风的库，这个pip库需要在安装时候make，是个cpython项目，先得把它需要的头文件准备好。我是debian，其他发行版看着办

```bash
sudo apt install portaudio19-dev
pip install pyaudio
```

## 使用

然后就可以写代码了，无论是写py文件还是shell随便。

这个语音是英文的语音api，所以需要准备英文的wav。我事先在github下了一个

https://github.com/realpython/python-speech-recognition

```python3
import speech_recognition as sr

r = sr.Recognizer
harvard = sr.AudioFile('harvard.wav')
with harvard as source:
    audio = r.record(source)

r.recognize_google(audio)
```

## 麦克风

还是当前环境，

```bash
python -m speech_recognition
```

然后就可以对话啦
