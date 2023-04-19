---
title: "Create a PDF to MP3 Converter with Python"
author: "Daniel K Baissa"
date: "2023-04-13"
categories: ["Python"]
tags: ["text-to-speech", "gtts", "pdfplumber", "tkinter", "pdf-to-mp3"]
highlight:
  - python
---




As a PhD student, I need to read a lot of academic papers, but I don't always have time to sit down and read them. To make the most of my time, I decided to learn how to code in Python and create a program that could convert journal articles and other PDFs into short audio books. In this tutorial, I'll show you how to create a simple PDF to MP3 converter using Python, the gTTS library (Google Text-to-Speech), and pdfplumber for extracting text from PDF files. We'll also create a graphical user interface (GUI) using tkinter to make the program more user-friendly.
Prerequisites

Before we get started, you'll need to have Python installed on your computer. You'll also need to install the following Python packages using `pip`:

    gtts: Google Text-to-Speech library for converting text to speech.
    pdfplumber: Extracting text from PDF files.
    tkinter: Creating a graphical user interface (GUI) for the converter.

You can install these packages using the following command:

```python
pip install gtts pdfplumber
```

Note that `tkinter` is included in most Python installations by default. If it is not available, refer to the official Python documentation for installation instructions.

## Extracting Text from a PDF

Our PDF to MP3 converter has two main processes: extracting text from the PDF and converting that text to speech using `gTTS`. Here's the code for the first process:

```python
import pdfplumber
from gtts import gTTS

def extract_text_from_pdf(pdf_path):
    with pdfplumber.open(pdf_path) as pdf:
        text = ""
        for page in pdf.pages:
            text += page.extract_text()

    return text

def text_to_speech(text, output_path, language='en'):
    tts = gTTS(text=text, lang=language)
    tts.save(output_path)

def main():
    pdf_path = 'path/to/your/pdf/file.pdf'  # Replace with your PDF file path
    output_path = 'output.mp3'  # Replace with your desired output path

    text = extract_text_from_pdf(pdf_path)
    text_to_speech(text, output_path)

    print(f"MP3 file saved at {output_path}")

if __name__ == "__main__":
    main()

```

The extract_text_from_pdf function uses pdfplumber to extract the text from each page of the PDF file, and then concatenates the text into a single string. We then use `gTTS` to convert the text to an MP3 file using the text_to_speech function.

Note that the main function is just an example of how to use these functions. We'll replace it with a more user-friendly GUI in the next section.
## Creating a GUI

To make our PDF to MP3 converter more user-friendly, we'll create a graphical user interface (GUI) using `tkinter`. Here's the updated code:


```python
import os
import tkinter as tk
from tkinter import filedialog
import pdfplumber
from gtts import gTTS

def extract_text_from_pdf(pdf_path):
    with pdfplumber.open(pdf_path) as pdf:
        text = ""
        for page in pdf.pages:
            text += page.extract_text()

    return text

def text_to_speech(text, output_path, language='en'):
    tts = gTTS(text=text, lang=language)
    tts.save(output_path)

def browse_pdf():
    file_path = filedialog.askopenfilename(filetypes=[("PDF files", "*.pdf")])
    if file_path:
        pdf_path_var.set(file_path)

def convert_to_mp3():
    pdf_path = pdf_path_var.get()
    if not pdf_path:
        return

    output_path = filedialog.asksaveasfilename(defaultextension=".mp3", filetypes=[("MP3 files", "*.mp3")])
    if not output_path:
        return

    text = extract_text_from_pdf(pdf_path)
    text_to_speech(text, output_path)

    result_var.set(f"MP3 file saved at {output_path}")

app = tk.Tk()
app.title("PDF to MP3 Converter")

pdf_path_var = tk.StringVar()
result_var = tk.StringVar()

frame = tk.Frame(app, padx=10, pady=10)
frame.pack()

pdf_label = tk.Label(frame, text="PDF File:")
pdf_label.grid(row=0, column=0, sticky="e")

pdf_entry = tk.Entry(frame, width=40, textvariable=pdf_path_var)
pdf_entry.grid(row=0, column=1)

pdf_button = tk.Button(frame, text="Browse", command=browse_pdf)
pdf_button.grid(row=0, column=2)

convert_button = tk.Button(frame, text="Convert to MP3", command=convert_to_mp3)
convert_button.grid(row=1, column=1, pady=10)

result_label = tk.Label(frame, textvariable=result_var)
result_label.grid(row=2, column=0, columnspan=3)

app.mainloop()

```


This is just a basic example of how you can create a GUI for the text-to-speech app using Tkinter. You can further customize and improve the interface as needed.

Now this is a fun toy, but like I said, I am a PhD student which means I will encounter lots of text with tables and figures. Even if the program can read the tables, which is questionable, it will likely sound obsurd and unhelpful to listen to the program read them. So I want it to write this to remove the tables and figures and say "see pdf for tables and figures"

