# My experience with [Kaldi for Dummies tutorial](https://kaldi-asr.org/doc/kaldi_for_dummies.html)

I use data from <https://hlthu.github.io/2017/02/22/kaldi-numbers-asr.html>

I recommend you to read the [tutorial](https://kaldi-asr.org/doc/kaldi_for_dummies.html) and use this as reference only

## Notation

In terminal, `$` mean run in local and `#` mean run in a docker container

## Process

## 1. Docker

- Download Dockerimage form [kaldi/docker/debian10-cpu/](https://github.com/kaldi-asr/kaldi/tree/master/docker/debian10-cpu)
- Run
  
  `$ docker build --tag kaldiasr/kaldi:latest .`

  Note:
  1. This process takes up to 3-4 hours
  2. Make sure your storage is more than 50 GB
  
- After finish make a container (in my case I will name it **kaldi**)

## 2. Create Project

Note: I create a project in my local-pc. After finish this, I will send it to the container later.

### 2.1 Data preparation

- Download [data](https://hlthu.github.io/public/downloads/numbers-en.tar.gz) then extract and rename to `digits_audio`. The directory should be like this

  ```text
    digits_audio
    ├── test
    └── train
  ```

- create folder `digits` then move `digits_audio` into `/digits`

    ```text
    digits
    └── digits_audio
    ```

### 2.2 Acoustic data

- In `/digits` dircctory, Create folder `data` with subfolders `test`, `train` and `local` inside.
- We will have to create 5 files and file in **1-4** put into both `/test` and `/train` depend on the data and **5** put into `/local`
  
  1. **spk2gender**

      `Pattern: <speakerID> <gender>`

     In the data first five speakers are male and the last three are female

  2. **wav.scp**

     `Pattern: <uterranceID> <full_path_to_audio_file>`

     On docker, Kaldi workspace path is `/opt/kaldi/` and since we have to move the `digits` into Kaldi workspace so our full path is `/opt/kaldi/egs/digits/digits_audio/<test or train>/<speakerID>/<fileName>.wav`

  3. **text**

     `Pattern: <uterranceID> <text_transcription>`

  4. **utt2spk**

     `Pattern: <uterranceID> <speakerID>`

  5. **corpus.txt**

     `Pattern: <text_transcription>`

       Data in `corpus.txt` come form `train/text` + `test/text`

- Now your directory should look like this
  
    ```Text
    digits
    ├── data
    │   ├── local
    │   │   └── corpus.txt
    │   ├── test
    │   │   ├── spk2gender
    │   │   ├── text
    │   │   ├── utt2spk
    │   │   └── wav.scp
    │   └── train
    │       ├── spk2gender
    │       ├── text
    │       ├── utt2spk
    │       └── wav.scp
    └── digits_audio
        ├── test
        └── train
    ```

### 2.2 Langauage data

- In `digits/data/local` create folder `dict` then we have to put 4 files into the folder

  1. **lexicon.txt**

     `Pattern: <word> <phone 1> <phone 2> ...`

  2. **nonsilence_phones.txt**

      `Pattern: <phone>`

  3. **silence_phones.txt**

      `Pattern: <phone>`

  4. **optional_silence.txt**

      `Pattern: <phone>`

- Now your directory should look like this
  
    ```Text
    digits
    ├── data
    │   ├── local
    │   │   ├── corpus.txt
    │   │   └── dict
    │   │       ├── lexicon.txt
    │   │       ├── nonsilence_phones.txt
    │   │       ├── optional_silence.txt
    │   │       └── silence_phones.txt
    │   ├── test
    │   └── train
    └── digits_audio
    ```

### 2.3 Configuration file

- In `/digits` create `conf` folder and create two following file
  
   1. **decode.config**
   2. **mfcc.conf**

- Now your directory should look like this

    ```text
    digits
    ├── conf
    │   ├── decode.conf
    │   └── mfcc.conf
    ├── data
    └── digits_audio
    ```

### 2.4 Script Run

- in `/digits` create 3 following files
  
  1. **cmd.sh**
  2. **path.sh**

     we have to change path of `DATA_ROOT` to be `/opt/kaldi/egs/digits/digits_audio`

  3. **run.sh**
- form kaldi [`kaldi/egs/voxforge/s5/local`](https://github.com/kaldi-asr/kaldi/tree/master/egs/voxforge/s5/local) copy `score.txt` and place in `digits/local`
- Now your directory should look like this

``` text
digits
├── cmd.sh
├── conf
├── data
├── digits_audio
├── local
│   └── score.sh
├── path.sh
└── run.sh
```

### 2.5 Move to Docker

- In local, run `$ docker cp <path_to_digits_folder> <container_name>:/opt/kaldi/egs/` to copy `/digits` to docker container (In my case container_name is **kaldi** )
  
- In container, `/opt/kaldi/egs/digits` path
  1. create **utils** and **steps** link

     ``` text
     # ln -s ../voxforge/s5/steps steps
     # ln -s ../voxforge/s5/utils utils
     ```

  2. Install SRILM
     - execute script `kaldi/tools/install_srilm.sh`

  3. Give execute permission
     - execute `# chmod u+x local/*.sh *.sh`
  
- Now your directory should look like this (in the container)

  ``` text
  digits
  |-- cmd.sh
  |-- conf
  |-- data
  |-- digits_audio
  |-- local
  |-- path.sh
  |-- run.sh
  |-- steps -> ../voxforge/s5/steps
  `-- utils -> ../voxforge/s5/utils
  ```

### 2.6 run

- execute `# ./run.sh`
- Hola! Now you should see the `exp` folder which is the result

## Troubleshooting

- **While Building a docker image if you face a problem like this [this](https://github.com/kaldi-asr/kaldi/issues/3987)**

  ``` console
  g++: internal compiler error: Killed (program cc1plus)

  Makefile:518: recipe for target 'determinize.lo' failed
  make[3]: *** [determinize.lo] Error 1

  Makefile:358: recipe for target 'install-recursive' failed
  make[2]: *** [install-recursive] Error 1

  Makefile:414: recipe for target 'install-recursive' failed
  make[1]: *** [install-recursive] Error 1
  make: *** [openfst_compiled] Error 2
  ```

  Remove -j option in docker file and rebuild the image
- **Install SRILM**

  When running `kaldi/tools/install_srilm.sh` if it prompts that cannot find awk command.
  
  Install awk by execute `# apt install gawk`

- **The `utt2spk is not in sorted order when sorted first on speaker-id  
(fix this by making speaker-ids prefixes of utt-ids)`**

     This is a tricky one. If you already follow the convention (making speaker-ids prefixes of utt-ids). Try using this command and find the problem.

     `sort -k2 <file_name>`

     Sometimes at the end of the line has extra space, so the `sort` will not work properly. Try to find and remove it.

- **Has invalid newline or disallowed UTF-8 whitespace character(s)**

  when you get an error such as
  
  ``` text
  <file_name> has invalid newline at -e line 1, <> line n.
  ```

  or

  ``` text
  ERROR: the text containes disallowed UTF-8 whitespace character(s)
  ````

  it might be a better solution. But for me, I open the file with `vim` then save it.

- **The "one line less than actual file lines" or "Does not end in newline." [(ref)](https://stackoverflow.com/questions/12616039/wc-command-of-mac-showing-one-less-result)**

    When you get an error such as

    ``` text
    data/local/dict/<file_name> does not end in newline.
    ```

    We can verify by executing a command like this

    ``` text
    # tail -3 nonsilence_phones.txt
    w
    v
    z%
    ```

    As you can see, the last line end with `%`

    **To fix this:** execute `# printf '\n' >> <file_name>`

    **Note:** use `wc -l <file_name>` to show number of lines in file
