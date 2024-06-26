# Prototypes using Gallery to read Art files

## ROOT Macro
### What It Does
The ROOT macros takes two arguments:
 * a text file containing a list of files, and
 * the number of files to run over

and makes a simple histogram of the reconstructed momentum at the tracker entrance for downstream electron tracks.

### How to Run

```
source setup.sh
root -l 'art_gallery_prototype_macro.C("../filelists/mcs.mu2e.CeEndpointMix1BBSignal.Tutorial_2024_03.art.list", n_files_to_read)'
```

## Compiled Program
### What It Does
The compiled program takes three arguments:
 * a JSON configuration file,
 * a text file containing a list of files, and
 * the number of files to run over

The program reads the configuration file to make multiple histograms. It current supports plotting and cutting on the following variables:
 * tracker fit momentum (```"momentum"```)
 * tracker fit time (```"time"````)
 * tracker fit surface id (```"surfaceId"```, 0 = entrance, 1 = middle, 2 = exit)

Note: this works on both SL7 and AL9 nodes

### How to Run
```
source setup.sh
mkdir build/
cd build/
cmake ../
make
./art_gallery_prototype ../hists.json ../../filelists/mcs.mu2e.CeEndpointMix1BBSignal.Tutorial_2024_03.art.list 2
```

### How to Generate JSON File
You can generate the JSON file using Apple's Pkl configuration language (https://pkl-lang.org/index.html). This isn't centrally available on the mu2egpvms at the moment but can be downloaded easily by:

```
curl -L -o pkl https://github.com/apple/pkl/releases/download/0.25.3/pkl-linux-amd64
chmod u+x pkl
```

Then, to create the JSON file, you can write a ```.pkl``` file (see ```example.pkl```) for an example and generate the ```.json``` like so:
```
./pkl eval -f json configs/pkl/example.pkl >| configs/json/example.json
```

Pkl does a bunch of validation checking and looks like it could be of general use.

### Classes

* ```Variable```: contains a pointer to a function that will return the requested variable.
   * Supported variables:
   * from ```KalIntersection```:
       * ```"momentum"``` is track momentum
       * ```"time"``` is track time
       * ```"surfaceId"``` is the track surface ID
   * default: ```""``` will always return 0
* ```SingleCut```: from a given string, it determines the variable, comparison operator, and value to compare against
   * defined in configuration as a simple string like ```"variable>number"```
* ```Cut```: starts to handle multiple cuts
   * currently only two ```SingleCuts``` with an ```&&``` is supported
* ```Hist```: contains a ```TH1F```, a ```Variable``` and a ```Cut```

## Pros and Cons

Andy's Pros:
 * can access everything in the Event
 * no art module overhead

Andy's Cons:
 * no access to SubRun- or Run-level data products
 * need to know details of full data products
   * e.g. for compiled program, need to know location of the Mu2e header files
 * will need to build our own configuration interface to e.g. have user choose histogram limits, cuts etc.
   * with the compiled program's JSON configuration, I feel like I'm reinventing the wheel

## Next Steps
* For ROOT macro
  * Make histograms with time cuts
  * Add a simple configuration interface?
* For compiled program:
  * make argument parsing safer
  * multiple cuts (including event-level cuts)
  * mathematical operations of variables (reinventing the ROOT TFormula's?)
  * add TTree as an optional output for unbinned fits
  * how to define collections to loop over

## Other notes

| commit hash | script | notes | time per event [ms] | total time [ms] | comment |
|----|----|----|-----|-----|------|
| 5bff4d2 | art_gallery_prototype_macro.C | one file: mcs.mu2e.CeEndpointMix1BBSignal.Tutorial_2024_03.001210_00000255.art | 0.353 | 5742 | |
| c2e809c | art_gallery_prototype_macro.C | as above but with start_time defined before histogram making | 0.367 | 6032 | |
| e5e9d64 | art_gallery_prototype_main | one file: mcs.mu2e.CeEndpointMix1BBSignal.Tutorial_2024_03.001210_00000255.art | 0.366 | 6615 | |
| b29b4ce | art_gallery_prototype_macro.C | added writing of output file | 0.370 | 6133 | |
|  | art_gallery_prototype_main | as above | 0.375 | 6800 | |
| dd2728c |  | can now run over a list of files | | | |
|   | art_gallery_prototype_main | 1 file | 0.369 | 6706 | |
|   | art_gallery_prototype_main | 2 files | 0.364 | 11939 | |
|   | art_gallery_prototype_main | 10 files | 1.304 | 192561 | unclear why time per event has changed... |
|   | art_gallery_prototype_macro.C | 1 file | 0.372 | 5990 | |
|   | art_gallery_prototype_macro.C | 2 files | 0.370 | 11527 | |
|   | art_gallery_prototype_macro.C | 10 files | 1.332 |191143  | macro also sees an increase in the time per event... |

### Time per event vs N Files
Commit hash: fa19843
Script: art_gallery_prototype_macro.C
Dataset: mcs.mu2e.CeEndpointMix1BBSignal.Tutorial_2024_03.art

| N Files |  Time per event (1st attempt) [ms] | Time per event (2nd attempt) [ms] | Time per event (3rd attempt) [ms] | Time per event (4th attempt) [ms] |   Time per event (5th attempt) [ms] |
|----|----|----|----|----|----|
| 1 | 1.928 | 0.359 | not done | not done | not done |
| 2 | 0.536 | 0.360 | not done | not done | not done |
| 3 | 0.494 | 0.378 | not done | not done | not done |
| 4 | 0.749 | 0.367 | not done | not done | not done |
| 5 | 0.797 | 0.376 | not done | not done | not done |
| 6 | 0.421 | 0.463 | not done | not done | not done |
| 7 | 1.180 | 0.891 | 0.573 | 0.729 | 0.873 |
| 8 | 1.016 | 0.816 | 0.738 | 0.844 | 1.677 |
| 9 | 1.552 | 1.966 | 1.627 | 1.281 | 1.107 |
| 10 | 1.247 | 0.999 | 0.879 | 0.824 | 0.818 |
