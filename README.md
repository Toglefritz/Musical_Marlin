# :musical_score: Musical_Marlin  :musical_score:
Make your 3D printer play music!

## Introduction
If you own or have used a 3D printer, you doubtless know that, unless your's is equipped with "silent" stepper drivers, 3D printers make a whole range of... interesting... noises while they operate.

Well, with the firmware in this repository, you can make all those random noises serve a higher purpose. By making use of acoustic science and clever timing, we will make your 3D printer play a little tune. The cool thing is that we will perform this upgrade without adding any hardware to your printer - no speakers even!

We will make use of an age-old hack to play music using stepper motors by making them vibrate at specific frequencies. With a couple modifications to the Marlin firmware, your 3D printer can play a lovely song before your print begins.

## Installation
Instructions for installing the firmware onto your Creatlity Ender 3 and creating music files to place on the SD card can be found on Instructables [link TBA after Instructable is published].

## Translating Music to G-Code
Here's the thing: 3D printers are not, as it turns out, built to play music. The controller inside the Ender 3 3D printer is, in essence, just a G-code interpreter. Therefore, the crux of this project is to translate musical notes to G-code.

### What is a Note?
To begin the explanation of how musical notation can be translated to into G-code, we first need to understand what a musical note represents. Musical notes encode two pieces of information: a frequency and a duration.

First, musical notes express a tonal frequency. Each letter in music (A, B, C, D, E, F, G, plus their various flats/sharps) is [defined as a frequency of sound in Hz](https://en.wikipedia.org/wiki/Piano_key_frequencies). There is a spreadsheet in this repository that lists the frequencies for three octaves of a musical scale.

Second, musical notes express the duration to play each tone. Different types of notes, like quarter notes, eighth notes, half notes, and so on, is a different duration based on the tempo for the song. Tempos are expressed in beats per minute. For example, the Song of Storms from Zelda Ocarina of Time has a tempo of 160 beats per minute. Although musicians do not think of notes this way (because they are not machines), the tempo gives the absolute length of time to play each note. In the Song of Storms example, each beat lasts 0.00625 minutes.

### Translating Music to G-Code
The G-code in question that we are looking to generate from music is the G0 command for a linear move. This command takes two main arguments. First, a distance to move one (or mroe but in this case just one) axis and a feedrate. So, for examnple, G0 X5 F100 means "move the X axis +5mm at a speed of 100 mm/min." Our goal is to map a note frequency to a feedrate and a note duration to a distance.

#### Note Frequency to Feedrate
The first challenge is to convert a frequency (in Hz) to a feedrate (in mm/min) given the steps per millimeter for an axis. For the Ender 3, The X-axis has a steps per millimeter value of 80; this is a fixed value based on the design of the 3D printer.

First, the note's frequency in Hz is equal to the target step rate for the stepper in steps per second. For example, the note C4 (middle C) has a frequency of 261.6256 Hz. If the stepper does 261.6256 steps per second, it will vibrate in a way that makes the C4 tone. We can easily convert this step frequecy to steps per minute to match the time scale of the freedrate (millimeters per minte). 261.6256 steps/s * 60 s/min = 15697.536 steps/min. 

Second, for that steps/min value we can calcualte how much distance the axis will cover in a minute based on the steps per millimeter value of 80 for the X-axis. In other words, if the stepper "plays" a note of C4, how much distance will be covered in a minute. 1597.536 steps/min ÷ 80 steps/mm = 196.2192 mm/min. So, while the X-stepper is "playing" C4, it moves at 196.2192 mm/minute. This is the feedrate for C4 then!

#### Note Duration to Distance
Now, for duration, since we know the tempo for the song and the note value, we know how long each quarter note in the song lasts. So, for example, as discribed previously, in Song of Storms, a quarter note has a duration of 0.00625 minutes. The distance covered by the stepper motor in this time period depends upon the rate at which that stepper is moving (the feedrate). To calculate the duration for each note, we simply multiply the duration of each note (in minutes) by the feedrate (in mm/min). For a quarter note at a tempo of 160 beats/min playing C4, 0.00625 min * 196.2192 mm/min = 1.22637 mm.

#### Creating the G-Code
So, bringing together the frequency to feedrate and duration to distance calculations, we can create a G-code command that will make the stepper motor make a C4 quarter note. The only little caveat is that we round the feedrate so it does not have a decimal value.

G0 X1.22637 F196

There is one additonal G-code needed to string a bunch of notes together. We will need to set the printer to use relative positioning rather than absoluate so it will keep moving along as we issue more commands. We can set relative positioning using the "M305 X1" command.

## Adding Songs to Marlin
So, the above sections, which are quite far into the weeds, describe translating music into G-code. How do we add this to Marlin? The way I've chosen to do it for this project is to add custom commands to the LCD interface to play songs. This is actually quite easy. In the Marlin firmware code, custom LCD commands can be created in the configuration_adv.h file. In there, there is a secion for #define CUSTOM_USER_MENUS. By commenting out the different lines, like #define USER_DESC_1 "Play a scale", we can add additional items to the LCD menus. 

Each menu item has two parts. The name is listed in the lines like #define USER_DESC_1 "Play a scale". Each menu item performs a custom set of G-code commands. These are listed in the lines right below the names. So, we will take the cutsom G-code representing the songs we want the printer to play and add those to the custom menu items. For example, the G-code below will play Song of Storms from Zelda. THe tralsation of this song into G-code is described in a spreadsheet in this repository.

> #define USER_GCODE_2 "G17 \nM350 X1 \nG91 \nG0 X0.6875 F220\nG0 X0.81875 F262\nG0 X1.6375 F131\nG0 X0.6875 F220\nG0 X0.81875 F262\nG0 X1.6375 F131\nG0 X4.63125 F494\nG0 X1.6375 F524\nG0 X1.54375 F494\nG0 X1.6375 F524\nG0 X1.54375 F494\nG0 X1.225 F392\nG0 X4.125 F330\nG0 X2.0625 F330\nG0 X1.375 F220\nG0 X0.81875 F262\nG0 X0.91875 F294\nG0 X6.1875 F330\nG0 X2.0625 F330\nG0 X1.375 F220\nG0 X0.81875 F262\nG0 X0.91875 F294\nG0 X4.63125 F247\nG0 X0.6875 F220\nG0 X0.81875 F262\nG0 X5.5 F440\nG0 X0.6875 F220\nG0 X0.81875 F262\nG0 X5.5 F440\nG0 X4.63125 F494\nG0 X1.6375 F524\nG0 X1.54375 F494\nG0 X1.6375 F524\nG0 X1.54375 F494\nG0 X1.225 F392\nG0 X4.125 F330\nG0 X2.0625 F330\nG0 X1.375 F220\nG0 X0.81875 F262\nG0 X0.91875 F294\nG0 X4.125 F330\nG0 X2.0625 F330\nG0 X4.125 F220"

When we select that custom command, the printer will "play" Song of Storms.
