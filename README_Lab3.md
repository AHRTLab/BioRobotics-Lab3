# Lab 3: Galvanic Skin Response and the Autonomic Nervous System

**BioRobotics**
**Duration:** In-lab data collection (~1.5 hours) + take-home analysis
**Group Size:** 2-3 students

---

## Abstract

This lab introduces students to electrodermal activity (EDA), also known as Galvanic Skin Response (GSR), using the BioRadio 150. Students configure the BioRadio for GSR mode, run a guided stress/relaxation protocol, and perform a Stroop color-word test to explore how cognitive stress affects autonomic nervous system activation. The lab emphasizes experimental design — students adjust Stroop parameters and develop hypotheses before collecting data.

**Please read through the entire document before beginning the lab.**

---

## Learning Objectives

By the end of this lab, students will be able to:

1. **Understand the physiological basis of EDA** — how sympathetic activation controls sweat gland activity and changes skin conductance
2. **Configure the BioRadio for GSR recording** — set GSR mode, DC coupling, appropriate sample rate
3. **Design and execute a stress/relaxation experiment** — guided protocol with baseline, breathing, arithmetic, and recovery phases
4. **Decompose GSR into tonic and phasic components** — separate Skin Conductance Level (SCL) from Skin Conductance Responses (SCR)
5. **Detect and measure SCR peaks** — amplitude, latency, rise time, frequency
6. **Perform statistical analysis** — compare autonomic responses across conditions using appropriate tests
7. **Develop and test experimental hypotheses** — predict how Stroop task parameters affect GSR

---

## Background

### The Autonomic Nervous System

The autonomic nervous system (ANS) regulates involuntary bodily functions including heart rate, digestion, and sweating. It has two branches:

- **Sympathetic nervous system (SNS)**: The "fight or flight" response. Increases heart rate, dilates pupils, and activates sweat glands.
- **Parasympathetic nervous system (PNS)**: The "rest and digest" response. Slows heart rate and promotes relaxation.

Eccrine sweat glands, particularly on the palms and fingers, are primarily controlled by the SNS. When sympathetic activity increases (due to stress, arousal, or cognitive effort), these glands produce more sweat, which increases the electrical conductance of the skin.

### Electrodermal Activity (EDA) / Galvanic Skin Response (GSR)

EDA measures the electrical conductance of the skin, which changes with sweat gland activity. It has two components:

- **Tonic component (SCL)**: Skin Conductance Level — the slow baseline that changes over minutes. Reflects overall arousal state.
- **Phasic component (SCR)**: Skin Conductance Responses — rapid transient peaks lasting 1-5 seconds. Reflect specific autonomic events or stimuli.

### GSR Signal Characteristics

| Property | Value |
|----------|-------|
| Frequency range | DC to ~5 Hz |
| Typical amplitude | 1-20 microsiemens |
| SCR onset latency | 1-5 seconds after stimulus |
| SCR rise time | 1-3 seconds |
| SCR recovery time | 3-10 seconds |
| Electrode placement | Fingers (palmar surface) |

### Key Difference from EMG

In Lab 1, EMG signals were high-frequency oscillatory signals (20-450 Hz) requiring bandpass filtering, rectification, and envelope extraction. GSR is the opposite — a slowly varying DC signal below 5 Hz. The processing pipeline is completely different: only low-pass filtering and tonic/phasic decomposition are needed.

---

## Files Overview

| File | Description |
|------|-------------|
| `src/gsr_processing.py` | GSR signal processing (low-pass filter, tonic/phasic decomposition, SCR peak detection) |
| `src/gsr_collect.py` | Guided data collection script for Part A protocol |
| `src/stroop_test.py` | PyQt6 GUI Stroop color-word test for Part B |
| `src/bioradio.py` | BioRadio interface (supports GSR mode) |
| `src/bioradio_example.py` | BioRadio examples (Example 7 = GSR config) |
| `src/visualizer.py` | Real-time visualization (optional, for monitoring) |
| `notebooks/Lab3_GSR_Analysis_Student_Copy.ipynb` | Analysis notebook |
| `environment.yml` | Conda environment specification |

---

## Part 1: Environment Setup

### 1.1 Activate the Conda Environment

```bash
conda activate biorobotics
```

If the environment doesn't exist yet, create it:

