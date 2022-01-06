---
layout: post
title: "Digital Photography Notes - Autofocus and Autoexposure"
categories: note
tags: photography
---

> This note is based on lectures 6 and 7 of Levoy's Digital Photography. It covers autofocus and metering. I found some content irrelevant so I moved them somewhere else for now.

<!--more-->

- TOC
{:toc}

# Goals For This Note

This note is based on lectures 6 and 7 of Levoy's Digital Photography. It covers autofocus and metering. I found some content irrelevant so I moved them somewhere else for now.

# Autofocus
For a given distance from the lens to the sensor, only one distance from the lens into the scene will be in focus (see [Optics]({% post_url 2021-11-01-optics %}#3d-perspective-transform)). In a "poorly focused" image, then, the thing we care about is not at this in-focus distance; see an example below:

![unfocused.jpg]{: width="400"}

## Anatomy
Let's revisit the SLR cross section shown in [Image Formation]({% post_url 2021-10-20-image-formation %}#single-lens-reflex-slr-camera).

![autofocus.png]{: width="400"}

The reflex mirror (big diagonal green line) is actually semi-transparent, letting some light through to another small mirror, which redirects it to an **autofocus** unit, which adjusts the lens distance (black arrows). When light bounces through to the optical viewfinder, some of it is directed to an **autoexposure** unit, which will adjust the pupil size. The horizontal green line is called the **focusing screen**.

## Autofocus Modes
There's three main autofocus modes (exact names vary by brand):
- **Single-shot AF**: focuses once then locks the focus
- **Continuous AF**: continuous focus without locking; may intelligently track a subject so focus doesn't lag
- **Automatic AF**: continuous focus when the subject is determined to be moving; locks focus otherwise

## Types of Autofocus
There's several focusing methods:
- **active autofocus**: uses an active sensor, where the camera emits light or sound and detects how it reflects off the subject
- **passive autofocus**: calculated only from the light entering the lens.
    - **phase-based**
    - **contrast-based**

The video lecture makes the mechanisms behind both types of autofocus so clear and a text summary won't do them justice. Watch starting [here](https://youtu.be/njlDb9-6aOU?t=2060) (timestamped). You can also play with the applets yourself: [phase](https://sites.google.com/site/marclevoylectures/applets/phase-detection) and [contrast](https://sites.google.com/site/marclevoylectures/applets/contrast-detection).

## Phase Detection
![autofocusapplet.png]{: width="500"}
*The disparity between the red and green peaks is shown at the bottom.*

The light on either side of the optical axis hits the secondary mirror, passes through one of two lenslets, and strikes one of two small CCD sensors. As the image moves in and out of focus on the main image sensor, the images on the CCD sensors shift. The **disparity** is the distance between the focus points on the CCD sensors, and when reduced past a certain point, the image will be focused on the main sensor. It's equivalent in principle to depth-from-stereo in computer vision.

In practice, the signals won't be single peaks as shown in the video, but more complex waveforms, and the distance to match the signals can be determined by **cross-correlation**.

![afpoints.jpg]{: width="400"}

The signals are calculated for small regions called **AF points** (aka autofocus points, focus points, or focusing points), with each point having its own set of mirrors and CCDs. Higher-end cameras can have over 50 AF points.

![dualtypeautofocus.jpg]{: width="400"}

The light entering either lenslet originates on opposite sides of a certain axis, say the vertical. That means horizontal disparities are easily calculated, while vertical ones are not. We can solve this by rotating the image 90 degress, but a better solution is to use **cross-type** AF points, which doubles the number of mirrors and CCDs per focusing point, and permits disparity calculation on both the horizontal and vertical.

The relative position of the CCDs also matters, and spreading them out increases the accuracy of the disparity calculation. A **double cross-type** AF point adds four *high precision* CCDs on the far corners of a cross-type AF point to increase the accuracy of the disparity calculation (they are high precision only due to their greater separation). Note that these AF points take up more space so they are saved for the center of the image, and are typically limited to very wide apertures, like $$\geq f/2.8$$.

![autofocussensor1.jpg]{: width="500"}
*Canon 7D autofocus array*

Here's an example autofocus CCD array represented in several different ways. To understand the diagram, note that the AFP numbers in the table correspond with the numbered white squares in the top subimage, and the horizontal and vertical numbers in the table correspond with the numbered colored CCDs in the middle subimage. Since every AF point in this array happens to be a cross-type (which isn't necessarily true even in high-end cameras), each horizontal and vertical comes in pairs, hence the a and b for each number.[^double] Finally, the bottom subimage shows how the CCDs are shared among the AF points.

Phase detection has the advantage of knowing how much and in what direction to change focus, based on the disparity, so it's fast. Most SLRs use phase detection.

Here's an excellent [article](http://www.exclusivearchitecture.com/?page_id=1291) on autofocus that goes into a little more depth.

## Contrast Detection
![autofocusapplet2.png]{: width="500"}
*The detected contrast is shown on the right*

A much simpler system. If the light does not converge at the main sensor, the image will be blurred, and the contrast in a small region will be low. Contrast is calculated using a local **gradient** of pixel values. The camera will then change focus until it detects that the gradient has reached a maximum value. It's equivalent in principle to depth-from-focus in computer vision.

There is no way to tell how much to change at any given time, so the camera has to "hunt" for the best focus, making it slow.

Most point and shoot and cell phone cameras use contrast detection, as well as DSLRs in live view.

## Notes
- Both approaches will fail on untextured images! Phase detection may also have trouble with repeating textures.
- Both approaches will fail in low light! In that case the camera may light up the scene with an **autofocus lamp**.
- Phase detection doesn't work well for longer focal lengths.
- Phase detection can experience calibration errors.
- There's a certain balance between focusing and metering. If the image is unfocused, the metering won't be accurate, and vice versa. There will probably be some kind of iterative process based on the camera.
- Poor autofocus is more noticeable with shallow depth of field.

## Alternatives
- **Laser time-of-flight**: Active autofocus. A beam of light hits some object of interest and the time it takes to return is measured, giving a focus depth. It is fast like phase detection but may fail in the outdoors or on windows.
- **Hybrid pixel**: The phase detection system is basically consolidated into single pixels, which are half covered so each pixel detects only the right or left half of the image. It becomes noisy in low light, considering each pixel is effectively cut in half. The pixels are spread throughout the main sensor, so there is no separate autofocus system. Depending on the implementation, the pixels may or may not be used for normal image capture as well; if not, then interpolation is required over the dedicated autofocus pixels. More information and visuals [here](http://www.exclusivearchitecture.com/?page_id=1332).
- **Dual pixel**: The successor to the hybrid pixel, where pixels are not dedicated to the left or right halves. Instead, each pixel is divided into two sub-pixels for autofocus purposes, and can be combined for regular image capture purposes. Every pixel on the main sensor is a dual pixel, so there's no interpolation concerns, and the autofocus system has a higher resolution (more data to work with), so it's less susceptible to noise and faster (but still not as fast as traditional phase detection). More information and visuals [here](http://www.exclusivearchitecture.com/?page_id=1342).

# Autoexposure
Just as a picture can be poorly focused, it can also be poorly exposed. Look at the picture below and you will see what an *incorrect* exposure might be.

![overexposed.jpg]{: width="400"}

The picture above is **overexposed** (colloquially: blown out); the opposite would be **underexposed**.

## Dynamic Range
The ratio of the minimum and maximum of any signal (here, light) is called the **dynamic range**.

Estimated dynamic range of the world:[^source]
- 800,000:1 – surface illuminated by the sun vs by the moon
- 100:1 – diffuse white surface vs diffuse black surface
- 80,000,000:1 – total dynamic range, from multiplication

Estimated dynamic range of human vision:
- 100:1 – photoreceptors range
- 10:1 – variation in pupil area
- 100,000:1 – neural adaptation
- 100,000,000:1 – total dynamic range, from multiplication

Estimated dynamic range of media (debatable):
- 40:1 – photographic print (more for glossy paper)
- 50:1 – artists paints
- 500:1 – negative film
- 1,000:1 – LCD display (may be outdated)
- 4,000:1 – digital SLR sensor

We can also express how many bits it would take to represent a given dynamic range $$d$$ by taking $$\vert\log_2(d)\vert$$. For example:
- The world – 26 bits!
- Digital SLR sensor – 12 bits
- JPEG – 8 bits (values from 0-255)
- Photographic paper – 5 bits

We can make a few observations.
- Most of the dynamic range of the world comes from illumination differences, i.e. the sheer amount of light the sun can pump out.
- Most of the dynamic range of human vision comes from neural adaptation.
- The camera sensor cannot see the full dynamic range of the world at once (by a long shot). We have to choose which 4000:1 out of the 80M:1 to capture on the sensor, a problem addressed by **metering**. The same decision has to be made again when reproducing the image on media, a problem addressed by **tone mapping**.

## Metering
Metering is when a camera uses exposure data to automatically adjust shutter speed and aperture, and correct the exposure.[^manual] Remember, it's about:
- shifting the limited dynamic range of the camera sensor...
- to properly match the portion of the dynamic range of the world...
- that's exhibited in the given scene.

It's assumed that the given scene itself doesn't vary more than the range of the sensor, otherwise special techniques like **exposure bracketing** have to be used.

![greycard.jpeg]{: width="250"}
*An 18% grey card*

Metering is hard because the camera doesn't know what it's looking at semantically. It will instead just assume the world is on average 18% gray, and increase or decrease exposure to match that level. The value 18% gray is called **middle gray** and lies between 0% reflective (black) and 100% reflective (white).

![meteringintro.png]{: width="400"}

The polar bear and gorilla above are not given their "true" colors due to the middle gray baseline.

## Zone System
Recall from [Image Formation]({% post_url 2021-10-20-image-formation %}#the-exposure-triangle) that exposure is irradiance multiplied with exposure time. While exposure time is the same for every element of an image, the irradiance obviously varies. We can then think about each element in the image as having a separate exposure. We want to align the exposure of each element in a photograph with the brightness of its real-world counterpart.

Ansel Adams' zone system offers guidance on just that. There are 11 zones, marked with roman numerals 0-X, and each zone differs by about 1 stop.
- X refers to the brightest white possible from a blank paper.
- V refers to middle gray
- 0 indicates the darkest black reproducible on the paper
- numerals in between are given examples from the real world, for example IX is "snow in sunlight"

The zone system was applied to film photography, but the same idea carries over to digital; we simply plan the tones we want for each part of our scene. One way to visualize the tones of an image is with a **histogram**, which shows the proportion of pixels at each brightness level:

![The-anatomy-of-a-histogram.jpg]{: width="400"}

Dark pictures will have left-leaning histograms, while bright pictures will be right-leaning. For color images, it can be further split into red, green, and blue histograms.

## Metering Modes
The idea of averaging over all pixels and shooting for 18% gray is a bit of an oversimplification. There are several metering modes to choose from.
- **center-weighted**: average with a greater weight on a large center region; suitable for portraits
- **spot**: average over a small, customizable region, usually the focus point; suitable for small objects of interest
- **matrix/evaluative**: the most sophisticated averaging method, encompassing many different techniques, and varying across different cameras (explained further below)
    - the image is divided into zones (the number varies), each zone is processed individually, then recombined
    - may depend on autofocus point, color, focus distance, local contrast
    - may learn from a database of images


![metering-center.jpg]{: width="400"}
*Center-weighted metering*

![metering-spot.jpg]{: width="400"}
*Spot metering*

Let's discuss evaluative mode in more depth. In most SLRs, the evaluative mode uses a low-res sensor looking at the focusing screen. The idea is to use large pixels or regions to get a high dynamic range.[^PNS] It's similar to Adams' zone system in that we're determining the global exposure by combining measurements on individual regions.

![metering1.png]{: width="800"}

Here's an example of low-res images used for metering. What should the exposures be? In particular, should we allow the regions with white pixels to be overexposed?

![metering3.png]{: width="800"}

Based on the full resolution images, we might not care that the white pixels in the fire are overexposed, but we might care about the white pixels on the book.

It's possible to keep a database of low-res images paired with good settings. To meter a new image, we could find the most similar image in the database and fetch the settings. Another option is to train a learning algorithm to do produce optimal settings for a given image.

Future approaches might add face detection, (learned) object recognition, personalization based on shooting history or online image collections, or collaborative metering (inspired by collaborative filtering for recommendations).

## Shooting Modes
The shooting mode (aka camera mode) determines what settings the camera controls based on metering:
- **Auto**: the camera controls aperture, shutter speed, and ISO (and potentially more)
- **Program (P)**: like auto, except you can choose among fixed combinations of aperture and shutter speed.
- **Aperture priority (A)**: auto, except you can change aperture
- **Shutter priority (S)**: auto, except you can change shutter speed
- **Manual (M)**: you can change aperture, shutter speed, and ISO

![modedial.jpg]{: width="400"}
*Typical mode dial on a DSLR*

## Other Exposure Tricks
There's even more ways to get the perfect exposure:
- **exposure compensation**: we can adjust the baseline a few stops higher or lower than 18%
    - this is the technique used on the polar bear and gorilla in the second column above  
- **exposure lock (AE lock)**: freezes the exposure
    - you can obtain the desired exposure by pointing your camera somewhere else, locking the exposure, then recomposing your shot and taking the picture
- **exposure bracketing (HDR imaging)**: useful when the dynamic range of the scene is wider than the camera can capture
    - the same picture is taken multiple times with different settings in order to capture the dynamic range of different regions of the image (e.g. the sky and the foreground)
    - they are combined in post, replacing poorly exposed regions in one shot with the correctly exposed regions in another shot
    - it is popular in phones to have an HDR setting, where the whole process is performed automatically in one press of the shutter button.

# Examples
## Sunlight vs Moonlight
**Q: How many stops do the settings used to take the two images below differ by?**

![sunmoon.png]{: width="800"}
**A:** First note that the left image was taken during the daytime, and the right image was taken at night. Although they look similarly bright in the images, it will be clear from the shooting settings how different the scenes were in terms of illumination.

Let's look at the ratio of each component of the exposure triangle:
- Shutter speed: $$8 : \frac{1}{500} \approx 2^{12}$$ 
- Focal length: $$\frac{1}{1.7} : \frac{1}{6.3} \approx 2^2$$
- ISO: $$400 : 100 = 2^2$$

Recall that one stop of light is given by a factor of $$\sqrt{2}$$ for focal length, but a factor of $$2$$ for shutter speed and ISO. So to get our answer we can add all the exponents above, but double the contribution of the focal length, giving $$12 + 2(2) + 2 = 18$$. The settings used to take the two images differ by 18 stops!

Apparently, the images above are JPEGs, and as a result, their brightness doesn't vary linearly with the brightness of the scene. The JPEG values are related to the RAW values by the following formula:

$$
\text{JPEG value} = (\text{RAW value}/65536)^{1/\gamma} \times 255
$$

where $$\gamma=2.1$$. The result is a logarithmic function which makes all pixels brighter, but more greatly affects the dark pixels:

![jpeg_raw.png]{: width="400"}
*Red is JPEG values as a function of RAW values; Blue is a linear relationship for comparison.*

This is known as **gamma correction** or gamma transformation, and is used because human perception is more sensitive to relative differences in dark tones compared to light tones. As a result, the RAW light tones are *compacted* in the output JPEG range, while dark tones are *expanded*.

We can therefore say the lighter tones, like the snow pixels, differ more in the RAW format, giving maybe a $$2\times$$ boost in variation.

# Unresolved Questions
- Does the use of an autofocus lamp make it active autofocus or is it still considered passive autofocus?
- I'm not sure I believe that the pupil is responsible for such a small fraction of our dynamic range. I'll look into it more.

[^double]: AF point 10 is a double cross-type, and the same point applies.
[^source]: I wish I had a source for the examples besides the slides.
[^manual]: The information is still available to the user in manual mode, even though it won't affect the settings.
[^PNS]: A PNS doesn't have this option so it has to use the main image sensor with smaller, more easily saturated pixels. If it's over- or under-exposed, it won't necessarily know how much to change the exposure, so it'll have to guess and check again.

[unfocused.jpg]: /assets/images/autofocus-and-autoexposure/unfocused.jpg
[autofocus.png]: /assets/images/autofocus-and-autoexposure/autofocus.png
[autofocusapplet.png]: /assets/images/autofocus-and-autoexposure/autofocusapplet.png
[afpoints.jpg]: /assets/images/autofocus-and-autoexposure/afpoints.jpg
[dualtypeautofocus.jpg]: /assets/images/autofocus-and-autoexposure/dualtypeautofocus.jpg
[autofocussensor1.jpg]: /assets/images/autofocus-and-autoexposure/autofocussensor1.jpg
[autofocusapplet2.png]: /assets/images/autofocus-and-autoexposure/autofocusapplet2.png
[overexposed.jpg]: /assets/images/autofocus-and-autoexposure/overexposed.jpg
[greycard.jpeg]: /assets/images/autofocus-and-autoexposure/greycard.jpeg
[meteringintro.png]: /assets/images/autofocus-and-autoexposure/meteringintro.png
[The-anatomy-of-a-histogram.jpg]: /assets/images/autofocus-and-autoexposure/The-anatomy-of-a-histogram.jpg
[metering-center.jpg]: /assets/images/autofocus-and-autoexposure/metering-center.jpg
[metering-spot.jpg]: /assets/images/autofocus-and-autoexposure/metering-spot.jpg
[metering1.png]: /assets/images/autofocus-and-autoexposure/metering1.png
[metering3.png]: /assets/images/autofocus-and-autoexposure/metering3.png
[modedial.jpg]: /assets/images/autofocus-and-autoexposure/modedial.jpg
[sunmoon.png]: /assets/images/autofocus-and-autoexposure/sunmoon.png
[jpeg_raw.png]: /assets/images/autofocus-and-autoexposure/jpeg_raw.png