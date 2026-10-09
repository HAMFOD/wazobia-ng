# WAZOBIA: a smart glove that turns sign language into speech in Yorùbá, Hausa and Igbo

WAZOBIA is a wearable glove that recognises a set of signed phrases and a browser dashboard that turns each phrase into spoken Yorùbá, Hausa or Igbo. Translation is done by **N-ATLaS**, the multilingual Nigerian language model from NCAIR and Awarri.

Built for the National AI Innovation Challenge 2026.

**Live dashboard:** https://wazobia-ng-vz23.vercel.app (open it in Google Chrome)

## How it works

1. The glove (an ESP32-S3 with flex sensors that monitor the bending of each finger as different gestures are made, plus a motion sensor) recognises a held hand pose and sends it over Bluetooth Low Energy to the dashboard.
2. The dashboard (a single web page, `index.html`) shows the recognised English phrase.
3. For Yorùbá, Hausa or Igbo, the dashboard asks an **N-ATLaS** server to translate the phrase, then speaks it with the recorded voice clip in the `audio/` folder for that language and phrase. If a clip is missing, the dashboard shows a "Recording missing" note and stays silent (the browser voice is off by default).
4. If the server does not reply, the dashboard shows a built-in reference dictionary instead, so the glove keeps working offline.
5. Every gesture can be judged by an observer (sign correct? translation correct?) and exported as a CSV for validation.

