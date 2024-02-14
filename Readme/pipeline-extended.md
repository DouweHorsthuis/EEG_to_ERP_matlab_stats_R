## Pipeline extended

Here you’ll see what each script does and what variables you can change.

## Pre-processing

### A_merge_sets

In this script EEGlab will look for all the raw EEG files for all the
IDs you enter. Even if you only have one file per participant, you need
to run them through this script, because you end up with .set files that
have the EEGlab structure in them. Besides, defining the home/save path,
you can choose how many blocks (or .BDF files) your participants have.
Normally everyone does the same amount of blocks so this would be the
same for everyone, but if not, you need to run that participant
separately.

**It is critical that you know what files you load**, this means that
it’s probably a good idea to first read the readme files or other
information you have of each participant. This is the place where
everything becomes one big file, so it is significantly more difficult
to get rid of data that shouldn’t be used to begin with.

You can optionally choose to pause after reading in the first file if
it’s uncommon that there are multiple files. This will cause a pause in
reading in the following file, to give you time to figure it out. After
that hit a key while selecting the Matlab command console should
continue everything.

**Optional files:**

1.  Highly recommended to load readme files. In our lab we used to
    format these in a standard way, which allows us to read them here
    and add them to the EEG files. This is excelent for quality control.
    Which we will do by building dashboards per participant.

2.  EDF files, or eyetracking files. The script comes with a function
    that will create an average gaze map per participant, combing all
    EDF files it can find in the participant folder. These become PNG
    files. They can be used for the dashboard, or simply visually
    inspected to see if the participant was looking where they should.

*Note: we used to re-reference the data here (to the mastoids most of
the time), but after realizing that it’s impossible to spot flat
channels after this we pushed this to later in the script.*

These are the variables to change:

``` matlab
subject_list = {'Participant_1_ID' 'Participant_2_ID'};
home_path    = 'C:\Users\whereisyourdata\'; 
save_path    = 'D:\wheretosaveyourdatat\';  
readme_file  ='yes or no'; 
eye_tracking ='yes or no';
Pausing_yn='yes or no';
```

**Special note for `save_path`; If you set this to a general folder,
individual subject folders with the ID will be created so that
everyone’s data is in their own folder.**

10/14/2022 update We added some quality control functions here. Both are
100% optional they will be used if the following variable are `'yes'`:

``` matlab
readme_file  ='yes';
eye_tracking ='yes';
```

