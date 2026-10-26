# GPT-SoVITS Batch Voice Generation

A small local workflow for turning an Excel list of dialogue into individually named WAV files using GPT-SoVITS. Each row supplies the file name, dialogue and optional output subfolder.

This repository contains a sanitised version of the production script and an empty spreadsheet template. It contains no company dialogue, production file names, reference audio or model weights. Model training is outside this tool’s scope.

## Files

- `batch_tts.py` — reads the spreadsheet, calls a local GPT-SoVITS API and saves the audio.
- `voice_lines_template.xlsx` — a blank workbook with the required headers.
- `requirements.txt` — dependencies for the batch script.

## 1. Prepare GPT-SoVITS

Install [GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS) separately and prepare compatible GPT and SoVITS character weights, a reference audio clip and its exact transcript. Confirm that the character works in GPT-SoVITS before running a full batch.

The spreadsheet uses a language-neutral `Dialogue` header. Set `TEXT_LANG` to the language you want to generate and `PROMPT_LANG` to the language spoken in your reference clip. Both default to `ja` (Japanese); change them for other languages using codes supported by your installed GPT-SoVITS API. The two settings describe different inputs and do not have to be the same. The spreadsheet header does not select the language.

## 2. Install the script dependencies

Download this repository and open a terminal in its folder. Use the same Python interpreter for installation and running the script:

```sh
python -m pip install -r requirements.txt
```

For a Windows integrated GPT-SoVITS package, use its bundled interpreter instead, replacing the example installation path as needed:

```bat
D:\GPT-SoVITS\runtime\python.exe -m pip install -r requirements.txt
```

GPT-SoVITS itself has separate installation requirements.

## 3. Edit the configuration

Open `batch_tts.py` in a text editor. Replace these values at the top:

| Setting | What to enter |
| --- | --- |
| `SOVITS_PTH` | Full local path to the character’s SoVITS `.pth` file. |
| `GPT_CKPT` | Full local path to the character’s GPT `.ckpt` file. |
| `REF_WAV` | Full local path to the reference audio clip. |
| `REF_TEXT` | The exact words spoken in that reference clip. |
| `TEXT_LANG` | Language code for the dialogue to generate. |
| `PROMPT_LANG` | Language code for the reference clip and transcript. |

Replace all `TODO` values before running. When switching characters, update both model paths and the reference clip/transcript together.

`XLSX_PATH` defaults to `voice_lines_template.xlsx` beside the script. `OUT_DIR` defaults to an `output` folder beside the script. Change these only if you want different locations. `API_BASE` defaults to `http://127.0.0.1:9880`; it must match your local service. Keep `CANDIDATES = 1` for one file per line.

## 4. Fill in Excel locally

Keep the headers exactly as supplied and fill the active worksheet:

| Header | Input |
| --- | --- |
| `Voice name` | The output file name without `.wav`. |
| `Dialogue` | The dialogue to generate. |
| `Output folder` | Optional relative subfolder inside `output`, such as `interaction`. Leave blank to save directly in `output`. |

Copy the file names and dialogue together from your source list so each row remains correctly paired. A row without a voice name or dialogue is skipped.

Use valid file names and simple relative subfolder names. Each **output folder + voice name** combination should be unique. Identical names can be used in separate folders; duplicate names within the same folder share one target file and later entries will be skipped.

Save and close Excel before running the batch. Add real dialogue locally after downloading the template. Keep populated spreadsheets and reference transcripts out of public commits and external AI conversations; this repository’s `.gitignore` does not prevent committing changes to the tracked template or script.

## 5. Start the local service

From the GPT-SoVITS installation folder, start its API v2 service using that installation’s Python environment:

```sh
python api_v2.py -a 127.0.0.1 -p 9880
```

With the Windows integrated package:

```bat
runtime\python.exe api_v2.py -a 127.0.0.1 -p 9880
```

Keep this terminal open. If your version uses different startup options, follow its API documentation. This script expects `/set_gpt_weights`, `/set_sovits_weights` and `/tts`. Run one character batch at a time because model switching changes the service’s active weights.

## 6. Generate the batch

Before **every batch**, confirm the local paths and language settings at the top of the script:

- Check both character model paths, the reference audio path, spreadsheet path and output path.
- Set `TEXT_LANG` to the language of the dialogue you are generating.
- Set `PROMPT_LANG` to the language spoken in the reference audio, and make sure `REF_TEXT` matches that clip.

Language settings must be checked just like local paths, especially when switching characters or localisation languages. They are not detected automatically from the spreadsheet, and both default to `ja` until you change them.

Open a second terminal in this repository’s folder:

```sh
python batch_tts.py
```

Or use the Windows bundled interpreter:

```bat
D:\GPT-SoVITS\runtime\python.exe batch_tts.py
```

The script selects the configured weights, reads the spreadsheet and saves each generated file as:

```text
output/<Output folder>/<Voice name>.wav
```

It prints successful output paths, API failures and a final summary. The skipped counter counts empty-name/dialogue rows; it does not count existing files. Try a few lines first to check the character, names and folders before filling the full list.

## 7. Check and regenerate

Sample files from the beginning, middle and end of a batch, checking voice quality and the correspondence between dialogue and file name. For game work, also check the voice lines in their actual in-game context.

Existing files are skipped when you run the script again. To regenerate specific lines, delete only their output WAV files and rerun the script; keep the corresponding spreadsheet rows filled in. Editing a row without deleting its existing output does not regenerate that file.

## Local processing and limitations

With `API_BASE` pointing to localhost, generation requests go to your local GPT-SoVITS service. The batch script does not call an external AI assistant. Terminal error responses can contain sensitive details, so review logs before sharing them.

This is a small production utility, not a full job-management system. File presence is used to skip completed work; it does not prove that a WAV is complete or correct. Connection errors can stop a run, and files left behind by an interrupted write should be checked and removed before retrying. Confirm both model weights loaded successfully in the startup output, especially when switching characters.

The repository has been checked for Python syntax and workbook structure. End-to-end generation requires your local GPT-SoVITS installation, models and reference audio.
