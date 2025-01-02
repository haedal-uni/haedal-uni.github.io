---
categories: Eng-Project
tags: [log]
---

# TTS(Text to Speech) 
영어 학습 시 예문을 음성으로 들을 수 있도록 TTS를 적용했다.   

이를 위해 무료로 사용할 수 있는 Web Speech API를 활용했다. 

<br><br>   

### 기본 사용 방법
TTS는 **SpeechSynthesisUtterance** 객체를 생성한 뒤 이를 `speechSynthesis.speak()`에 전달하여 음성을 재생한다.

```js
// 음성 재생
window.speechSynthesis.speak(new SpeechSynthesisUtterance("Breakaway"));

// 일시 중지
window.speechSynthesis.pause();

// 다시 재생
window.speechSynthesis.resume();

// 재생 중단
window.speechSynthesis.cancel();
```

<br><br><br>     

### `SpeechSynthesisUtterance`
**SpeechSynthesisUtterance**는 읽을 텍스트와 읽기 옵션(속도, 음높이, 음량 등)을 설정하는 객체다.

- 속도(rate): 0.1 ~ 10 사이 값 설정          
- 음높이(pitch): 0 ~ 2 사이 값 설정         
- 음량(volume): 0 ~ 1 사이 값 설정
  
```js
const utter = new SpeechSynthesisUtterance("Breakaway");

utter.rate = 1; // 말하는 속도
utter.pitch = 1; // 음높이
utter.volume = 1; // 음량

window.speechSynthesis.speak(utter);
```
           
<br><br><br>      

### 지원되는 음성 목록 가져오기
`speechSynthesis.getVoices()`를 사용하면 브라우저에서 지원하는 음성 목록을 가져올 수 있다.    

이를 활용하여 특정 음성을 선택하거나 사용자 UI를 구성할 수 있다.  
 
```js
const synth = window.speechSynthesis;

let voices = synth.getVoices(); // 지원되는 음성 목록 가져오기
voiceSelect.innerHTML = ""; // 이전 목록 초기화 

// 각 음성을 select 요소에 추가
voices.forEach((voice) => {   
    const option = document.createElement("option");
    option.textContent = `${voice.name} (${voice.lang})`; // 형식 설정
    option.setAttribute("voice-name", voice.name);
    voiceSelect.appendChild(option);    
});

const selectedVoice = voiceSelect.selectedOptions[0].getAttribute("voice-name");   

// voice, pitch, rate 설정
utterance.voice = voices.find((voice) => voice.name === selectedVoice); // 선택된 음성 적용
utterance.pitch = parseFloat(pitchInput.value); // pitch 설정
utterance.rate = parseFloat(rateInput.value); // rate 설정 

synth.speak(utterance); // 음성 재생
```
 
![image](https://github.com/user-attachments/assets/4a28c32b-d149-40ba-a1af-5fecc02a8398){: width="50%"}  

<br><br><br>      
 
### 영어 학습용 TTS    
영어 학습을 위해 `en-US`를 고정 설정하고 pitch와 rate 조정을 위한 옵션을 추가했다.  
```js
const synth = window.speechSynthesis;

let voices = [];
// 음성 목록 업데이트
function populateVoiceList() {
    voices = synth.getVoices(); // 지원되는 음성 목록 가져오기
}

populateVoiceList();
if (speechSynthesis.onvoiceschanged !== undefined) {
    speechSynthesis.onvoiceschanged = populateVoiceList;
}

// 실시간 pitch와 rate 값 표시  
pitchInput.addEventListener("input", () => {
    pitchValue.textContent = pitchInput.value;
});
rateInput.addEventListener("input", () => {
    rateValue.textContent = rateInput.value;
});

// 텍스트 읽기
function speakText() {
    const lang = "en-US"; // 언어 설정  
    const utterance = new SpeechSynthesisUtterance(sentence.textContent);


    utterance.lang = lang;
    // pitch와 rate 설정
    utterance.pitch = parseFloat(pitchInput.value);
    utterance.rate = parseFloat(rateInput.value);

    synth.speak(utterance); // 텍스트 읽기  
}
```

<br><br> 

tts를 활용해 단어를 학습할 때 자동으로 예문을 읽어주고 더 읽고 싶을 때 읽기 버튼을 클릭하게 했다.

옵션 설정 값은 아래와 같다.    

```html
<input type="range" id="tts-pitch" min="0" max="2" step="0.1" value="1" />
<!-- 0부터 2까지 조정 가능하며 사용자는 0.1 단위로 값을 선택할 수 있고 기본값은 1이다. -->

<input type="range" id="tts-rate" min="0.1" max="2" step="0.1" value="1" />
<!-- 0.1부터 2까지 조정 가능하며 사용자는 0.1 단위로 값을 선택할 수 있고 기본값은 1이다. -->
```
<br><br>

![image](https://github.com/user-attachments/assets/ba4197bf-5e5e-4140-89a4-ba4a7edc761b)


<br><br><br><br>   

**REFERENCE**          
- [Web Speech API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API)    
- [Web Speech API로 프론트엔드에서 TTS 구현하기](https://wormwlrm.github.io/2024/03/09/Web-Speech-API.html)    
