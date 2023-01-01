---
title: "Python으로 PDF 오디오북 만들기"
tags:
- python
---
- `pyttsx3` : TTS(text-to-speech) 용 (version: `2.90`)
- `PyPDF2` : PDF 파일을 다루기 위한 라이브러리 (version : `3.0.0`)

## Code 
```python 
import sys
import pyttsx3
import PyPDF2

# python main.py ./test.pdf 인자로 읽을 파일 받기
file_path = sys.argv[1]

if len(sys.argv) != 2:
	print("Insufficient arguments")
	sys.exit()

# binary mode로 파일 읽기
pdf_file = open(file_path, 'rb')

# PdfFileReader로 파일 읽을 Reader Object 생성
pdfReader = PyPDF2.PdfReader(pdf_file)

# 읽은 총 페이지 수 
pages = len(pdfReader.pages)

# pyttsx3.Engine 초기화 해주기
speaker = pyttsx3.init()

# 페이지 돌며 text 읽어오기 
for num in range(pages):
	# 해당 페이지 읽기
	page = pdfReader.pages[num] # pdfReader.getPage(num)
	# 해당 페이지 내 text 뽑아내기 
	text = page.extract_text() 
	# 뽑아낸 text 읽기 
	speaker.say(text)
	speaker.runAndWait() 

# mp3로 저장하려면 아래와 같이 하면 된다.
speaker.save_to_file(text, 'audio.mp3')
speaker.runAndWait()
```

## GitHub Code (with Poetry)
- https://github.com/jiyeonseo/pdf-to-tts-mp3-python 

## References
- [Make Audio Book from any PDF using Python | Python Project](https://morioh.com/p/42a6957afa8a?f=5c21fb01c16e2556b555ab32)
- [How to save pyttsx3 results to MP3 or WAV file?](https://www.geeksforgeeks.org/how-to-save-pyttsx3-results-to-mp3-or-wav-file/)
- [Change pyttsx3 language](https://stackoverflow.com/questions/65977155/change-pyttsx3-language)