# 😀 MY MEMOJI
# | Introduction <br>
**My Memoji**는 **유저 사진을 input으로 그와 닮은 memoji를 생성해주는 서비스**입니다. <br>
SDXL을 finetuning하고, 독립적인 ldm을 사용하여 post-hoc img2img 성능을 개선하였습니다. 더하여, React를 사용해 custom ui를 빌드하였습니다.<br>
📄 [Visit our notion for more detail!](https://sparkly-onion-be7.notion.site/My-Memoji-16f8e2ec5d7a81cea547f92474f83cd0?pvs=4)
<br><br>
**🔥 Motivation 🔥**<br>
미모지는 사용자가 자신을 닮은 맞춤형 캐릭터를 만들 수 있도록 도와주는 Apple의 앱입니다.
하지만, 미모지를 만들 때 지정할 수 있는 스타일에 한계가 있기에 나와 닮게 만들기 힘들다는 문제점이 있습니다.<br>
저희는 이러한 문제점을 바탕으로, 사진을 찍으면 나와 닮은 미모지를 자동으로 생성해주는 프로젝트를 진행해보기로 하였습니다.


# | Installation <br>
## SD WebUI
**MyMemoji**의 UI는 automatic1111을 활용하여 작성되었습니다. 따라서, MyMemoji webUI를 실행하기 전에, automatic1111에서 적절한 환경설정이 필요합니다 <br>

**1. Clone the repo**<br>
```
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
```
<br>

**2. Download models**<br>
➡️ checkpoint for ControlNet : [diffusers_xl_canny_full.safetensors](https://huggingface.co/lllyasviel/sd_control_collection/blob/main/diffusers_xl_canny_full.safetensors)<br>

➡️ [Download our model checkpoints here](https://drive.google.com/drive/folders/10c-bRDNM-EAHATRaCPTQ3ZGfspJUgASs)<br>
- checkpoint for finetuned Samaritan SDXL : `./Lora/memoji-07.safetensors`<br>
- checkpoint for additional ldm : `./Stable-diffusion/samaritan3dCartoon_v40SDXL.safetensors`<br><br>


**3. Set up files**<br>
다운로드한 모델 파일 위치를 다음과 같이 조정합니다.

```
└──...── extentions/
│        └── sd-webui-controlnet/
│           └── models/
│              └── diffusers_xl_canny_full.safetensors
│                            
└──...── models/           
│        └── Stable-diffusion/        
|            └── samaritan3dCartoon_v40SDXL.safetensors
│        └── Lora/        
|            └── memoji-07.safetensors
|
```
<br>

**4. set up api**<br>
`webui-user.bat`코드를 다음과 같이 수정합니다.
```
@echo off

set PYTHON=
set GIT=
set VENV_DIR=
set COMMANDLINE_ARGS=--api

call webui.bat
```
<br>

**5. run SD WebUI**<br>
```
./webui.sh --api
```

## MyMemoji WebUI 
**1. Clone the repo**<br>
```
git clone https://github.com/alswn-03/emoji-generation-webui
```
<br>

**2. run MyMemoji WebUI**<br>
```
npm start
```
<br>
      
### 3. 웹사이트 사용 방법
   <img src='./assets/emoji-generation-webui.png' width=800><br><br>
   이모지 생성을 원하는 사진을 넣습니다 <br> 
   -> 성별과 얼굴 표정 선택합니다 <br> 
   -> **Generate** 를 누릅니다. 
   <br><br>
    
# | Model
<img src='./assets/architecture.jpg' width=800><br>
➡️ [Download model checkpoint here](https://drive.google.com/drive/folders/10c-bRDNM-EAHATRaCPTQ3ZGfspJUgASs)<br>
- checkpoint for finetuned Samaritan SDXL : `./Lora/memoji-07.safetensors`<br>
- checkpoint for additional ldm : `./Stable-diffusion/samaritan3dCartoon_v40SDXL.safetensors`<br><br>

### 1. base model : <br>
- [Samaritan-3d-Cartoon-SDXL](https://huggingface.co/imagepipeline/Samaritan-3d-Cartoon-SDXL)
- 최대한 프로젝트 목적에 맞는, cartoon 화풍을 가진 모델로 선정함 <br>
<img src='./assets/vanilla samaritan.jpg' width=500><br><br>
  
### 2. 원하는 그림체 만들기 (LoRA finetuning)<br>
- vanilla Samaritan SDXL은 cartoon 화풍을 보여주긴 했지만, 우리가 원하는 emoji 화풍과는 거리가 있었음
- 총 2748개의 preprocessed data를 사용해 LoRA finetuning 진행<br>
<img src='./assets/lora finetuning.jpg' width=500><br>
- 🚨 원하는 스타일이 적용되긴 했지만, 원본 이미지 속 인물과 표정과 자세가 일치하지 않는다는 문제점 발견<br><br>


### 3. 원본 이미지에 충실하기 (ControlNet w/ Canny Edge)<br>
- 2에서 생긴 문제점을 해결하기 위해서 입력 이미지의  edge 정보를 활용하기로 결정함
- Controlnet w/ canny edge를 활용하여 입력 이미지의 edge 정보를 반영한 생성<br>
<img src='./assets/controlnet.jpg' width=500><br>
- 🚨 원본 이미지 속 인물의 윤곽을 잘못 인식하여 생성 이미지에 unwanted artifacts가 나타남
    - 위 예시의 경우, 사진 속 인물의 쌍꺼풀과 애굣살까지 눈으로 인식해버림<br><br>


### 4. 더 부드러운 결과물 얻기<br>
- 3에서 생긴 문제점을 해결하기 위해서, 독립적인 ldm을 추가로 사용함
- 추가 ldm의 역할 :  
    - 3에서의 artifact를 덮어버림  
    - upscaling을 통한 이미지 해상도 향상  
- empirical한 이유로, 추가 ldm은 vanilla 모델을 사용함<br>
<img src='./assets/additional ldm.jpg' width=500><br>
- ✅ 원본 이미지의 윤곽을 반영하면서, 원하는 화풍이 적용된 이미지 생성 성공 !  <br><br>







