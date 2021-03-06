# One Model, Many Languages: Meta-learning for Multilingual Text-to-Speech  
  
Tomáš Nekvinda, Ondřej Dušek  
Charles University  
INTERSPEECH, 2020  
  
## Reference
  
* [ArXiv](https://arxiv.org/abs/2008.00768)  
* [Source Code](https://github.com/Tomiinek/Multilingual_Text_to_Speech)  
* [Demo Webpage](https://tomiinek.github.io/multilingual_speech_samples/)  
* [Tacotron](https://arxiv.org/abs/1703.10135), [Tacotron2](https://arxiv.org/abs/1712.05884)   
* [DC-TTS](https://arxiv.org/pdf/1710.08969.pdf)
* [CSS 10 Dataset](https://github.com/Kyubyong/css10)  
* [Common Voice Dataset](https://commonvoice.mozilla.org/en/datasets) 
  
## Summary
  
* Multilingual Speech Synthesis  
* Meta-learning   
* Cross-lingual
* Voice Cloning : Speech in multiple languages with the same voice  
* Code switching : Speak two (or more) languages with a single utterance.  
  
## TTS System  
  
<img src="https://user-images.githubusercontent.com/42150335/102853834-74af5c80-4464-11eb-9609-6f815e67b25a.png" width=700>  
  
- Mel-Predict Network: 생성하고 싶은 텍스트를 입력으로 받아 Mel-Spectrogram을 생성하는 네트워크  
- Vocoder: Mel-Predict Network가 생성한 Mel-Spectrogram을 입력으로 받아 음성신호(Waveform)로 생성하는 네트워크
  
## Tacotron
  
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FwnGyQ%2FbtqDblNauXg%2FJsSXkwgQY1yc3lIHtdgIP0%2Fimg.png" width=700>

* 딥러닝 기반 음성합성의 대표적인 모델  
* Attention + Sequence-to-Sequence의 TTS 버전  
* Griffin-Lim Vocoder 사용 (빠르지만 성능은 좋지 못함)  
  
## DC-TTS 
  
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FxJaRU%2FbtqL3vzi2G6%2FipgAflHEiTfvSor836jJmk%2Fimg.png" width=700>
  
- Tacotron과 Tacotron2 사이에 발표  
- DC-TTS (Deep-Convolutional TTS): LSTM, GRU 같은 RNN 기반 레이어가 없는 것이 특징  
- 모든 레이어가 1-dimensional convolution이고 dilation을 적용해서 RNN과 유사하게 동작  
- Convolution으로 이루어진 Sequence-to-Sequence로 생각하면 이해가 쉬움  

  
## Tacotron2  
  
<img src="https://user-images.githubusercontent.com/42150335/94840259-1cfbe900-0453-11eb-8803-cac2ea30b425.png" width=470>  
  
* Mel-Prediction Network : Attention based Sequence-to-Sequence Network  
  + 인코더에서 Bi-directional LSTM 적용
  + Location Sensitive Attention 적용 (음성 Alignment에 강한 어텐션)
  + 인코더, 디코더에 Convolution Layer 적용  
* Stop Token 사용
* Vocoder : WaveNet  
  + 장점 : 상당히 고품질의 음성으로 변환
  + 단점 : 엄청나게 느림
  

## Model Architecture
  
<img src="https://github.com/Tomiinek/Multilingual_Text_to_Speech/raw/master/_img/generated.png" width=800>
  
* Tacotron2 기반의 모델들로 실험 진행  
* WaveRNN Vocoder 사용

### This Paper`s Model: Generated (GEN)  
  
* **Parameter Generation Convolutional Encoder**   
  + DC-TTS 스타일 인코더와 Tacotron2 스타일 디코더를 사용  
  + Cross-lingual knowledge-sharing을 가능하게 하기 위해 인코더 컨볼루션 레이어의 파라미터를 생성하여 사용  
  + 입력되는 Language ID에 따라 Fully Connected 레이어를 통해 다른 다른 파라미터를 생성  
  
* **Speaker Embedding**  
  + Multi-speaker, Cross-lingual voice cloning을 위해 Speaker Embedding을 사용  
  + 인코더 아웃풋에 Concatenate하여 스펙트로그램 생성시에 반영되도록 함

* **Adversarial Speaker Classifier**  
  + 이상적으로 Voice cloning을 위해서는 텍스트(언어)로부터 화자의 정보가 반영되면 안됨  
  + Speaker Classifier와 나머지 모델(인코더, 디코더)은 forward에서는 독립적이지만,  backpropagation을 진행할 때, 두 loss (L2 of predict spectrogram, cross entropy of predicted speaker ID)가 인코더 파라미터 업데이트에 영향을 미침  
  + Gradient reversal layer를 통해 인코더가 speaker에 대한 정보를 반영 못하도록 학습

### Baselines: Shared, Separate & Single    
    
※ GEN과 다른점만 비교  
  
* **Single (SGL)** 
  + Monolingual Vanilla Tacotron 2 (Code-switching에 사용 X)
* **Shared (SHA)** 
  + GEN과 다르게 Tacotron 2의 인코더 사용 (Multilingual)  
* **Separate (SEP)** 
  + Multiple convolution layer를 사용
  + Parameter generation 사용 X
  + Adversarial speaker classifier 사용 X


## Dataset
  
10개의 언어로 구성된 CSS10과 Common Voice 데이터셋의 일부를 사용
Code-switching을 학습하기 위해 multi-speaker 데이터가 필요 (언어와 화자 일치를 없애기 위해)   
  
<img src="https://user-images.githubusercontent.com/42150335/95888064-9680c900-0dbb-11eb-9967-a30b21dbfa80.png" width=600>  

## Experiment  
  
SGL, SHA, SEP, GEN을 비교했을 때 GEN이 거의 모든 결과에서 우수한 성능을 보임
  
![image](https://user-images.githubusercontent.com/42150335/95888889-a816a080-0dbc-11eb-81a9-9a2d036f2def.png)  
  
<img src="https://user-images.githubusercontent.com/42150335/95888982-cbd9e680-0dbc-11eb-984f-1524ab3a9f38.png" width=400>
  
## Conclusion
  
* 하나의 모델로 10개 언어의 TTS를 커버함  
* Cross-lingual voice style transfer 지원   
  