To achieve this, we can use the pdfplumber library to find images and tables within the PDF document, and then replace their text with "See PDF for tables and figures." To do this, we'll modify the extract_text_from_pdf function:

```python
import os
import tkinter as tk
from tkinter import filedialog, ttk
import pdfplumber
from gtts import gTTS
import threading

def extract_text_from_pdf(pdf_path):
    with pdfplumber.open(pdf_path) as pdf:
        text = ""
        for page in pdf.pages:
            # Extract text and ignore text within tables and figures
            current_text = page.extract_text() or ""
            tables = page.extract_tables()
            figures = page.images

            if tables or figures:
                current_text += " See PDF for tables and figures."

            text += current_text

    return text

def text_to_speech(text, output_path, language='en'):
    tts = gTTS(text=text, lang=language)
    tts.save(output_path)

def browse_pdf():
    file_path = filedialog.askopenfilename(filetypes=[("PDF files", "*.pdf")])
    if file_path:
        pdf_path_var.set(file_path)

def convert_to_mp3():
    pdf_path = pdf_path_var.get()
    if not pdf_path:
        return

    output_path = filedialog.asksaveasfilename(defaultextension=".mp3", filetypes=[("MP3 files", "*.mp3")])
    if not output_path:
        return

    progress_bar["maximum"] = 100
    progress_bar["value"] = 0
    result_var.set("Converting...")

    def conversion_thread():
        text = extract_text_from_pdf(pdf_path)
        text_to_speech(text, output_path)

        progress_bar["value"] = 100
        result_var.set(f"MP3 file saved at {output_path}")

    threading.Thread(target=conversion_thread, daemon=True).start()

app = tk.Tk()
app.title("PDF to MP3 Converter")

pdf_path_var = tk.StringVar()
result_var = tk.StringVar()

frame = tk.Frame(app, padx=10, pady=10)
frame.pack()

pdf_label = tk.Label(frame, text="PDF File:")
pdf_label.grid(row=0, column=0, sticky="e")

pdf_entry = tk.Entry(frame, width=40, textvariable=pdf_path_var)
pdf_entry.grid(row=0, column=1)

pdf_button = tk.Button(frame, text="Browse", command=browse_pdf)
pdf_button.grid(row=0, column=2)

convert_button = tk.Button(frame, text="Convert to MP3", command=convert_to_mp3)
convert_button.grid(row=1, column=1, pady=10)

progress_bar = ttk.Progressbar(frame, length=300)
progress_bar.grid(row=2, column=0, columnspan=3, pady=10)

result_label = tk.Label(frame, textvariable=result_var)
result_label.grid(row=3, column=0, columnspan=3)

app.mainloop()

```


Now let's add code that tells us have the gui say "working" while its processing and "complete"  when its finsihed and  says "Conversion Failed" if there is an error?

To do this, we'll use a try-except block within the conversion_thread function. Here's the updated code:

```python
import os
import tkinter as tk
from tkinter import filedialog, ttk
import pdfplumber
from gtts import gTTS
import threading

def extract_text_from_pdf(pdf_path, progress_callback=None):
    with pdfplumber.open(pdf_path) as pdf:
        text = ""
        total_pages = len(pdf.pages)
        for i, page in enumerate(pdf.pages):
            current_text = page.extract_text() or ""
            tables = page.extract_tables()
            figures = page.images

            if tables or figures:
                current_text += " See PDF for tables and figures."

            text += current_text

            if progress_callback:
                progress_callback(i+1, total_pages)

    return text

def text_to_speech(text, output_path, language='en'):
    tts = gTTS(text=text, lang=language)
    tts.save(output_path)

def browse_pdf():
    file_path = filedialog.askopenfilename(filetypes=[("PDF files", "*.pdf")])
    if file_path:
        pdf_path_var.set(file_path)

def convert_to_mp3():
    pdf_path = pdf_path_var.get()
    if not pdf_path:
        return

    output_path = filedialog.asksaveasfilename(defaultextension=".mp3", filetypes=[("MP3 files", "*.mp3")])
    if not output_path:
        return

    result_var.set("Working...")

    def conversion_thread():
        try:
            text = extract_text_from_pdf(pdf_path)
            text_to_speech(text, output_path)
            result_var.set("Complete")
        except Exception as e:
            result_var.set("Conversion Failed")

    threading.Thread(target=conversion_thread, daemon=True).start()

app = tk.Tk()
app.title("PDF to MP3 Converter")

pdf_path_var = tk.StringVar()
result_var = tk.StringVar()

frame = tk.Frame(app, padx=10, pady=10)
frame.pack()

pdf_label = tk.Label(frame, text="PDF File:")
pdf_label.grid(row=0, column=0, sticky="e")

pdf_entry = tk.Entry(frame, width=40, textvariable=pdf_path_var)
pdf_entry.grid(row=0, column=1)

pdf_button = tk.Button(frame, text="Browse", command=browse_pdf)
pdf_button.grid(row=0, column=2)

convert_button = tk.Button(frame, text="Convert to MP3", command=convert_to_mp3)
convert_button.grid(row=1, column=1, pady=10)

result_label = tk.Label(frame, textvariable=result_var)
result_label.grid(row=2, column=0, columnspan=3)

app.mainloop()

```


