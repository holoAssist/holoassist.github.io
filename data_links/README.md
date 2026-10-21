# ICCV 2023 HoloAssist: an Egocentric Human Interaction Dataset for Interactive AI Assistants in the Real World

This guide describes how to download the HoloAssist dataset and interpret its files and annotations. See the [baseline code](https://github.com/Ember-HoloAssist/holoassist-release) for benchmark implementations.

[Project website](https://holoassist.github.io/) · [Paper](https://openaccess.thecvf.com/content/ICCV2023/html/Wang_HoloAssist_an_Egocentric_Human_Interaction_Dataset_for_Interactive_AI_Assistants_ICCV_2023_paper.html)


## Download the data and annotations

The dataset is released under the [Community Data License Agreement – Permissive, Version 2.0](https://cdla.dev/permissive-2-0/).

1. Open the [dataset downloads](https://holoassist.github.io/#download) and choose the components you need. The links download archives or annotation files directly.
2. Extract the `.tar` archives into a common dataset directory, preserving their recording-session paths.
3. Download the raw training and validation annotations, [`data-annotation-trainval-v1_1.json`](https://hl2data.z5.web.core.windows.net/holoassist-data-release/data-annotation-trainval-v1_1.json). This is a JSON file and does not need extraction.
4. Download and extract [`data-splits-v1_2.zip`](../label_files/data-splits-v1_2.zip). It contains `train-v1_2.txt`, `val-v1_2.txt`, and `test-v1_2.txt`, which identify the recording sessions in each split. The test split list is not a set of test annotations.

For example, after downloading `video_compress.tar`:

```sh
mkdir -p holoassist-data
tar -xf video_compress.tar -C holoassist-data
```

## Dataset Structure
After extracting the archives, each recording-session folder contains an `Export_py` directory with files for the downloaded modalities. The tree below illustrates the layout; only the components you download will be present. Files with `_sync` or `_synced` in their names contain data synchronized to the RGB stream. The sensors have different sampling rates, and the experiments use the synchronized data.

The dataset was collected using the [HoloLens capture tools in Microsoft Psi](https://github.com/microsoft/psi/tree/master/Sources/MixedReality/HoloLensCapture). See the [HoloLensCaptureExporter documentation](https://github.com/microsoft/psi/tree/master/Sources/MixedReality/HoloLensCapture/HoloLensCaptureExporter) for details about the exported data formats.


<pre>
.
├── R007-7July-DSLR
│   └── Export_py
│       ├── AhatDepth
│       │   ├── 000000.png
│       │   ├── 000001.png
│       │   ├── ...
│       │   ├── AhatDepth_synced.txt
│       │   ├── Intrinsics.txt
│       │   ├── Pose_sync.txt
│       │   └── Timing_sync.txt
│       ├── Eyes
│       │   └── Eyes_sync.txt
│       ├── Hands
│       │   ├── Left_sync.txt
│       │   └── Right_sync.txt
│       ├── Head
│       │   └── Head_sync.txt
│       ├── IMU
│       │   ├── Accelerometer_sync.txt
│       │   ├── Gyroscope_sync.txt
│       │   └── Magnetometer_sync.txt
│       ├── Video
│       │   ├── Pose_sync.txt
│       │   ├── Intrinsics.txt
│       │   └── VideoMp4Timing.txt
│       ├── Video_pitchshift.mp4
│       └── Video_compress.mp4
├── R012-7July-Nespresso/
├── R013-7July-Nespresso/
├── R014-7July-DSLR/
└── ...
</pre>



## Annotation Structure

The raw annotation file is a JSON array of recording-session objects. Each session has an `events` array containing individual annotations. The examples below show complete events from the published v1.1 file.

The baseline repository provides [processed labels and label mappings](https://github.com/Ember-HoloAssist/holoassist-release/tree/main/src/data_2221), including `labels_20240719_2221_classes.json` and `labels_20240719_2221_label2idx.json`. Its [`process_labels.py`](https://github.com/Ember-HoloAssist/holoassist-release/blob/main/src/data_2221/process_labels.py) script converts raw annotations into the baseline format.

Each event has these fields:

- `id`: integer identifying the original annotation.
- `label`: annotation category: `Narration`, `Conversation`, `Fine grained action`, or `Coarse grained action`.
- `start` and `end`: numeric timestamps in seconds relative to the video.
- `type`: `range` for a time interval.
- `attributes`: an object containing fields specific to the annotation category.

Attribute names and categorical values are case-sensitive. The lists below describe their meanings; use the strings in the released JSON when parsing annotations.

### Narration
```json
{
    "id": 1,
    "label": "Narration",
    "start": 0.002,
    "type": "range",
    "attributes": {
        "Long form description": "The student grabs the GoPro from the packaging and holds it in his hands with a little difficulty, grabs the camera and opens it, and just touches the battery, holds the GoPro in his hands for a few minutes, then removes the battery and the SD card, placing them on the table, inserts the battery and turns on the GoPro, and turns it off, take the lock screw and inserts it, removes it, places the stick, puts the lock screw, removes it and finally places it on the table."
    },
    "end": 406.135
}
```
`Long form description` summarizes the session, including mistakes and how they are corrected. Narration covers the overall session, but its timestamps need not be exactly zero or the video's reported duration. Use the recorded `start` and `end` values.

### Conversation
```json
{
    "id": 116,
    "label": "Conversation",
    "start": 7.451,
    "type": "range",
    "attributes": {
        "Conversation Purpose": "instructor-start-conversation_other",
        "sequence": "none",
        "Transcription": "*unintelligible*",
        "Transcription Confidence": "low-confidence-transcription",
        "Conversation Purpose -- Other": "none"
    },
    "end": 8.872
}
```

- **Conversation Purpose**: Select an option that best describes the purpose of the speech. This is limited to the individual speaking and does not include any pause time waiting for a response.
    - Instructor-start-conversation:  Describing high-level instruction
    - Instructor-start-conversation:   Opening remarks
    - Instructor-start-conversation:   Closing remarks
    - Instructor-start-conversation:  Adjusting to capture better quality video
    - Instructor-start-conversation:  Confirming the previous or future action
    - Instructor-start-conversation:   Correct the wrong action
    - Instructor-start-conversation:   Follow-up instruction
    - Instructor-start-conversation: Other
    - Instructor-reply-to-task performer: Confirming the previous or future action
    - Instructor-reply-to-task performer: Correct the wrong action
    - Instructor-reply-to-task performer: Follow-up instruction
    - Instructor-reply-to-task performer: other
    - task performer-start-conversation: ask questions
    - task performer-start-conversation: others

- **Transcription**: The spoken words transcribed as text.
- **Transcription Confidence**: The human annotator’s confidence in the transcription.

### Fine grained action
```json
{
    "id": 119,
    "label": "Fine grained action",
    "start": 11.01,
    "type": "range",
    "attributes": {
        "Action Correctness": "Correct Action",
        "Incorrect Action Explanation": "none",
        "Incorrect Action Corrected by": "none",
        "Verb": "approach",
        "Adjective": "none",
        "Noun": "gopro",
        "adverbial": "none"
    },
    "end": 12.21
}
```

- **Action Correctness**: Indicate whether the action is correct or a mistake to achieve the task. The options are
    - Correct action
    - Wrong action, corrected by instructor verbally
    - Wrong action, corrected by performer
    - Wrong action, not corrected
    - Others

- **Incorrect Action Explanation**: Provided by the human annotators to explain why they believe the action is wrong.
- **Incorrect Action Corrected by**: Indicate whether the wrong action is later corrected by the instructor or the task performer.
- **Verb**, **Adjective**, **Noun**, **adverbial**: Verb, (adjective), Noun, (adverbial) for describing the fine-grained actions.

### Coarse grained action
```json
{
    "id": 3,
    "label": "Coarse grained action",
    "start": 10.975,
    "type": "range",
    "attributes": {
        "Action sentence": "The student grabs the GoPro.",
        "Verb": "grab",
        "Adjective": "none",
        "Noun": "gopro"
    },
    "end": 28.21
}
```

- **Action sentence**: A factual statement describing the interaction that the collector/camera wearer is performing with a digital device and the software on the device.
    - Example 1: A man changes the battery of the bright green GoPro.
    - Example 2: A woman attaches the leg to a chair.
- **Verb**: The action verb associated with the sentence.
    - Example 1: Change
    - Example 2: Attach
- **Adjective**: This is the adjective(s) that helps distinguish the noun from other similar items. This field is optional if the noun is unique enough on its own.
    - Example 1: bright green
    - Example 2: [blank]
- **Noun**: This is the generic noun that is part of the Coarse-Grained Action sentence.
    - Example 1: GoPro
    - Example 2: Leg



**Citation**


If you find the code or data useful, please cite the paper:
```
@inproceedings{wang2023holoassist,
  title={Holoassist: an egocentric human interaction dataset for interactive ai assistants in the real world},
  author={Wang, Xin and Kwon, Taein and Rad, Mahdi and Pan, Bowen and Chakraborty, Ishani and Andrist, Sean and Bohus, Dan and Feniello, Ashley and Tekin, Bugra and Frujeri, Felipe Vieira and others},
  booktitle={Proceedings of the IEEE/CVF International Conference on Computer Vision},
  pages={20270--20281},
  year={2023}
}
```
