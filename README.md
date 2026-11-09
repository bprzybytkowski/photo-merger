# photo-merger

A small shell script for quickly preparing trading card photos for marketplace listings, such as Cardmarket. It takes a folder of photos showing the front and back of each card, pairs them up by capture time, and merges each pair into a single side-by-side image.

```
front.jpg  +  back.jpg  ->  merged/merged-01.jpg  (front on the left, back on the right)
```

## How it works

1. Reads the capture date of every JPG, JPEG, PNG, and HEIC file in the folder using `exiftool`. If a photo has no `DateTimeOriginal`, it falls back to `CreateDate`, and then to the file modification date.
2. Sorts the photos from oldest to newest.
3. Takes them two at a time: the older photo becomes the left half (front), the newer one the right half (back).
4. Merges each pair with ImageMagick, fixing the rotation based on EXIF orientation, and saves the result as `merged/merged-NN.jpg`.

Subfolders are not scanned, so earlier results in `merged/` are never picked up as input.

## Requirements

- macOS or Linux with `bash`
- [ExifTool](https://exiftool.org/)
- [ImageMagick 7](https://imagemagick.org/) (the `magick` command), built with HEIC support if you shoot in HEIC

On macOS with Homebrew:

```sh
brew install exiftool imagemagick
```

## Installation

Clone the repository and link the script into a folder on your `PATH`:

```sh
git clone https://github.com/bprzybytkowski/photo-merger.git ~/photo-merger
chmod +x ~/photo-merger/cardmerge
ln -s ~/photo-merger/cardmerge ~/.local/bin/cardmerge
```

If `~/.local/bin` is not on your `PATH` yet, add this line to `~/.zshrc` (or `~/.bashrc`) and open a new terminal:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

## Usage

```sh
cardmerge                  # process photos in the current folder
cardmerge ~/Pictures/cards # process photos in a given folder
```

Example output:

```
merged/merged-01.jpg  <-  PXL_20261107_111914745.jpg + PXL_20261107_111927868.jpg
done: 1 pair(s) in /Users/me/Pictures/cards/merged
```

## Shooting tips

The pairing relies entirely on the order the photos were taken in, so:

- Always photograph the front first, then the back, then move on to the next card.
- Delete any retakes or accidental shots before running the script. A single extra photo shifts every pair after it.
- If the number of photos is odd, the script warns about the photo left without a pair and skips it. Treat this as a sign that something is missing or extra.

## Limitations

- Rerunning the script overwrites files in `merged/` with the same numbers, but leftover files from an earlier, larger run are not removed.
- Output images keep the full camera resolution, so they can be large (around 7 MB for a pair of Pixel phone photos). To make them smaller, add for example `-resize x1600` after each `-auto-orient` in the script.
- When the two photos have different heights, the smaller one is aligned to the top and the remaining space is filled with the background color.