```bash
conda env create -f environment.yml
conda activate biorobotics
```

### 1.2 Verify Installation

```bash
python -c "import neurokit2; import pyserial; from PyQt6.QtWidgets import QApplication; print('All packages installed!')"
```

---

## Part 2: BioRadio Setup for GSR

### 2.1 Hardware Setup

**Electrode placement:**

1. Clean the palmar surface of the **index and middle fingers** on the **non-dominant hand** 
2. Let the skin dry completely (30 seconds)
3. Apply a small amount of electrode gel if using dry electrodes
4. Attach the finger electrodes snugly — firm contact but not uncomfortable
5. Rest the hand comfortably on the table — **do not move this hand during recording**

**Important:**
- No hand lotion before recording (affects conductance)
- Keep the electrode hand still throughout the experiment
- Room temperature affects baseline GSR — note if the room is unusually hot/cold

### 2.2 Connect and Configure the BioRadio

**Option A: Using the data collection script (recommended)**

```bash
python src/gsr_collect.py --port COM9 --participant YOUR_ID
```

This automatically configures the BioRadio for GSR mode.

**Option B: Using the example script**

```bash
python src/bioradio_example.py --example 7 --port COM9
```

This configures GSR mode and acquires a short test recording.

**Option C: Manual configuration (advanced)**

```python
from src.bioradio import BioRadio, BioPotentialMode, CouplingType

radio = BioRadio(port="COM9")
radio.connect()
config = radio.get_configuration()

# Configure channel 1 for GSR
ch = config.biopotential_channels[0]
ch.operation_mode = BioPotentialMode.GSR
ch.coupling = CouplingType.DC
ch.bit_resolution = 16
ch.enabled = True
ch.name = "GSR"
radio.set_channel_config(ch)
radio.set_sample_rate(250)
```

### 2.3 Verify the Signal

Before starting the experiment, verify you have a good GSR signal:

1. Run a short test acquisition (10-20 seconds)
2. The signal should be **stable** (not jumping wildly)
3. Take a deep breath — you should see a small change in conductance
4. If the signal is flat (zero) or extremely noisy, check electrode contact

---

## Part 3: Data Collection — Part A (Guided Protocol)

### 3.1 Protocol Overview

| Phase | Duration | Activity | Expected GSR Effect |
|-------|----------|----------|-------------------|
| **Baseline** | 2 min | Quiet rest, breathe normally | Low, stable SCL |
| **Deep Breathing** | 2 min | Paced: 5s inhale, 5s exhale | Decreased SCL (parasympathetic) |
| **Mental Arithmetic** | 2 min | Count back from 1000 by 7, aloud | Increased SCL + frequent SCRs |
| **Recovery** | 2 min | Quiet rest | Gradual return toward baseline |

### 3.2 Running the Protocol

```bash
python src/gsr_collect.py --port COM9 --participant YOUR_ID
```

The script will:
1. Connect to the BioRadio and configure GSR mode
2. Guide you through each phase with timing prompts
3. Provide breathing cues during the deep breathing phase
4. Save all data to a CSV file with condition markers

**Important instructions for the participant:**
- Keep the electrode hand completely still
- During baseline and recovery: sit quietly, relax
- During deep breathing: follow the on-screen inhale/exhale prompts
- During mental arithmetic: say each number **aloud** as fast as you can

### 3.3 Data Collection Checklist (Part A)

| Phase | Completed | Notes |
|-------|-----------|-------|
| Baseline (2 min) | ☐ | |
| Deep Breathing (2 min) | ☐ | |
| Mental Arithmetic (2 min) | ☐ | |
| Recovery (2 min) | ☐ | |

---

## Part 4: Data Collection — Part B (Stroop Test)

### 4.1 The Stroop Effect

The Stroop effect is one of the most well-known phenomena in cognitive psychology. When the **ink color** of a word conflicts with the word itself (e.g., the word "RED" printed in blue ink), naming the ink color takes longer and produces more errors.

This cognitive interference is a mild stressor that activates the sympathetic nervous system, potentially producing measurable GSR responses.

### 4.2 Write Your Hypothesis FIRST

**Before running the Stroop test**, discuss with your group and write down:

