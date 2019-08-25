## Week 6,7 - Show boundary detection
As discussed in the previous blog, the Imagecluster library wasn't doing a good job at clustering faces, probably because faces have much more subtle features than normal images. So I wrote a custom face clustering algorithm (<a href="https://www.pyimagesearch.com/2018/07/09/face-clustering-with-python/">reference</a>) that works as follows:
* Recognizes locations of all faces present in the frame.
* Cluster these faces using <a href="https://scikit-learn.org/stable/modules/generated/sklearn.cluster.DBSCAN.html">sklearn's DBSCAN</a> algorithm.
* Store all faces of a cluster(person) in a separate directory.

So, we now have a bunch of faces identified from the video and we know each face's class(person - we don't know the name of the person yet) and time of occurrence in the video.


### Finding possible anchors
As discussed in earlier weeks, an anchor is not someone who appears for the most amount of time in a duration, but one who is the most evenly distributed in that duration.<br> In order to put this property into code, I divided the whole video into a bunch of 15 minute segments, the logic being that if a person is a host, he/she must be present atleast once in every consecutive segment.
<br>For example if person X is the host of a show airing for 2 hours, he/she must be present in roughly 7/8 consecutive segments in the video.

### What is a show?
A 'show' can be thought of as an entity with 3 associated attributes.
* Host(s) of the show
* starting_time (where the show starts in the video)
* ending_time

<i>From here on, the word 'show' will mean just this. In the algorithm, we start with a bunch of 'shows' and filter some out to finally end up with our required 'shows'.</i>

So using the above 15-minute-segment code's output, we get a bunch of shows.
<br>For example, in a 120 minute long video, we'd have 8 segments. Let's say for each segment, the below listed persons(face-classes) appear atleast once in them.

| Segment | person | person | person | person |
|---------|--------|--------|--------|--------|
| 1       | a      | b      | c      | d      |
| 2       | b      | c      | d      |        |
| 3       | a      | b      | d      |        |
| 4       | b      | c      |        |        |
| 5       | c      | f      | g      | h      |
| 6       | g      | h      | i      |        |
| 7       | f      | g      | h      |        |
| 8       | g      | f      |        |        |

From this information, we extract the following shows: <br>

| Host | start_segment | end_segment |
|------|---------------|-------------|
| b    | 1             | 4           |
| c    | 1             | 2           |
| d    | 1             | 3           |
| c    | 4             | 5           |
| g    | 5             | 8           |
| h    | 5             | 7           |
| f    | 7             | 8           |

For simplicity, I'm only mentioning the start and end segments here. But the actual code contains the start_time (the time in the segment where the face first appeared) and the end_time for each show.
<br><br>
These are all taken as shows for now.
But in the original video, there are only two shows
* b - from 1 to 4
* g - from 5 to 8

The others are just guests or news reporters who appear in the show but are not the hosts.<br>
We have to filter these out from our list, for which after a lot of experimentation I defined a few rules that a show must follow to be considered as one. A show which doesn't follow any of these will be removed from our list
<br><br>
The filtering algorithm's rules are listed in order:
1. We remove shows that are too short (\<20 minutes) in length.
2. We remove shows within shows. For example, if show X is from 0 to 60 and show Y is from 5 to 50, show Y is removed.
3. We merge consecutive shows with very high overlap.boat For example, X is from 0 to 55 and Y is from 5 to 60. We combine these into a single show from 0 to 60 and both the hosts are considered hosts of the show.
4. We remove shows which overlap between two shows. For example, X is from 0 to 60, Y is from 65 to 130 and Z is from 50 to 80. Z is removed from our list.
5. In the end, if starting time of a show is less than the ending time of the previous show, its starting time is taken as the ending time of the previous show. For example, X is from 0 to 60, Y is from 55 to 130. Y is changed to be from 60 to 130. This problem generally arises when show Y is 'teased' during the end of show X in the video.

The resultant list of shows is very close to actual show boundaries present in the video. So, the task of finding boundaries between the shows is finished. In the following weeks, we try to find the names of the shows we've just identified.