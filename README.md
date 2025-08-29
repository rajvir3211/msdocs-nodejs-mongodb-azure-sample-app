---
dtmf_app/
│── dtmf_app.py
│── requirements.txt


---

🐍 dtmf_app.py

import numpy as np
import sounddevice as sd
import tkinter as tk
from scipy.fft import fft

# Sampling info
fs = 8000
duration = 0.5

# DTMF frequency map
dtmf_freqs = {
    '1': (697, 1209),
    '2': (697, 1336),
    '3': (697, 1477),
    '4': (770, 1209),
    '5': (770, 1336),
    '6': (770, 1477),
    '7': (852, 1209),
    '8': (852, 1336),
    '9': (852, 1477),
    '*': (941, 1209),
    '0': (941, 1336),
    '#': (941, 1477)
}

# Function to generate & play tone
def play_tone(key):
    f1, f2 = dtmf_freqs[key]
    t = np.linspace(0, duration, int(fs * duration), endpoint=False)
    tone = np.sin(2*np.pi*f1*t) + np.sin(2*np.pi*f2*t)
    sd.play(tone, fs)
    sd.wait()

# Function to detect DTMF from microphone input
def detect_tone():
    duration_rec = 0.5
    recording = sd.rec(int(duration_rec * fs), samplerate=fs, channels=1, dtype='float64')
    sd.wait()

    # FFT analysis
    signal = recording[:,0]
    fft_data = np.abs(fft(signal))
    freqs = np.fft.fftfreq(len(fft_data), 1/fs)

    positive_freqs = freqs[:len(freqs)//2]
    positive_fft = fft_data[:len(freqs)//2]

    # Find two main peaks
    peak_indices = positive_fft.argsort()[-5:]
    detected = sorted([round(positive_freqs[i]) for i in peak_indices])

    # Match with DTMF table
    detected_key = None
    for key, (f1,f2) in dtmf_freqs.items():
        if any(abs(d-f1)<20 for d in detected) and any(abs(d-f2)<20 for d in detected):
            detected_key = key
            break

    if detected_key:
        result_label.config(text=f"Detected: {detected_key}")
    else:
        result_label.config(text="No valid tone detected")

# GUI
root = tk.Tk()
root.title("DTMF Generator + Decoder")

# Generator buttons
buttons = [
    ['1','2','3'],
    ['4','5','6'],
    ['7','8','9'],
    ['*','0','#']
]

for r,row in enumerate(buttons):
    for c,key in enumerate(row):
        btn = tk.Button(root, text=key, font=("Arial", 20), width=5, height=2,
                        command=lambda k=key: play_tone(k))
        btn.grid(row=r, column=c, padx=5, pady=5)

# Decoder button
detect_btn = tk.Button(root, text="🎤 Detect Tone", font=("Arial", 16),
                       command=detect_tone, bg="lightblue")
detect_btn.grid(row=5, column=0, columnspan=3, pady=10)

# Result label
result_label = tk.Label(root, text="Detected: None", font=("Arial", 16))
result_label.grid(row=6, column=0, columnspan=3, pady=5)

root.mainloop()

page_type: sample
languages:
- azdeveloper
- javascript
- bicep
products:
- azure
- azure-app-service
- azure-cosmos-db
- azure-virtual-network
- azure-key-vault
- azure-cache-redis
urlFragment: msdocs-nodejs-mongodb-azure-sample-app
name: Deploy a Express.js web app with MongoDB in Azure
description: This is a CRUD web app that uses Express.js and Azure Cosmos DB.
---
<!-- YAML front-matter schema: https://review.learn.microsoft.com/en-us/help/contribute/samples/process/onboarding?branch=main#supported-metadata-fields-for-readmemd -->

# Deploy a Express.js web app with MongoDB in Azure

This is a CRUD (create-read-update-delete) web app that uses Express.js and Azure Cosmos DB. The Node.js app is hosted in a fully managed Azure App Service. This app is designed to be be run locally Linux Node.js container in Azure App Service. You can either deploy this project by following the tutorial [*Tutorial: Deploy a Node.js + MongoDB web app to Azure*](https://learn.microsoft.com/azure/app-service/tutorial-nodejs-mongodb-app) or by using the [Azure Developer CLI (azd)](https://learn.microsoft.com/azure/developer/azure-developer-cli/overview) according to the instructions below.

## Run the sample

This project has a [dev container configuration](.devcontainer/), which makes it easier to develop apps locally, deploy them to Azure, and monitor them. The easiest way to run this sample application is inside a GitHub codespace. Follow these steps:

1. Fork this repository to your account. For instructions, see [Fork a repo](https://docs.github.com/get-started/quickstart/fork-a-repo).

1. From the repository root of your fork, select **Code** > **Codespaces** > **+**.

1. In the codespace terminal, run the following command:

    ```shell
    npm install && npm start
    ```

1. When you see the message `Your application running on port 3000 is available.`, click **Open in Browser**.

### Quick deploy

This project is designed to work well with the [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/overview), which makes it easier to develop apps locally, deploy them to Azure, and monitor them. 

🎥 Watch a deployment of the code in [this screencast](https://www.youtube.com/watch?v=JDlZ4TgPKYc).

Steps for deployment:

1. Sign up for a [free Azure account](https://azure.microsoft.com/free/)
2. Install the [Azure Dev CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd). (If you opened this repository in a Dev Container, it's already installed for you.)
3. Log in to Azure.

    ```shell
    azd auth login
    ```

4. Provision and deploy all the resources:

    ```shell
    azd up
    ```

    It will prompt you to create a deployment environment name, pick a subscription, and provide a location (like `westeurope`). Then it will provision the resources in your account and deploy the latest code. If you get an error with deployment, changing the location (like to "centralus") can help, as there may be availability constraints for some of the resources.

5. When `azd` has finished deploying, you'll see an endpoint URI in the command output. Visit that URI, and you should see the CRUD app! 🎉 If you see an error, open the Azure Portal from the URL in the command output, navigate to the App Service, select Logstream, and check the logs for any errors.

6. When you've made any changes to the app code, you can just run:

    ```shell
    azd deploy
    ```

## Getting help

If you're working with this project and running into issues, please post in [Issues](/issues).
