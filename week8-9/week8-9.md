## Week 8,9 - Show name identification
We've made great progress towards identifying the boundaries between shows present in the video. Our focus from here on was to automate the process of finding the names of shows we've just identified.<br>

We could think of 2 ways of doing this:
<br><b>Anchor detection</b>
   * We identify the names of anchors for each show using some kind of classification model built on the <a href="https://megapixels.cc/datasets/msceleb/">MSCeleb dataset</a>.
   * We then use the <a href="https://www.imdb.com/interfaces/">IMDb Dataset</a> to get a list of all shows that the anchors have acted in.
   * We apply some filters on this list to get the required show's name.

<b>Extracting information from subtitles</b>
* Almost 99.9% of video files in the dataset have an associated .txt3 file containing the subtitles for that video.
* In most cases, the anchor's name and even the show's name are directly present in the .txt3 file.
* If we could somehow identify which word(s) in the .txt3 file is the name of the show/anchor, then our problem is solved.

### Anchor detection
The MS-Celeb dataset is almost 250GB in size, is hard to find, poorly documented and hard to extract. So before starting to work on the dataset, I wanted to know if that method is feasible, i.e., if we're indeed successful in finding the names of the anchors, is it possible to get the name of the show from this information?
<br>
So I downloaded the IMDb dataset on my machine and ran some tests on it using Pandas. The notebook for the same can be found <a href="https://github.com/eonr/ShowSegmentation/blob/master/W6-7-end-show_identification/shows_from_names_W9.ipynb">here</a>.

The results were promising so I took a free trial of Azure's computer vision service to test out celeb detection. The celeb identification was quick and very accurate. But we needed a free version of it, so I decided to start setting up MS-Celeb on RedHenLab's HPC cluster the following week.

### Subtitle Analysis
The feasibility tests for this were performed on .srt files instead of .txt3 files as these are easier to work with. But if put into production, .txt3 files are the ones to work on as some videos do not have a .srt file. The notebook for this can be found <a href="https://github.com/eonr/ShowSegmentation/blob/master/W6-7-end-show_identification/Subtitle%20analysis.ipynb">here</a>.<br> I used <a href="http://www.nltk.org/api/nltk.tag.html?highlight=stanford#nltk.tag.stanford.NERTagger">Stanford's NERT Tagger</a> to identify names of people and organizations(shows) present in the subtitle file, but finding out the name of the host/show from this bunch of names would require a rule-based model/ deep-learning model.

Due on time constraints, we couldn't work further on the subtitles method but only on the MS-Celeb method.
But all the information regarding the anchors' names and the show's name are present in the subtitles. One 'just' needs to find a suitable way to model/extract this information.


<b>Sample .txt3 file</b>
```
TOP|20060102000000|2006-01-02_0000_US_00001057_V11_M2_VHS10_H4_JA
COL|Communication Studies Archive, UCLA
UID|57e7f4a3-4723-4fdd-9220-af54ac0a2129
DUR|08:13:07.97
SRC|Rosenthal Collection, UCLA
CMT|
LAN|ENG
LBT|2006-01-02 00:00:00 America/Los_Angeles
20060102000037.771|20060102000042.809|SEG_00|Type=Commercial
20060102000037.771|20060102000038.071|CC1| -- Captions by VITAC --
20060102000037.771|20060102000038.071|CC1|         www.vitac.com
20060102000042.809|20060102000045.278|SEG_00|Type=Story start
20060102000042.809|20060102000045.278|CC1|>>> A GREAT 2006 TO YOU IN
20060102000045.411|20060102000045.945|CC1|NO, CITY.
20060102000046.079|20060102000047.847|CC1|WE'LL GO AHEAD AND START THE
20060102000047.980|20060102000049.916|CC1|YEAR, FIRST MONDAY AND, BOY, IS
20060102000050.049|20060102000051.517|CC1|WEATHER E WORD TODAY FOR MU
20060102000051.617|20060102000052.251|CC1|OF TH U.S.
20060102000052.385|20060102000054.053|CC1|THE O MOST DESTRUCTIVE FORMS
20060102000054.187|20060102000056.322|CC1|OF WEATHER ARE OUT THERE IN
```

<b>Sample .srt file</b>
```
1
00:00:37,771 --> 00:00:38,070
    --<font color="#ffff00"> Captions by VITAC</font> --     
        <font color="#00ffff"> www.vitac.com</font>          

2
00:00:42,809 --> 00:00:45,944
>>> A GREAT 2006 TO YOU IN      
NO, CITY.                       

3
00:00:46,079 --> 00:00:49,915
WE'LL GO AHEAD AND START THE    
YEAR, FIRST MONDAY AND, BOY, IS 

4
00:00:50,049 --> 00:00:52,250
WEATHER E WORD TODAY FOR MU     
OF TH U.S.                      

5
00:00:52,385 --> 00:00:56,321
THE O MOST DESTRUCTIVE FORMS    
OF WEATHER ARE OUT THERE IN     

6
00:00:56,456 --> 00:00:59,458
POLAR OPPOSITES.                
WE HAVE FIRE AND RAIN.          

7
00:00:59,592 --> 00:01:02,394
FIRES SCORCHING THE SOUTHWEST   
AND ENTI TOWNS HAVE BEEN WIPED
```
<br><br>In the following weeks, I worked on setting up the MSCeleb dataset on the HPC cluster and creating a celeb-detection model out of it which will be discussed in the next blog.