1. **Primary hypothesis**: How will incongruent vs. congruent trials differ in reaction time?
2. **GSR hypothesis**: Will you see different GSR responses for incongruent vs. congruent trials?
3. **Parameter hypothesis**: Pick ONE parameter to vary (stimulus time, ITI, congruent ratio). Predict how changing it will affect both RT and GSR.

Write these in your notebook before proceeding.

### 4.3 Running the Stroop Test

```bash
python src/stroop_test.py
```

A settings dialog will appear. Configure:

| Parameter | Description | Default | Range |
|-----------|-------------|---------|-------|
| **Number of Trials** | Total trials in the test | 30 | 10-100 |
| **Congruent Ratio** | Percentage of congruent trials | 50% | 0-100% |
| **Stimulus Time** | How long the word is displayed | 3.0s | 0.5-5.0s |
| **Inter-Trial Interval** | Pause between trials | 2.0s | 0.5-4.0s |

**During the test:**
- A color word appears in colored ink on a dark background
- Press the key matching the **INK COLOR** (not the word):
  - **R** = Red, **B** = Blue, **G** = Green, **Y** = Yellow
- Respond as quickly and accurately as possible
- Results are saved automatically to a CSV file

### 4.4 Run the Stroop Test While Recording GSR

For the richest dataset, run the Stroop test while simultaneously recording GSR. You can use the visualizer or a separate BioRadio acquisition script running alongside the Stroop test. Note the Stroop test start time to synchronize timestamps.

```bash
python src/bioradio.py --lsl
```
---

## Part 5: Take-Home Analysis

Open the analysis notebook:

```bash
conda activate biorobotics
jupyter lab notebooks/Lab3_GSR_Analysis_Student_Copy.ipynb
```

The notebook guides you through:

1. **Loading and exploring** your GSR recording
2. **Signal processing** — low-pass filtering, tonic/phasic decomposition
3. **Part A analysis** — SCL comparison across conditions, SCR rate, statistical tests
4. **Part B analysis** — Stroop behavioral results, RT distributions, GSR correlation
5. **Discussion** — comparative analysis, confounds, applications

---

## Deliverables

Submit the following:

### 1. Data Files (ZIP)
- Part A: GSR recording CSV (from `gsr_collect.py`)
- Part B: Stroop results CSV (from `stroop_test.py`)
- Any additional GSR recordings from simultaneous Stroop + GSR

### 2. Analysis Notebook
- Completed `Lab3_GSR_Analysis_Student_Copy.ipynb` with all cells executed
- All TODO sections implemented
- All 10 questions answered

### 3. Hypothesis Document
- Written hypotheses from Part 4.2 (before running the Stroop test)
- Brief reflection on whether results matched predictions

---

## Troubleshooting

### BioRadio GSR Issues

| Problem | Solution |
|---------|----------|
| Signal is zero/flat | Check electrode contact; ensure electrodes are on palmar surface |
| Signal is extremely noisy | Ensure hand is still; check for loose electrode connections |
| Signal saturates (goes to max) | Too much electrode gel; wipe and reapply |
| Signal drifts continuously | Normal for first 1-2 minutes; wait for stabilization |
| Cannot configure GSR mode | Ensure BioRadio firmware supports GSR; try power cycling the device |

### Stroop Test Issues

| Problem | Solution |
|---------|----------|
| Window doesn't appear | Ensure PyQt6 is installed: `pip install pyqt6` |
| Keys don't register | Click on the test window first to ensure it has focus |
| Colors look wrong | Try windowed mode: `python src/stroop_test.py --windowed` |

### General Tips

- **Electrode stabilization**: Wait 2-3 minutes after attaching electrodes before recording
- **Baseline**: Always record a quiet baseline first for comparison
- **Movement**: Any movement of the electrode hand will create artifacts
- **Temperature**: Cold hands = low baseline GSR; warm hands = higher baseline
- **Time of day**: GSR can vary with circadian rhythms

---

## Additional Resources

- **NeuroKit2 Documentation**: https://neuropsychology.github.io/NeuroKit/
- **Boucsein, W. (2012)**: *Electrodermal Activity* (2nd ed.) — comprehensive EDA reference
- **Stroop, J.R. (1935)**: "Studies of interference in serial verbal reactions" — the original paper
- **Lab Streaming Layer (LSL)**: https://labstreaminglayer.org/

---

*Last updated: February 2026*