The architecture, setup steps and usage are described in [docs/WAZOBIA_Technical_Documentation.pdf](https://wazobia-ng-vz23.vercel.app/doc/WAZOBIA_NG_Technical_Documentation.pdf).

## Platform Compatibility

**Current implementation:** WAZOBIA.NG operates through a PC-based web dashboard using Google Chrome. The ESP32-S3 smart glove connects to the dashboard via Bluetooth Low Energy (BLE), enabling real-time gesture recognition and translation into text and speech.

**Current limitation:** Direct smartphone and feature-phone compatibility have not yet been implemented or validated. The current validated setup requires a PC running Google Chrome.

**Proposed development:** Future versions may explore mobile-accessible interfaces and alternative communication methods to extend WAZOBIA.NG beyond its current PC-based environment. Feature-phone compatibility would require additional implementation and testing; it is not a capability of the existing prototype.

## How N-ATLaS is used

- Model: [`NCAIR1/N-ATLaS`](https://huggingface.co/NCAIR1/N-ATLaS) (Llama-3 8B based, gated on Hugging Face).
- It is served from a GPU notebook in 4-bit precision behind a small FastAPI server that accepts OpenAI-style chat requests. The validation sessions on 2 and 3 October used the Kaggle notebook ([`server/natlas_server_kaggle.ipynb`](server/natlas_server_kaggle.ipynb)). The first test session used the earlier Colab version ([`server/natlas_server_colab.ipynb`](server/natlas_server_colab.ipynb)).
- The dashboard sends a short instruction ("reply with only the translation") plus up to three example pairs, and uses the model's answer as the translation shown and spoken to the user.
- Names (for example "Hameed") are not translated. Answers are cached for the page session. The sentence composer joins gestures signed in a row into one sentence and sends the whole sentence to N-ATLaS.

## Repository contents

| Path | What it is |
|---|---|
| `index.html` | The WAZOBIA dashboard (open it in Google Chrome) |
| `audio/` | Recorded voice clips played by the dashboard (`en_1.mp3` to `ig_10.mp3`) |
| `server/natlas_server_kaggle.ipynb` | Kaggle notebook that loads N-ATLaS and exposes it as an API (used for the validation sessions) |
| `server/natlas_server_colab.ipynb` | Earlier Colab version of the same notebook (used for the first test session) |
| `docs/WAZOBIA_Technical_Documentation.pdf` | Technical documentation: architecture, setup and usage |

## Quick start

> **Use Google Chrome.** The dashboard needs Web Bluetooth, which has only been confirmed working in Chrome.

**1. Start the N-ATLaS server (Kaggle)**
1. In [Kaggle](https://www.kaggle.com) (an account with a verified phone number), create a notebook and import `server/natlas_server_kaggle.ipynb` (File > Import Notebook).
2. In Settings choose Accelerator **GPU T4 x2** and turn **Internet** on. Under Add-ons > Secrets, add `HF_TOKEN` (a Hugging Face token, `hf_...`, from an account that has been granted access to `NCAIR1/N-ATLaS`) and attach it to the notebook.
3. Run the cells in order. At the end, the notebook prints a **public URL** and an **API key**. Keep the session open, and stop it when you finish so the weekly GPU quota is not used while idle.

The earlier Colab version (`server/natlas_server_colab.ipynb`) works the same way: choose Runtime > Change runtime type > **T4 GPU** and paste the token when asked. The free Colab tier timed out during testing, which is why the validation sessions used Kaggle.

**2. Open the dashboard**
1. Open the live dashboard at https://wazobia-ng-vz23.vercel.app in **Google Chrome** (Web Bluetooth is required), or open `index.html` from this repository.
2. In the "N-ATLaS connection" panel, paste the URL and API key and press **Save & test**.
3. Press connect, choose the `WAZOBIA_Glove` device, pick a language, and sign.

Without a server the dashboard still runs, using the reference dictionary below. No glove? Open the **Simulator** panel and press a phrase button to try the dashboard without Bluetooth.

**Browser requirement: use Google Chrome.** The dashboard connects to the glove with Web Bluetooth, and it has been tested and confirmed working in Chrome. Web Bluetooth is not available in Safari or Firefox, or on iPhone and iPad, and other browsers have not been tested. The page must be served over HTTPS (such as Vercel) or opened from localhost. The glove accepts one connection at a time, and the connection list shows only devices named `WAZOBIA_Glove`.

## Gesture set

| Channel | Phrase | Yorùbá (reference) | Hausa (reference) | Igbo (reference) |
|---|---|---|---|---|
| 1 | Good Morning | Ẹ kú àárọ̀ | Ina kwana | Ụtụtụ ọma |
| 2 | My name is | Orukọ mi ni | Sunana | Aha m bụ |
| 3 | I am a student | Akẹkọọ ni mi | Ni ɗalibi ne | Abụ m nwa akwụkwọ |
| 4 | Hameed | Hameed | Hameed | Hameed |
| 5 | This is my work | Iṣẹ mi ni eyi | Wannan shine aiki na | Nke a bụ ọrụ m |
| 6 | FUTminna | FUTminna | FUTminna | FUTminna |
| 7 | Please | Ẹ jọwọ | Don Allah | Biko |
| 8 | How are you | Bawo ni? | Yaya kake? | Kedu ka ị mere? |
| 9 | I need help | Mo nilo iranlọwọ | Ina buƙatar taimako | Achọrọ m enyemaka |
| 10 | Stop | Duro | Tsaya | Kwụsị |

The reference dictionary is the offline fallback and the source of the example pairs given to the model.

## Validation results

Results from the real-user sessions on 2 and 3 October 2026: 120 judged gestures from 10 participants (Yorùbá 39, Hausa 38, Igbo 43). Participants appear only as IDs. The full data and report are part of the challenge submission (Real-World Validation).

| Measure | Result |
|---|---|
| Sign recognised correctly (glove) | 114 / 120 = 95.0% |
| Translation judged correct, all languages* | 92 / 109 = 84.4% (77.1% as originally rated) |
| Hausa* | 30 / 33 (22 / 33 as originally rated) |
| Yorùbá | 33 / 38 |
| Igbo | 29 / 38 |

\*Excludes the 11 name gestures, which are passed through unchanged. Eight Hausa "Good Morning" ratings were changed from No to Yes after review ("Barka da safiya" is a valid Hausa greeting); the original rating and a note are kept in the validation data.

## Limitations

- Translations were rated by one native speaker for each language (Yorùbá, Hausa and Igbo), not by an independent panel of linguists, and the Yorùbá rater is also a member of the project team. Some outputs were judged incorrect (for example Yorùbá "Good Morning").
- 106 of the 109 translation ratings reused an answer cached earlier in the same session from a live N-ATLaS call, so the ratings cover roughly 23 distinct language-and-phrase translations. Cached rows show 0 ms latency; live calls took about 2 seconds each.
- The gesture set is 10 phrases, and some phrases were signed far more often than others.
- The sentence composer speaks a sentence by playing the recorded clip for each gesture in order, so the spoken words can differ slightly from the sentence N-ATLaS wrote on screen.
- Gestures are recognised from the sensor readings (finger bending and hand orientation) with fixed thresholds (no per-wearer calibration). The glove's "confidence" is a margin score between 60 and 99, not a calibrated probability, so it is not a reliable indicator of a wrong sign.
- The server runs on a free GPU notebook, which stops when the session ends and gets a new address each time.

## Hardware

The glove is built around an ESP32-S3. **Flex sensors** monitor the bending of each finger as different gestures are made, an MPU-6050 motion sensor measures the hand's orientation, and a DFPlayer Mini with speaker plays audio on the glove. A gesture is recognised once the hand has been held still for about 0.8 seconds.

## Team

A two-person team from the Department of Mechatronics Engineering, Federal University of Technology, Minna, Niger State, Nigeria.

| Name | Role | Responsibilities |
|---|---|---|
| Hameed Fodlullahi Abidemi | Team Lead | System architecture, embedded implementation, TinyML, N-ATLAS integration and overall coordination |
| Ikhaghu Hassan | Team Member | Software and dashboard development, validation and user testing, documentation and presentation |

Engr. Justice Chikezie Anunuso served as project supervisor and technical adviser for the original WAZOBIA project. He is not a member of the two-person challenge team.

## Attribution

Translation is performed by **N-ATLaS**, developed by Awarri Technologies and the National Centre for Artificial Intelligence and Robotics (NCAIR) under the Nigerian Languages AI Initiative. Please check the model's licence on its Hugging Face page for attribution and usage terms.
