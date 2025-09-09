# Install necessary libraries
%pip install PyMuPDF gtts pydub

# Import libraries
import fitz  # PyMuPDF
from google.colab import files
from gtts import gTTS
from pydub import AudioSegment
import os
import time # Import the time library
import re

# Upload the PDF file
print("👉 Please upload your PDF file")
uploaded = files.upload()

# Get the filename
pdf_filename = list(uploaded.keys())[0]

# Extract text from the PDF
text = ""
try:
    with fitz.open(pdf_filename) as doc:
        for page in doc:
            text += page.get_text()
except Exception as e:
    print(f"Error extracting text from PDF: {e}")
    text = "" # Ensure text is empty if extraction fails

print("✅ Extracted characters:", len(text))

# Check if text was extracted
if text:
    # Split text into chunks (TTS limit handling)
    def split_text(text, max_chars=1500):
        parts, cur = [], ""
        for s in re.split(r'(\.|\n)', text):
            if len(cur) + len(s) > max_chars and cur:
                parts.append(cur.strip())
                cur = s
            else:
                cur += s
        if cur.strip():
            parts.append(cur.strip())
        return parts

    parts = split_text(text, 1500)

    # Convert text to speech
    print("Converting text to speech...")
    temp_files = []
    for i, part in enumerate(parts):
        print(f"Converting part {i+1}/{len(parts)}...")
        try:
            tts = gTTS(text=part, lang='en')
            fname = f"_part_{i}.mp3"
            tts.save(fname)
            temp_files.append(fname)
            time.sleep(3) # Add a delay between requests (increased from 2)
        except Exception as e:
            print(f"Error converting part {i}: {e}")


    # Merge parts into one audiobook
    audiobook = AudioSegment.empty()
    for f in temp_files:
        try:
            audiobook += AudioSegment.from_mp3(f)
        except Exception as e:
            print(f"Error merging file {f}: {e}")

    output_audio_filename = os.path.splitext(pdf_filename)[0] + ".mp3"
    audiobook.export(output_audio_filename, format="mp3")
    print(f"Audiobook saved as {output_audio_filename}")

    # Download result
    files.download(output_audio_filename)

    # (Optional) Play the audio
    # from IPython.display import Audio
    # Audio(output_audio_filename)

else:
    print("No text extracted from the PDF. Cannot create audiobook.")