These functions come originally from [this
project](https://github.com/DouweHorsthuis/EEG-quality-analysis) that we
created to test the data while one collects it. They are also explained
in-dept in the readme file there.

In short: The `readme_to_EEG` function searches for a readme file (.txt
file) that we use in our lab to describe data collection. - this
function can be addapted but currently is functioning only for the
template of the readme files we use in our lab The `edf_to_figure`
function uses edf files created by our eye-tracker (sr-research eyelink
1000plus) and creates a gaze plot so you can see where the participant
looked throughout the experiment.

[Back to top](#eeg-pipeline-using-eeglab)

### Ba_cleaning_optional

This script is explained in depth
[here](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/Readme-optional_cleaning.md).
When using the `pop_clean_rawdata` different filters and settings for
the function it self impact the data heavily, specially since this
function deletes both channels and continues data. This **optional
script** will plot for you how much data and channels get rejected based
on the settings you choose. As an example, while the default settings
are 0.8 for channel correlation, and 20 for burst rejection. Using a
0.1hz and 45hz filter made us decide to set them to 0.75 and 45. The
0.05 difference cause us to loose 7 less participant and the as you can
see in the plot below, the difference between 20 and 45 for the burst
rejection “saves” us 36 participants. While this obviously comes at a
cost related to cleanliness of the data, visualizing this might make the
decision worth it.  
![cleaning
data](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/cleaning_optional.png)  
![Cleaning
channels](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/cleaning_Channels.png)

\*\*For more information on the `pop_clean_rawdata` function see [their
github](https://github.com/sccn/clean_rawdata) or read more [in this
document](#deleting_channels)

[Back to top](#eeg-pipeline-using-eeglab)

### Bb_downs_filter_chaninfo_exclchan

#### Downsampling

This script starts by loading the previously created .set file, so you
need to set the home_path to where you saved the data and the new data
will also be saved there. The first thing the script does is down-sample
to 256 Hz. We collect data at 512Hz, and there is no real reason to keep
it at this high resolution, but it does take up more space and slows
down the process when analyzing.

#### Filtering

The second step is a 1Hz high-pass filter. To change this you need to
also decide on the filter order. [See this for more
info](https://github.com/widmann/firfilt/blob/master/pop_eegfiltnew.m)

Same counts for the third step, which is a 45Hz high-pass filter. For
more detailed info on filters see the [“More info on filtering”
chapter](#more_info_on_filtering).

To optimize the ICA solutions, these are the suggested filters. However,
if you want to look at components that show up later in the data, 1Hz
might be too high. [See this paper for more info on
filters.](https://www.sciencedirect.com/science/article/pii/S0896627319301746)

#### Adding channel info

The fourth step is adding information to the channels. This is why you
need to define the path to EEGlab or to the ‘biosemi160’ file. It will
look for a file to import the channel information. The difference
between the two paths has to do with that for 64channels we use a 10-20
layout for the BIOsemi caps, however for the 160channel caps we have a
spherical layout. The first file is part of EEGlab, but this is not the
case for the 160channel cap. 64channels is defined as 64 to 95, because
a full extra ribbon would be 96 channels. In or lab we normally go up to
8 channels, but we have data that has more. This takes that in
consideration. **small update 2/14/2022** We were always using the
`standard-10-5-cap385.elp` file. However we work with caps that have the
10-20 system. Because of this we are now using the `standard_1020.ELC`
file. This is only the case for 64 channel data. When using 160channel
data, we need to rely on the biosemi file, because 160 channel data has
a spherical configuration and does not follow the 10-20 system.

#### Deleting channels

**5/31/2021 update** ~~Lastly, in step 5, it will reject channels based
on a kurtosis threshold. It is set to 5, which is the standard. Channels
with a kurtosis \> 5 will be deleted.~~ We are using the
`pop_clean_rawdata` function instead of the previously used
`pop_rejchan`. This new functions allows for more cleaning. Currently we
are only use the parts of the function that focus on channel deletion.

We delete a channel if

- It’s more than 5 seconds flat
- there is high frequency noise than is bigger 4 standard deviantionsof
  the rest of the signal
- if there is a minimal acceptable correlation with the nearby channels
  of 0.8
- **If** you set `clean_data ={'y'}` data gets segmented in 0.5 second
  blocks and if the noise exceeds a set threshold these blocks are
  deleted.
  - This threshold is currently set to 5 standard deviations, but this
    will need to be adjusted depending on how clean the data is.

1.  It’s worth it to note that when we epoch the data, we reject noisy
    epochs, so this also takes care of noisy moments.
2.  The threshold for when to delete data is found in the line that
    calls `pop_clean_rawdata`. More specifically after ‘burstCriterion’.

All of these settings are the standard settings and result in clean
data, without losing excessive amount of channels.

**important for now**  
Even though for now we cannot exclude externals from the cleaning
process, and thus we need to delete them beforehand. [The EEGLAB people
have said that they are working on a fix base on my
request](https://github.com/sccn/clean_rawdata/issues/28). Currently
(when manually updating the function) there are still errors when
excluding externals from this cleaning, but hopefully quick this will be
solved. So if you need to use externals, use the old cleaning functions,
this one is still in the code.

These are the variables you NEED to change:

``` matlab
subject_list = {'some sort of ID' 'a different id for a different particpant'}; 
eeglab_location = 'C:\Users\wherever\eeglab2021.1\'; %needed if using a 10-5-cap
scripts_location = 'C:\\Scripts\'; %needed if using 160channel data
home_path  = 'the main folder where you store your data';
```

These you can change if you want to change settings

``` matlab
downsample_to=256; % what is the sample rate you want to downsample to
lowpass_filter_hz=45; %45hz filter
highpass_filter_hz=1; %1hz filter
```

10/14/2022 update \### plotting_bridged_channels We added this function,
which uses `eBridge`. This is a function that uses the EEG structure and
checks if there are channels that are bridged. The full explanation can
be found
[here](https://psychophysiology.cpmc.columbia.edu/software/eBridge/index.html).
Or in [this
paper](https://psychophysiology.cpmc.columbia.edu/pdf/alschuler2013a.pdf)
that was written and resulted in the function. `bridge=eBridge(EEG)`
gives us a structure in which `bridge.Bridged.Labels` gives us the
labels of all the bridged channels. Later we use this to plot a figure
of the location of bridged channels. **note that bridged channels are
not deleted**. In the `plotting_bridged_channels` function we use
`eBridge` and plot the locations of the bridged channels.

[Back to top](#eeg-pipeline-using-eeglab)

#### More info on filtering

You do not need to follow the filtering in this script. EEGlab makes it
relatively easy to created/use new filters.

##### Using a new filter in EEGlab

**3/1/2022 update, we won’t need to add filter order anymore**. It is
very important to know the filter order (which is [the width of the
sliding window in data
points](https://sccn.ucsd.edu/wiki/Makoto's_preprocessing_pipeline#Dependency_across_the_preprocessing_stages_.2807.2F05.2F2019_updated.29)
or [see this EEGLAB information about
this](https://eeglab.org/others/Firfilt_FAQ.html#q-what-is-the-difference-between-filter-length-and-filter-order))

Before this EEGLAB update to their filter function, you would have to
set the filter order yourself. While still suggested in when opening the
GUI, you do not need to do this. If you do not set it yourself EEGLAB
will calculate it for you and you can find it in the output text in the
command window. as seen in figure3.

![step1](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/filtering/Open_filter.PNG "open")  
figure 1.
![step2](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/filtering/filter%20inputs.PNG "input")  
figure 2. hit “ok”  
![step3](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/filtering/figure%20out%20filter%20order.PNG "output")
figure 3.

This shows up in Matlab. Replace the filter number with the number 1.
followed by the number “preforming $$number$$ point high-pass filter” -1
as the filter order.

``` matlab
EEG = pop_eegfiltnew(EEG, 'locutoff',1);
EEG = pop_eegfiltnew(EEG, 'hicutoff',45);
```

However, but this in the script itselve, you can simply replace the
number for

``` matlab
lowpass_filter_hz=45; %45hz filter
highpass_filter_hz=1; %1hz filter
```

##### What filter should I choose

Choosing what filters to use will have a big impact on your data. There
are a couple things to consider because filters will have impact in
several different ways on your data.

**ICA** The [EEGlab
people](https://sccn.ucsd.edu/wiki/Makoto's_preprocessing_pipeline#High-pass_filter_the_data_at_1-Hz_.28for_ICA.2C_ASR.2C_and_CleanLine.29.2809.2F23.2F2019_updated.29)
suggest using a 1Hz and 45Hz filter to get the best ICA solutions. But
if one only uses the ICA for removing eye blinks and eye movement it
might be worth it to think this through.

###### Low-pass filter

We usually use a low-pass filter to get rid of high frequency noise that
cannot be caused by the brain. By using a 45Hz filter this can be
solved.Unless you are interested in specific frequencies that go above
40Hz it’s normally safe to use it. This is what the filter will do to
<a href="%5Bdata:%5D(data:)%7B.uri%7D"
class="uri">[data:\\](data:){.uri}</a>
![45hzfilter](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/filtering/Hit-Po7-downs-45hz.jpg "45Hz")  
The black line is down-sampled like the Red line + a 45Hz filter is ran
on it. If you look at the zoomed in parts it is clear the this smooths
out the ERP.

###### High-pass filter

A high-pass filter is used to stop Baseline drift. [This drift is
stronger for kids and some patient populations (0.1Hz) then for Adult
subjects
(0.01Hz).](https://sccn.ucsd.edu/wiki/Makoto's_preprocessing_pipeline#High-pass_filter_the_data_at_1-Hz_.28for_ICA.2C_ASR.2C_and_CleanLine.29.2809.2F23.2F2019_updated.29)
One other thing it helps with is solving artifacts created by sweat.  
When looking at early components, one can usually use a 1Hz filter. This
filter might however cause issues if you look at later components
(starting at P2 and onward). This is what a 1 Hz filter does
![1hzfilter](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/filtering/Hit-Po7-downs-1hz.jpg "1Hz_hit")
It’s interesting to point out that the impact of the filter is more
strongly seen the later you look at the ERP. Since we are interested in
the P1 (90-130ms) which is early, we can use this filter. The impact is
not big enough to say that it distorts the data.
![fa1hzfilter](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/filtering/Fa-FCz-downs-1hz.jpg "1Hz_fa")
In the first figure, we were interested in the first component, whereas
in the second figure we were interested in the error-related positivity
(Pe), that is around 200-400ms. Here the filter causes a pretty big
difference. For us to use these data, we need to use a lower high-pass
filter.

##### Comparing different filters

It is very important that you know what the impact of your filter can
be. One of the things to think of is what frequency you expect in the
ERP. If it’s in the higher Range you could go for a 0.5Hz or maybe a 1Hz
filter (1Hz might be too much according to some).
![1-01-001hzfilters](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/filtering/Hit-Po7-downs-1-01-001hz.jpg)
As you can see here, the filters seem to have more or less the same
impact, since we are looking at the early components.  
However here we want to look at later components and we are looking at a
ERP based on a False alarm which should be a lower frequency response.
![001-01-1hzfilters](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/filtering/Fa-FCz-downs-1-01-001hz.jpg)  
Here the 1Hz filter has way too much effect and distorts the data.

##### Comparing different filter orders

In the following 2 plots we ran the same 1Hz filter on the data, but we
changed the filter order. The standard filter order suggested by EEGlab
for a 1Hz filter is 1690 (this order is different for different filters;
a 0.1Hz filter has a suggested order of 16895). We chose 846 as filter
order because this is the smallest filter order that can be chosen for
this filter. If you would choose a different filter the minimum is
different. There is no max filter order, but when we tested 100000000 as
an order, after 2 days EEGlab hadn’t completed 5%.

![hit-filteroder](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/filtering/filterorder_hit.jpg)
![fa-filteroder](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/filtering/filterorder_fa.jpg)

##### Final product

This is the combination of a 1Hz and a 45Hz filter
![1hz_45hzfilter](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/filtering/Hit-Po7-downs-1%252645hz.jpg "1Hz_45Hz")

[Back to top](#eeg-pipeline-using-eeglab)

### C_manual_check

In this script each subject’s data gets loaded, externals get deleted
and the data gets plotted in the EEGlab GUI. Make sure you always set
the scale to the same number, since EEGLAB bases it on the noise level
of the data. In the GUI set the scale to 5, so you can see if there are
flat channels. **to clarify, this is not an optional script, the
pipeline does not work without it. This is on purpose since looking at
the data is a very (or maybe the most) important step in pre-processing
the data.**

(example)

![flat
channels](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/flat%20channel.PNG "flat channel")
After that Change the scale to 50 (this value will be automatically set
and different for each data set). It is important to always set it to
the same scale, so that you can compare noise between data sets.

<figure>
<img
src="https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/very%20noisy.PNG"
title="noisy channel" alt="Noisy channels" />
<figcaption aria-hidden="true">Noisy channels</figcaption>
</figure>

When looking at 160 channel data, be sure to check in settings how many
channels you want to see. Because it is clear that there are bad
channels, but their label is hard to spot.

<figure>
<img
src="https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/160-badchan.PNG"
title="160 noisy channel" alt="160 channels" />
<figcaption aria-hidden="true">160 channels</figcaption>
</figure>

After figuring out which channels to delete, type their labels in the
command window of Matlab. They need to be entered in the following
structure:

``` matlab
{'FC1' 'P1' 'po3'}
```

**Be critical, but if you delete too many channels (\>10) you should
consider whether that data set should be included.**

### plot_deleted_chan_location

This function uses the locations of the deleted channels and creates a
scalp map with these locations so one can see if too many channels close
to each other are deleted.

### plot_group_deleted_chan_location

This function plot on a group level how often a channel is deleted. This
means that if a channel is deleted, on the location of that channel in
the headmap you will see a number. That number indicates how often that
channel is deleted. This is saved as one figure.

### quality

This is a variable that will store the ID, % deleted data, seconds of
data left and N - deleted channels of each participants. When all the
individual subjects are run, we load `participant_info` and include this
variable and save it again.

[Back to top](#eeg-pipeline-using-eeglab)

### D_reref_exclextrn_interp_avgref_ica_autoexcom

#### Interpolate

After that we re-referencing we interpolate. We moved this up from where
it was before (after the ICA), because this allows us to use the ICA
weights and gives us all the channels.

Here we interpolates all the channels that got deleted before. It does
this using the pop_interp function. It loads first the \_info.set file
(that was created in B script) to see how many channels there were
originally (it deletes the externals because you don’t want these in
your new data). Then we use the pop_interp to do a spherical
interpolation for all channels that were rejected.

**It is important to realize that if too many channels from around the
same location are rejected, the newly formed channels have inaccurate
data.**

For each participant, data get stored containing the ID number and how
many channels were interpolated.

These are the variables you NEED to change:

``` matlab
subject_list = {'some sort of ID' 'a different id for a different particpant'}; 
name_paradigm = 'name' % this is how the .mat file with the info is being called
home_path  = 'the main folder where you store all the data';
```

These you can change if you want to change settings

``` matlab
EEG = pop_interp(EEG, ALLEEG(1).chanlocs, 'spherical');% 
```

[Back to top](#eeg-pipeline-using-eeglab)

#### Average reference

After this we reference the data to the average. There are mulitiple
reasons to do this but we do it mainly for 2 reasons.  
1, in preparation for Independent Component Analysis (ICA).  
2, [The extra referencing step, will give you 40 dB extra CMRR (Common
mode rejection ratio)](https://www.biosemi.com/faq/cms&drl.htm)

We only do it now because we just interpolated, which minimizes
potential bias in the average referencing stage. For example, if there
are 64 channels, and 16 channels are identified as bad and rejected but
only from the right hemisphere. Then, the number of channels in the left
vs. right hemispheres are 32 vs. 16, with which average will be biased
toward the left hemisphere. To avoid this bias, scalp electrodes may be
interpolated. [See Makoto’s pipeline for more info, the previouse text
is his
explanation](https://sccn.ucsd.edu/wiki/Makoto's_preprocessing_pipeline#Interpolate_all_the_removed_channels_.2803.2F05.2F2021_updated.29)

[Back to top](#eeg-pipeline-using-eeglab)

### PCA

The PCA is set to the amount of channels deleted -1 (for the average
reference), the PCA will dictate how many components the ICA will
create. This is especially important because we are interpolating and
doing an average reference before the ICA. This could cause “ghost
components”, or just make the data go bad all together due to working
with data that is rank deficiant. Setting the PCA preferents this rank
issue. Another solution is to delete a channel (after the average ref).
But this would still not solve the issue for the interpolated channels +
we would lose a good channel.

[Back to top](#eeg-pipeline-using-eeglab)

### ICA

We are using the pop_runica function, as suggested by EEGLab, but there
are other options that might be quicker (this might, however, come at a
cost). We do an ICA mainly to delete artifacts that are repeated, such
as eye blinks, eye movement, muscle movement and electrical noise.

[Back to top](#eeg-pipeline-using-eeglab)

### IClabel

We are using
[IClable](https://www.sciencedirect.com/science/article/pii/S1053811919304185)
as a function to automatically label the components. After that, we only
delete the eye-components. We only focus on eye-blinks because we know
they have a bad/strong impact on the data and as you can see in [the
next part](#impact-of-ica-on-simple-erps) deleting more has a very
strong impact on the data and we are not sure what gets deleted. In our
case components will only get deleted if they are \>80% eye and \<10%
brain. We decided on these criteria after comparing how many components
experts in our lab would delete and what criteria would match this the
closesed.

Matlab will save a figure with the deleted Eye components and with all
the remaining components grouped separately. Lastly, Matlab will save a
variable called components, with the ID and how many of each type of
component reached the threshold.

These are the variables you NEED to change:

``` matlab
subject_list = {'some sort of ID' 'a different id for a different particpant'}; 
home_path  = 'the main folder where you store all the data';
```

These you can change if you want to change settings

``` matlab
EEG = pop_runica(EEG, 'extended',1,'interupt','on'); % you can choose a different ICA function, or command it out if you don't want to use it (you will also need to command out the IClable part)
bad_components = find(ICA_components(:,3)>0.80 & ICA_components(:,1)<0.05);% look for >80% eye and <5% brain
```

[Back to top](#eeg-pipeline-using-eeglab)

#### Impact of ICA on simple ERPs

This is an example of what the ICA does to an ERP.
![ERP-ICA-vs-no-ICA](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/filtering/1%252645-withandwithout-ica.jpg)

Here you see the impact of the ICA on data after using IClabel to auto
delete bad components. While all of the components that are deleted are
non-brain related and we even check that within the components there is
less than 5% brain data, the impact is huge. When using ICA always make
sure to use the right setting and make sure your data look the way it
should. Doing it manually takes a lot of practice and understanding of
the data and a lot of time. This is why we currently prefer the more
objective [IClabel](https://github.com/sccn/ICLabel) function in EEGlab

To make more sense of the impact of deleting components I’ve plotted 4
different ERPs.

- For the first ERP I deleted for each participant every component if
  the labels\* of the sum of bad\*\* components \>90 and the brain label
  \<3%\*\*\*
- For the second ERP I only deleted a component if the eye label would
  be \>80% and the brain label \<5%
- For the third ERP I deleted for each participant every component if
  the labels of the sum of bad components \>80 and the brain label \<5%
- For the fourth ERP I did not delete any component, this is the ERP
  before IClabel is ran.

The first plot is an VEP directly after seeing a stimulus  
![ERP-ICA-vs-no-ICA-hit](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/iclabledifferences_hit.jpg)  
The second plot is an ERP after a False Alarm (button press when they
were supposed to inhibit)
![ERP-ICA-vs-no-ICA-fa](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/iclabledifferences_fa.jpg)

\* IClabel labels for each component how much % they are made up out of
$$Brain, muscle, eye, Heart, Line Noise, channel noise and other$$  
\*\* We only use a sum of muscle, eye, Heart, Line Noise, channel noise
to create bad components  
\*\*\* every label will always have something above 0%, this is why I
didn’t want to go lower then 3%

#### re-referencing

Thanks to the suggestion of [Ana
Francisco](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/issues/20)
we moved this to after the ICA. Since the average reference would undo
this part.  
After realizing that re-referencing causes flat channels to have the
data of the reference channel and thus making it impossible to see if
it’s flat, we only re-reference here (after having deleted all the
channels that are noisy/flat).  
You can choose a reference channel. [Biosemi explains why it matters for
their system](https://www.biosemi.com/faq/cms&drl.htm) but that you
should [delete flat channels
first](https://www.biosemi.nl/forum/viewtopic.php?f=7&t=810&p=3871#p3871).
[Brainproducts](https://pressrelease.brainproducts.com/referencing/) and
[this
paper](https://www.frontiersin.org/articles/10.3389/fnins.2017.00205/full)
Also agree that mastoids are commonly used and good, the paper also
talks about different options that could work.

In our case we use the average of both mastoids (channel 65/66 for us),
but you can change this to a different channel or leave it empty if you
don’t want to do this.

This is the impact it has on our data. Here we compare data referenced
to the mastoid externals with data where we left the ref_chan variable
empty.  
![hit-ref](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/hit-po7-ext-noext.jpg "hit-ref")  
![fa-ref](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/fa-fcz-ext-noext.jpg "fa-ref")  
The first plot is the ERP after a Hit. The second one is after a False
alarm. It is clear that the amplitudes increase significantly, however
it does seem like the standard error also increases.

[Back to top](#eeg-pipeline-using-eeglab)

### E_epoching

This is the last file for pre-processing the data. In this script the
interpolated data get epoched, cleaned, and transformed into an ERP.
Some of these functions are ERPlab based.

Firstly, the data needs to have their events (or triggers) to be
updated. You need to create a text file that assigns this info. There
are 2 ways of doing this.  
You can define all trials using an eventlist. This is more restrictive,
because it seems like you cannot add a sequence of triggers, only
individual ones [See this tutorial for more
info.](https://github.com/lucklab/erplab/wiki/Creating-an-EventList:-ERPLAB-Functions:-Tutorial).  
Instead, we use the Binlist option. You can create as many bins as
needed. Each bin will create a different ERP, and if you want to use
ERPlab to plot ERPs you can choose which ones to plot. If you want to
use another program it might be worth it to just save the ERPs of 1
specific bin and run the script multiple times. [see this for
information on how to create a binlist.txt
file](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/blob/main/images/binlist.PNG),
or [see this
tutorial](https://github.com/lucklab/erplab/wiki/ERP-Bin-Operations)

After that you set the time for the epoch. This is pre-defined in the
variable epoch_time and baseline_time at the start. After that we use
the pop_artmwppth function to flag all the epochs that are too noisy. We
save this info, after which we delete them.  
**bug when using EEGlab V2019_1**  
You need to create at least 2 bins, or ERPlab breaks when you want to
average the ERPs. When plotting you can always choose not to use one of
the two bins. Later on in the F_individual_trials_export script you can
choose to only include the bin you want to (or multiple if that is what
you want). This is not the case if you use EEGlab V2021.1 **bug when
using EEGlab V2019_1** Lastly we create the ERPs and save the data as a
final .set file. You will also have a file at the end with each
participant’s ID number and the percentage of data that got deleted.

You can choose to use the pop_binlister function (see line 35).

~~If you end up wanting Reaction time for the ERPs, consider including
the
[pop_rt2text](https://github.com/lucklab/erplab/blob/master/pop_functions/pop_rt2text.m)
function. For this to work you need to define the event that reflects
the reaction you want to measure in the eventlist.~~ We created a
different script for this

These are the variables you NEED to change:

``` matlab
subject_list = {'some sort of ID' 'a different id for a different particpant'}; 
name_paradigm = 'name'; % this is needed for saving the table at the end
participant_info_temp = []; % needed for creating matrix at the end
home_path  = 'the main folder where you store your data\';
eventlist_location = 'the folder where you stored your eventlist\'; %event list should be named event.txt
epoch_time = [-50 400]; %epoch start and end time in ms
baseline_time = [-50 0];%baseline start and end time in ms
```

These you can change if you want to change settings

``` matlab
EEG = pop_interp(EEG, ALLEEG(1).chanlocs, 'spherical');% 
```

[Back to top](#eeg-pipeline-using-eeglab)

### F_RT

This script continues where the previouse one left in case you want
reaction times. We adds all the RTs together in an excel file and a
matlab file. This file Contains 4 columns with the
IDs,group,Condition,RTs of each particpant. The script requires that the
.erp file has been created using a binlist that stores reaction times.

[Back to top](#eeg-pipeline-using-eeglab)

## Visualizing

The next 2 script focus on visualizing the final data. Both on a group
level and on a individual level. We will focus on quality and ERPs.

[Back to top](#eeg-pipeline-using-eeglab)

### G_building_dashboard_group

This is running tested and gives you a nice quality focused dashboard
(PDF.) per participant and on a group level Will add more info later.

[Back to top](#eeg-pipeline-using-eeglab)

### H_grandmeans

Here you can plot grandmeans and modify how you want to look at your
data, which channels you want to see etc. Will add more info later.

[Back to top](#eeg-pipeline-using-eeglab)

### I_individual_trials_export

If you want to do stats in R or any program that doesn’t read .mat
files, you need to export your data. This happens in the F script. [For
detailed info see its own
repo](https://github.com/DouweHorsthuis/trial_by_trial_data_export_eeglab).

[Back to top](#eeg-pipeline-using-eeglab)

## Statistics

### Loading and setting up the structure

Here we compute statistics using R and Rstudio. The script does the
following. It imports data from .txt file, it creates factors and it
randomly selects 200 trials per subject (so everyone has equal amounts
of data)

### Statistics (including mixed-effects models)

First it creates a summary of the data, which includes mean and standard
deviation. After that it plots the data in a violin/box plot.

You can design your mixed-effects model and, after that, create a
summary with the results of your model computation.

[Back to top](#eeg-pipeline-using-eeglab)

# Issues

See the [open
issues](https://github.com/DouweHorsthuis/EEG_to_ERP_pipeline_stats_R/issues)
for a list of proposed features (and known issues).