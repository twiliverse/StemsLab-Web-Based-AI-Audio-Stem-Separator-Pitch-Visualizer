# 🎵 StemsLab — AI Audio Stem Separator & Real-Time Pitch Visualizer

> An end-to-end, web-based audio separation and vocal analysis platform. StemsLab isolates multitrack stems (Vocals, Drums, Bass, Accompaniment) using deep neural networks and renders real-time pitch accuracy tracking via the Web Audio API for interactive karaoke and fan cover experiences.

![License](https://img.shields.io/badge/license-MIT-purple)
![Python](https://img.shields.io/badge/Python-3.11+-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.110+-009688?logo=fastapi&logoColor=white)
![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black)
![PyTorch](https://img.shields.io/badge/PyTorch-HT--Demucs-EE4C2C?logo=pytorch&logoColor=white)
![Web Audio API](https://img.shields.io/badge/Web_Audio_API-Worklet-ff69b4)

---

## 🚀 Executive Summary

Platforms like **Weverse** and modern fan portals thrive on User-Generated Content (UGC)—specifically karaoke covers, dance practices, and audio remixes. However, processing multitrack stems on cloud servers for millions of users creates massive infrastructure costs and high latency.

**StemsLab** bridges deep learning audio separation with client-side Web Audio API rendering:
1. **Server-Side AI Isolation:** Uses fine-tuned **HT-Demucs v4** (Hybrid Transformer Demucs) to isolate raw audio uploads into 4 distinct stems (`Vocals`, `Drums`, `Bass`, `Other Accompaniment`) or 2 stems (`Vocals` vs `Instrumental`).
2. **Client-Side Digital Signal Processing (DSP):** Loads isolated stems into a custom Web Audio API node graph. Users can mute, solo, or adjust stem gain with sub-millisecond synchronization without re-fetching data from the backend.
3. **Interactive Pitch Visualizer:** Extracts the target vocal pitch contour (`PyIN` / `CREPE`) and compares it in real-time against live user microphone input via an in-browser `AudioWorklet` running the YIN autocorrelation pitch algorithm.

---

## ✨ Key Features

### 🎙️ 1. High-Fidelity AI Stem Separation
* **Model Pipeline:** Powered by Meta AI's **HT-Demucs v4** neural network model with spectrogram & time-domain hybrid processing.
* **Lossless Export:** Generates high-bitrate WAV or FLAC stems with minimal cross-stem audio bleed.
* **Async Processing Queue:** Offloads heavy GPU tasks via FastAPI background tasks and Redis/Celery worker queues with WebSocket progress updates.

### 🎛️ 2. Web Audio API Multitrack Mixer
* **Sub-Millisecond Synchronization:** Renders separate `AudioBufferSourceNode` objects routed through dedicated `GainNode` and `PannerNode` chains.
* **Karaoke Mode:** One-click vocal mute with instant instrumental accompaniment playback.
* **Dynamic Waveform Visualizer:** Real-time HTML5 Canvas rendering powered by `AnalyserNode` frequency/time-domain data.

### 🎯 3. Live Pitch Accuracy Visualizer
* **Target Vocal Extraction:** Pre-calculates target pitch contours (F0 frequency in Hz mapped to MIDI semitones) during stem processing.
* **Mic Pitch Tracking:** Captures user voice via `getUserMedia()` and processes pitch in an `AudioWorkletThread` using the YIN pitch-detection algorithm.
* **Cents Variance & Accuracy Scoring:** Compares live microphone pitch against target vocal pitch in real time, displaying pitch drift in signed cents ($\pm 50\text{ cents}$) and giving an instant accuracy score.

---

## 🏗️ System Architecture

```text
                               ┌─────────────────────────────────────────────────────────────┐
                               │                    FRONTEND (React + Web Audio)             │
                               │                                                             │
                               │  [ User Upload MP3 ] ──> [ HTML5 Canvas Waveform Visualizer ]  │
                               │                                   │                         │
                               │  [ Live Mic Input ]  ──> [ Web Audio Worklet (YIN Pitch) ]    │
                               └─────────────────────────┬───────────────────────────────────┘
                                                         │
                                                  HTTP POST / WebSocket
                                                         │
                                                         ▼
┌────────────────────────────────────────────────────────────────────────────────────────────┐
│                                   BACKEND (FastAPI / Python)                               │
│                                                                                            │
│  [ API Router ] ──> [ Async Task Queue ] ──> [ HT-Demucs v4 Neural Model ]                 │
│                                                       │                                    │
│                                           ┌───────────┴───────────┐                        │
│                                           ▼                       ▼                        │
│                                   [ Stem Separation ]   [ Pitch Contour (PyIN) ]           │
│                                   (Vocals, Accompaniment)  (F0 Frequencies / MIDI)         │
└───────────────────────────────────────────┬───────────────────────┬────────────────────────┘
                                            │                       │
                                            └───────────┬───────────┘
                                                        │
                                                        ▼
                                       [ Client Receives Stems & JSON Pitch Map ]