The conversion_thread function now includes a try-except block to catch any exceptions that may occur during the conversion process. If an exception is caught, the result_var is updated with "Conversion Failed."

Now this is great, but I don't want to have to do one PDF at a time. Again, I have a lot of reading to do... So let's make it convert every PDF in a folder.

```python
import os
import tkinter as tk
from tkinter import filedialog, ttk
import pdfplumber
from gtts import gTTS
import threading
import glob

def extract_text_from_pdf(pdf_path, progress_callback=None):
    with pdfplumber.open(pdf_path) as pdf:
        text = ""
        total_pages = len(pdf.pages)
        for i, page in enumerate(pdf.pages):
            current_text = page.extract_text() or ""
            tables = page.extract_tables()
            figures = page.images

            if tables or figures:
                current_text += " See PDF for tables and figures."

            text += current_text

            if progress_callback:
                progress_callback(i+1, total_pages)

    return text

def text_to_speech(text, output_path, language='en'):
    tts = gTTS(text=text, lang=language)
    tts.save(output_path)

def browse_pdf():
    file_path = filedialog.askopenfilename(filetypes=[("PDF files", "*.pdf")])
    if file_path:
        pdf_path_var.set(file_path)

def browse_folder():
    folder_path = filedialog.askdirectory()
    if folder_path:
        folder_path_var.set(folder_path)

def convert_to_mp3():
    pdf_path = pdf_path_var.get()
    if not pdf_path:
        return

    output_path = filedialog.asksaveasfilename(defaultextension=".mp3", filetypes=[("MP3 files", "*.mp3")])
    if not output_path:
        return

    result_var.set("Working...")

    def conversion_thread():
        try:
            text = extract_text_from_pdf(pdf_path)
            text_to_speech(text, output_path)
            result_var.set("Complete")
        except Exception as e:
            result_var.set("Conversion Failed")

    threading.Thread(target=conversion_thread, daemon=True).start()

def convert_folder_to_mp3s():
    folder_path = folder_path_var.get()
    if not folder_path:
        return

    result_var.set("Working...")

    def conversion_thread():
        try:
            pdf_files = glob.glob(os.path.join(folder_path, '*.pdf'))
            for pdf_path in pdf_files:
                output_path = os.path.splitext(pdf_path)[0] + '.mp3'
                text = extract_text_from_pdf(pdf_path)
                text_to_speech(text, output_path)
            result_var.set("Complete")
        except Exception as e:
            result_var.set("Conversion Failed")

    threading.Thread(target=conversion_thread, daemon=True).start()

app = tk.Tk()
app.title("PDF to MP3 Converter")

pdf_path_var = tk.StringVar()
folder_path_var = tk.StringVar()
result_var = tk.StringVar()

frame = tk.Frame(app, padx=10, pady=10)
frame.pack()

pdf_label = tk.Label(frame, text="PDF File:")
pdf_label.grid(row=0, column=0, sticky="e")

pdf_entry = tk.Entry(frame, width=40, textvariable=pdf_path_var)
pdf_entry.grid(row=0, column=1)

pdf_button = tk.Button(frame, text="Browse", command=browse_pdf)
pdf_button.grid(row=0, column=2)

folder_label = tk.Label(frame, text="Folder:")
folder_label.grid(row=1, column=0, sticky="e")

folder_entry = tk.Entry(frame, width=40, textvariable=folder_path_var)
folder_entry.grid(row=1, column=1)

folder_button = tk.Button(frame, text="Browse", command=browse_folder)
folder_button.grid(row=1, column=2)

convert_button = tk.Button(frame, text="Convert PDF to MP3", command=convert_to_mp3)
convert_button.grid(row=2, column=1, pady=10)

convert_folder_button = tk.Button(frame, text="Convert Folder to MP3s", command=convert_folder_to_mp3s)
convert_folder_button.grid(row=3, column=1, pady=10)

result_label = tk.Label(frame, textvariable=result_var)
result_label.grid(row=4, column=0, columnspan=3)

app.mainloop()


```

Now you can select a folder containing PDF files using the "Browse" button next to the folder path entry. Clicking on the "Convert Folder to MP3s" button will convert all PDF files in the folder to MP3 files with the same name and in the same location as the original PDF files.

I hope you found this tutorial helpful in creating your own PDF to MP3 converter using Python. Feel free to modify and improve the code to suit your needs.
