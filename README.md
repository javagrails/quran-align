quran-align
===========

A tool for producing word-precise segmentation of recorded Qur'anic recitation. Designed to work with [EveryAyah](http://everyayah.com) style audio input.

Data
----

If you just want the data, you may not need to actually use this tool. I've generated timing files for many qura'a already - visit the Releases tab to download them.

These data files are licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/). Please consider emailing me if you use this data, so I can let you know when new & revised timing data is available.

### Data Format

Data files are JSON, of the following format:

    [
        {
            "surah": 1,
            "ayah": 1,
            "segments": {
                [word_start_index, word_end_index, start_msec, end_msec],
                ...
            },
            "stats": {
              "insertions": 123,
              "deletions": 456,
              "transpositions": 789
            }
        },
        ...
    ]

Where...

* `word_start_index` is the 0-base index of the first word contained in the segment.
* `word_end_index` is the 0-base index of the word _after_ the last word contained in the segment.
* `start_msec`/`end_msec` are timestamps within the input audio file.
* `stats` contain statistics from the matching routine that aligns the recognized words with reference text.

Here, a "word" is defined by splitting the text of the Qur'an  by spaces (specifically, `quran-uthmani.txt` from [Tanzil.net](http://tanzil.net/download) - without me_quran tanween differentiation!). Note that the language model used for recognition treats muqata'at as sets of words (ا ل م instead of الم) - but they will appear as a single word in the alignment output.

### Data Quality

Between the subjective nature of deciding exactly one word ends and the next begins, ambiguity surrounding repeated phrases, and most significantly, the lack of human-reviewed reference data, it is hard to measure the accuracy of this system. However, I was able to compare these results with those from the creators of [ElMohafez](http://elmohafez.com/), who use a different, independently-developed methodology than my own.

Using this data as a reference, I found that word timestamps fell an average of <73 msec away fro the reference data on a per-span basis, with standard deviations averaging 139 msec across all 6 recordings. 98.5-99.9% of words were individually segmented. These results except certain cases, most significantly, where the qari repeated or skipped a phrase (generally <1% of all words).

As our two independent implementations produce very similar results, it's reasonable to conclude that the data is largely correct, or that both implementations made the same mistakes.

### Data Completeness

In some cases, it was not possible to automatically differentiate two words. This is a rare occurrence. In all cases, the segment's start and end word indices indicate the range of words contained by the segment. It is possible that some words may be omitted from the result sequence if their bounds could not be determined directly or inferred from adjacent words.

Methodology
-----------

1. A [CMU Sphinx](http://cmusphinx.sourceforge.net/) speaker-specific acoustic model is trained using the verse-by-verse recitation recording and a Qur'an-specific language model.
1. PocketSphinx full-utterance recognition is run on each ayah's audio, provided with a filtered LM dictionary containing only words from that ayah to improve recognition rates and runtime performance.
1. Recognized words are matched to the reference text of each ayah, accounting for insertions, deletions, transpositions, etc.
1. Raw audio data and a derived [MFCC](https://en.wikipedia.org/wiki/Mel-frequency_cepstrum) feature stream are used to refine alignment of words to syllable boundaries within the ayah audio.

Usage
-----

Unfortunately, a key component - the script that generates the speech model training inputs and supporting data files - is currently in [an unpublishable state](https://media.tenor.co/images/3d6ef5c0cacab962cd9db2e309114a7e/raw). Nonetheless, with this excercise left to the reader, the `align` tool's help output explains its full usage. You may need to override `CMUSPHINX_ROOT` in the Makefile. Note that WAV files must be generated by FFMPEG because I hard-coded an offset to the audio data to avoid writing a RIFF parser.

### Requirements

 - A UNIX machine (Windows Bash/LWS works)
 - PocketSphinx, SphinxTrain, and cmuclmtk from [CMU Sphinx](http://cmusphinx.sourceforge.net/)
 - [EveryAyah](http://everyayah.com)-style audio recordings of the recitation
 - A C++11-compatible compiler
 - Python
