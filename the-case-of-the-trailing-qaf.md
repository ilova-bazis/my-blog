---
title: "The Case of the Trailing Қ"
date: "2026-10-07"
excerpt: "Fine-tuning a bilingual handwriting recognizer on a small English and Tajik dataset improved the overall score — while teaching the model to end English sentences with a Tajik **қ**."
tags: ["handwriting-recognition", "machine-learning", "fine-tuning", "tajik", "multilingual", "character-error-rate", "ocr", "quirks"]
---

I’m building a local handwriting recognition tool for scanned notes in English and Tajik. The workflow is straightforward: crop a line, straighten it, transcribe it, and use those corrections to fine-tune a model.

Then my model developed a peculiar habit.

It started giving English sentences a Tajik farewell:

> to the progress of our knowledge theқ

That final **қ** is a perfectly legitimate Tajik Cyrillic letter. It just has no business standing at the end of that English sentence.

## Was it the UI?

My first question was whether the interface was accidentally appending it. A keyboard issue? A string concatenation bug?

The saved prediction files answered that: the character appeared before predictions were integrated into the UI.

On our 29 English test lines:

| Model and input | Lines ending in қ |
|---|---:|
| Pretrained model, original scans | 0 |
| Pretrained model, isolated ink | 0 |
| Fine-tuned model, original scans | **15** |
| Fine-tuned model, isolated ink | 0 |

The annotation data wasn’t responsible either. No English training transcript contained **қ**, and no training transcript ended with it.

The model had learned this little signature all by itself.

## A bilingual model with an uneven education

We had fine-tuned a multilingual handwriting recognizer on a small dataset:

- **129 Tajik training lines**
- **24 English training lines**

The pretrained model lacked the Tajik-specific letters we needed, so fine-tuning expanded its output alphabet.

That worked: the model began producing Tajik characters. Tajik character error rate on the original test crops dropped from **23.5% to 17.5%**.

But the English output acquired an unexpected souvenir.

The imbalance is a plausible contributor, though it doesn’t fully explain why **қ** appears specifically at line endings—and only with original scans in this test. That pattern suggests the paper background or crop-edge appearance may also be involved. We haven’t confirmed the exact trigger yet.

## Better overall doesn’t mean better everywhere

Overall character error rate improved from **19.2% to 16.6%**. A single summary score would make the experiment look like a straightforward success.

Looking at the languages separately told a more interesting story. Tajik improved substantially, while English word error rate worsened.

A model can improve on average and still develop a very visible bad habit.

## What comes next?

We’ll investigate the end-of-line predictions, balance the training exposure, and collect more English pages. Since we already label each line’s language, we can also test language-aware decoding.

Simply removing every trailing **қ** would hide the symptom. I’d rather understand why it appears—and make sure fixing it doesn’t just replace it with a different wrong character.

For now, it’s a useful reminder that fine-tuning isn’t a magic “make it better” button.

Sometimes it teaches your model another language.

Sometimes it teaches it to end English sentences with **қ**.
