---
layout: post
title: "Digital Photography Notes - Image Formation"
categories: note
tags: photography, perspective, camera
---

> This note is based on lecture 1 of Levoy's Digital Photography. It covers the basics of photography, including perspective, imaging properties, camera anatomy, and tradeoffs. Beyond the bare minimum, I tried to include intuition about things, and clarify common confusions.
> 
> I expanded the material and rearranged the contents from the original lecture to be a bit more comprehensive. I like how Levoy starts with perspective and basic optics, before diving into all of the camera internals, so I kept that.

<!--more-->

- TOC
{:toc}

# Goals For This Note
This note is based on lecture 1 of Levoy's Digital Photography. It covers the basics of photography, including perspective, imaging properties, camera anatomy, and tradeoffs. Beyond the bare minimum, I tried to include intuition about things, and clarify common confusions.

I expanded the material and rearranged the contents from the original lecture to be a bit more comprehensive. I like how Levoy starts with perspective and basic optics, before diving into all of the camera internals, so I kept that.

# Perspective
First we have two main assumptions:
- Light moves in a **straight line**.
- The lines **converge to a point** at the eye.

There's two versions of perspective:
- **Natural perspective** (Euclid): larger objects subtend smaller visual angles
- **Linear perspective** (Brunelleschi): lines of sight intersect a **visual plane**; larger objects correspond with larger images on the visual plane (aka. image plane, canvas, sensor)
 
![perspectives.png]{: width="500"}
*Left: natural perspective; right: linear perspective* 

Note that the two versions are not equivalent since $$\frac{y_2}{y_1}\neq\frac{\theta_2}{\theta_1}$$. Linear perspective is chosen going forward.

![similartriangles.png]{: width="400"}

Consider a point $$P$$ in the scene and correspondent point $$P'$$ on the intersecting image plane. Mostly ignoring what the variables mean, we can set up an equation using similar triangles:

$$\frac{y}{h}=\frac{f}{Z}$$

We can see that the image height $$y$$ depends *inversely* on the real distance $$z$$. That is, farther objects look smaller on the visual plane.

## Vanishing Points

![alberti.png]{: width="600"}
*Alberti's Method*

We know that lines of light converge at the eye, and the image above illustrates how these lines are represented on the image plane. Take note of the **vanishing point**. The left portion of the diagram shows that points further away land higher on the visual plane. In the limit, infinitely far points will be located at the vanishing point in the image. The right portion shows (not as clearly) parallel lines in the real world converging to a vanishing point on the visual plane. Note that if the parallel lines in the scene are *also* parallel to the image plane, then they *will not* converge to a vanishing point.

![vanishingpoints.png]{: width="600"}

In the image above, the red lines trace out (some) of the lines parallel in the scene which converge to a vanishing point in the image. Notice how, in 1-point perspective (top-left subimage), the horizontal and vertical lines parallel to the image plane don't converge, as stated.

The same image mentions the three well-known modes of perspective, supposedly based on the "number of vanishing points".

![4thvp.png]{: width="300"}

Actually, any pair of parallel lines will converge to a point, so there are technically infinite vanishing points. The reason we usually only consider three modes of perspective is the tendency to picture "blocky" shapes, like buildings.

## Perspective Distortion

![portraitdolly.png]{: width="500"}

Finally, observe the three different pictures taken of the same woman. The left and maybe right images can be considered distorted, while the middle one is normal. What's going on?

![lowangle.png]{: width="600"}

The apparent "distortion" is simply perspective working as usual. The diagram above shows a birds-eye view of a head with ears and a nose. The left side corresponds with the left image of the woman, and the right side corresponds with the right image of the woman. The left image was taken very close to the camera (roughly marked by $$O$$) while the right image was taken far away. **Perspective distortion** is just perspective applied to objects at distances very different from typical viewing distances. I'd say it's the least "distorty" distortion.

Note: The effect on the right is often called *lens compression*, which is dumb because the lens has nothing to do with it. Stick with perspective distortion.

You can quickly replicate this in front of a mirror. Look at your reflection at a normal distance, then move forward as much as possible. Watch your ears and nose specifically.[^accommodation]

## Distortion on the Periphery

![naturalperspectivedistortion.png]{: width="300"}

There appears to be a kind of perspective distortion which concerns objects on the periphery, which may or may not be separate from the previous section. Supposedly, an object viewed from a different viewing distance than the picture was taken will look distorted near the edges. The same topic was brought up in this [article](https://blog.photoshelter.com/2018/06/a-mathematician-weighs-in-on-lens-compression/) but I can't find a name for the phenomenon. I'll update this section if I find out more.

# Directing The Light
![scatter.png]{: width="300"}

Objects scatter light in many angles, and sensors will integrate light from many angles. Without directing the light from the scene to the sensor in some fashion, the whole sensor would end up recording similar colors.

## Early Imaging Devices

![covsdurer.png]{: width="400"}

The image above shows two devices. The bottom one shows light passing through a glass plane, exactly like our earlier construction of an intersecting visual plane. While it works for tracing, it cannot function as a sensor due to the scattering as previously mentioned. The top device is a *camera obscura*, or **pinhole camera**. The light passes through the pinhole, producing a flipped image on the wall behind it. Except for the flip and some scale differences, the images on the glass and the wall are the same.

Benefits of pinhole camera:
- No distortion (except diffraction)
- Infinite depth of field: everything is in focus

![widerpinhole.png]{: width="400"}

If the pinhole is too large, the whole image becomes blurred, as we can see above. It is too small, we experience diffraction effects (blurring again). If we avoid both those problems, the pinhole still has the downside of letting in little light.

![nowlens.png]{: width="400"}

Instead we can use a convex chunk of glass called a **lens**.

## Lenses

![lensoptics.png]{: width="400"}

Observe how a simple lens works in the image above. The **focal length** $$f$$ is:
- a property of the lens
- cannot be changed (for simple lenses)
- signifies where parallel light rays converge

![gausslens1.png]{: width="400"}

Above is an idealized diagram of light passing through a lens. Levoy calls it **Gauss' lens tracing construction**, but everyone else seems to call it a ray diagram.

The focal length is *not* where any given object is in focus. The diagram above shows a lens, the focal length (the distance from the lens to either of the two dots), an **object** (the left arrow), and its flipped **image** (the right arrow). The horizontal distance from the object to the center of the lens is called the **object distance**; the **image distance** is similarly defined.

The diagram also traces the light rays from *a single point* on the object. We can see that the outer rays form a diamond, and the left and right points of the diamond indicate where the object and sensor should be for the object to be in focus. We can actually say that any image point on the same vertical line is in focus (we'd just have to draw lots of diamonds). Since we usually work in three dimensions, the vertical line turns into a vertical plane, called the **focal plane**, which corresponds to the sensor of the camera. The position of the sensor determines what objects in the world are in focus. The distance from the center of the lens to an in-focus plane in the scene is called the **focus distance**.[^working]

**Clarification**: The object and image distances can be used outside the context of the sensor. Usually we also discuss objects forming a focused image at the focal plane, but if not it must be explicitly stated.

The basic relationship between object distance $$s_o$$, image distance $$s_i$$, and focal length $$f$$ is:

$$
\frac{1}{s_o} + \frac{1}{s_i} = \frac{1}{f}
$$

To get a good intuition about the formula, here is a [good website](https://ophysics.com/l12.html) to play with the optics of a simple lens. In particular:
- see what happens to the image when you move the object past twice the focal length (or past $$2f$$).
- when you move the object in between $$2f$$ and $$f$$.
- when you move the object to $$f$$
- when you move the object between $$f$$ and $$0$$.

# Camera Anatomy

Equipped with an intuition about light and lens, we can now learn how an image is captured. Though a camera has many components, we will cover three basic ones: the **lens**, **shutter**, and **sensor**.

## Lens

![lenscross.jpg]{: width="300"}
*Cross section of a lens*

Earlier we made the simplification of a single lens. In fact, a typical camera lens has multiple lens elements mounted on a common axis, forming a **compound lens**. The advantage over a single lens is a reduction of optical abberations. 

Compound lenses can be classified by whether they can change their focal length:
- **zoom**: the lens elements can move to change the focal length of the compound lens
    - **varifocal zoom**: the objects in focus change as the focal length (zoom amount) changes
    - **parfocal zoom**: the objects in focus *don't* change as the focal length changes
- **prime**: the focal length is fixed
 
Prime lenses tend to be smaller, sharper, but more expensive than zoom lenses. Varifocal lenses are more common in photography, while parfocal lenses are used more in cinema and broadcast TV. Remember, we can always change the focus distance, even on prime lenses, by moving the lens closer to or farther from the sensor. 

The lens elements are all contained in a complex housing. In the middle lies the **aperture**, surrounded by about half the lens elements on both sides. It's an adjustable opening that affects how much light *passively* enters the camera body. 

**Clarification**: When we're talking about a camera "lens", we're referring to the whole assembly, including the lens elements, the aperture, and the housing. The lens is sometimes referred to as the "glass". While the term still refers to the whole assembly, it comes from the large amount of glass in the lens due to the lens elements.

![imagecircle.jpg]{: width="300"}
*A simulation of the image circle*

The cross section of the cone of light exiting the lens is called the **image circle**. Until now we've assumed the sensor was inscribed within the image circle, but that's not necessariy true. If the sensor is larger, then **vignetting** can occur in the image, meaning the corners and/or edges of the sensor won't be fully exposed. If the sensor is smaller, then it is underutilizing the light provided by the lens, or **cropping**. Vignetting is usually undesirable, but cropping isn't necessarily bad.

Each lens has a **lens format**, defining the size of the light cone it casts, which determines its sensor compatibility. A lens can be used on sensor sizes less than or equal to the size it was designed for, without vignetting.

In summary, the lens has three independent, distinguishing features:
- focal length range
- aperture range
- lens format

## Shutter

 The shutter is crucial for letting *precise* amounts of light hit the sensor. It begins closed, letting *no* light hit the sensor, and only opens briefly when a picture is taken. A shutter cycle (open, close, and reset) is often called an **exposure** (a term which also refers to the amount of light hitting the sensor *during* the exposure; see the section Exposure Triangle). The **shutter speed** is an adjustable property of the shutter determining the speed of the exposure.

Shutters can be classified in several ways:
- **mechanical**: a physical mechanism in front of the sensor to block light
    - **leaf (or diaphragm) shutter**: a series of blades in a circle which open and close, located in the lens
    - **focal-plane shutter**: one or two "curtains" which open and close; located just in front of the sensor
- **electronic**: implemented in the sensor electronically
    - **global shutter**: the entire sensor is exposed at once
    - **rolling shutter**: the sensor is exposed line by line

![2shutter.jpg]{: width="400"}
*Left: leaf shutter; right: focal-plane shutter*

Note: a leaf shutter looks kind of like an aperture but it's not the same!

Leaf shutters are quiet and relatively slow, while focal-plane shutters are loud and fast, and can cause vibrations during the shot. Electronic shutters can be even faster than focal-plane shutters, without any noise or vibrations. Rolling shutters are pretty common on consumer cameras, while global shutters are mostly limited to small resolutions or high-end cameras. Finally, rolling shutters are susceptible to **rolling shutter distortion**, also called the "jello effect".

## Sensors

Light is directed by the lens onto either a photographic film or a digital sensor. We'll limit the discussion in this section to the latter.

First, there are two main types of imaging sensors:
- **Charge-coupled device (CCD)**
- **Complementary metal–oxide–semiconductor (CMOS)**

CMOS sensors are almost ubiquitous in digital consumer cameras, while CCDs have been relegated to niches like scientific imaging or astrophotography.

![sensorsizes.jpg]{: width="500"}

The **sensor format** or sensor size, captures the dimensions of the sensor in milimeters. The image above shows many common digital sensor sizes. The 35mm format is called a **full-frame** sensor.

**Terminology Rant**: Sensor formats have names like Pentax 645D or Micro Four Thirds. These usually have *nothing* to do with the sensor dimensions they represent. The 35 mm format is not 35 mm in width, height, or diagonal, and gets its name from 35 mm film. The 1-inch format is closer to half an inch in width, and gets its name from television camera tubes.

A sensor is a big grid of **pixels** or **photosites**, and each pixel is like a bucket for photons. When a picture is taken, light fills the buckets for the desired period of time. The data read off the sensor corresponds to the amount of light gathered in each pixel.

The number of pixels on a sensor, or the **resolution**, is measured in megapixels or MP, which stands for a million pixels. The resolution can also be represented by an ordered pair of pixels along the width and height. The ratio of pixels along the width vs height is called the **aspect ratio**. For example, 24 MP is equivalent to 6000 x 4000 pixels, which gives an aspect ratio of 3:2.[^aspectratio]

The same lens can give different aspect ratios via cropping. For example, the 3:2 sensor can be cropped to 16:9. Since pixels are cut off, the resolution decreases.

![aspectratio1.png]{: width="300"}
*Different aspect ratios inscribed on the image circle*

Finally, the pixels themselves can be different sizes, and are measured in microns (μm). Two sensors can have the same size but differing resolutions if the pixel sizes are different.

In summary, the three independent, distinguishing features of a sensor are:
- sensor format
- resolution
- pixel size

# Types of Photographic Cameras

Here is a collection of different kinds of cameras used for taking still pictures, presented in roughly chronological order by the time they were created. We will see how the components from the previous section can be combined.

## View Camera

![viewcamera.png]{: width="300"}

A view camera is composed of two parts, a front and rear standard, connected by a pleated box. Modern view cameras are sometimes called *large-format* cameras, referring to their large sensors. Though the basic design is fairly old, dating to the nineteenth century, it offers the greatest freedom of positioning for the lens relative to the sensor, which is why it still finds use today.

The light passes through the lens and aperture (not pictured), casting an inverted image on a glass pane in the back. Once the image is composed, the shutter is closed, the glass is manually replaced with film, and the shutter is triggered, exposing a photographic film. The film is a light-sensitive material used to capture the image.

View cameras use a leaf shutter, located in the middle of the lens.

## Single-Lens Reflex (SLR) Camera

![slr.png]{: width="300"}

The basic SLR design is shared by most modern cameras.

Here it's easier to see that the lens (not labeled) is really a complex assembly of lenses, with an aperture in the middle. Instead of viewing the image on a glass pane, we now have a viewfinder. The light normally passes through the lens, bounces off a mirror, passes through a pentaprism or pentamirror above (not labeled), and exits the viewfinder. Finally, notice how the shutter is located right in front of the film, rather than inside the lens.

When a picture is taken, the mirror flips up, allowing light to pass to the shutter, and the shutter opens briefly, exposing the film. Finally the shutter closes and the mirror flips back down. The mirror movement makes SLRs louder than other types of cameras.

SLRs use focal-plane shutters. A digital SLR or DSLR just replaces the film with a digital sensor.

## Point-and-Shoot (PNS) Camera

![pns.jpg]{: width="400"}

PNS or compact cameras are typically cheap, small, and useful for taking quick photos. Unlike on (D)SLRs and mirrorless cameras, the lenses are usually non-interchangeable. Mobile phones have mostly pushed PNS cameras out of the market.

Since the light in the viewfinder takes a different path from the the light hitting the film, the viewfinder isn't a completely accurate representation of the final image.

There are digital versions, replacing the film with a digital sensor. Unlike the previous two camera types, most PNS cameras use electronic shutters.

## Phone Camera

![phonecamera.jpg]{: width="400"}

Mobile phones come with one or more small cameras. These cameras have electronic shutters, lenses with fixed focus and tiny focal lengths, and small sensors with small pixels and medium-low resolutions. A recent trend has been to include multiple cameras with different focal length lenses. Instead of optical zoom, most phone cameras use digital zoom, meaning the image is cropped and enlarged *after* exposure. The phone screen contains all the controls and serves as the viewfinder.

## Mirrorless Camera

![mirrorless.jpg]{: width="500"}

A mirrorless camera is very similar to a DSLR, but it ditches the mirror and optical viewfinder. The new electronic viewfinder relies on the light hitting the image sensor in real-time, meaning the shutter must be open in its default state.

Like (D)SLRs, mirrorless cameras primarily use mechanical shutters. When a picture is taken, the shutter closes in preparation, then opens briefly, closes again, then opens to make the electronic viewfinder available again. Some mirrorless cameras implement hybrid electronic-mechanical shutters to reduce vibrations. For both DSLRs and mirrorless cameras, a live electronic viewfinder with lower resolution will utilize an electronic shutter. A purely electronic shutter doesn't have fast enough readout speeds for the full resolution, however, and would produce a strong jello effect.

The term "mirrorless camera" always refers to a digital camera.

# Lens and Sensor Interactions

For this section, let's limit the settings we have direct control over to the following:
- focal length
- sensor size
- image distance
- object distance

By manipulating those, we can predictably affect properties of our image like:
- focus distance
- field of view
- perspective distortion

## Changing Focus Distance

![changefocus.png]{: width="300"}

To focus on different things, we change the distance of the sensor relative to the lens (in cameras, it is actually the lens that moves). That is:
- by *fixing* the focal length...
- and *changing* the distance between lens and sensor...
- we are *changing* the focus distance

Looking at the top lens diagram, we put the sensor at the focal length. This gives us parallel lines on the left, which will only converge *at infinity*. In other words, if we place the sensor right at the focal length, only objects infinitely far away will be perfectly in focus.

The **minimum focus distance**[^mfd] represents the closest distance at which a lens can focus, based on the the maximum distance the sensor can move away from the lens. Though it will likely be longer, it's physically limited by the focal length, since the sensor would have to be placed at infinity.

## Maintaining Focus Distance

![tree.png]{: width="400"}

The diagram above[^focallength] illustrates lenses with different focal lengths. **Stronger lenses**, so called because they bend light more strongly, have smaller focal lengths. **Weaker lenses** have longer focal lengths. We can also say:
- if we want to *fix* the focus distance...
- and *change* the focal length (changing the lens)...
- we need to *change* the distance between lens and sensor

## Changing Field of View 

The extent of the world captured in the image is the **field of view (FOV)**, and is represented as an angle. There's two ways to change the field of view:
- change the focal length
- change the sensor size

![sensorfov1.png]{: width="400"}

In the original tree diagram, we ignored the sensor size. Now we show a fixed sensor size, shown in red. Notice how a longer focal length corresponds with a lower fov, which makes sense since a weaker lens can bend less light to hit the same sensor. 
 
![focallengthfov.png]{: width="600"}

The image above offers an even better visualization of the same relationship.

![sensorfov.png]{: width="400"}

The other way to change FOV is to change the sensor size. This version of the diagram includes a red and a yellow sensor, with corresponding fovs. Obviously, if the sensor is smaller, it will capture a smaller angle of the world.

Both relationships are captured in the following equation, derivable from simple trigonometry:

$$FOV = 2\arctan\left(\frac{h}{2f}\right)$$

where $$h$$ is the height of the sensor and $$f$$ is the focal length.

## Maintaining Field of View

Sensor size and focal length should increase proportionally to keep the same FOV. This principle is the basis for a popular way of comparing lens-sensor systems, described below.

Sensor sizes can be compared using a **crop factor**, which can be expressed as a formula:

$$\text{crop factor} = \frac{\text{diagonal of full-frame sensor}}{\text{diagonal of this sensor}}$$

A full-frame sensor has a crop factor of 1, and serves as a reference format to which other sizes are compared. For example, the smaller micro 4/3 sensor has a crop factor of ~2. Besides facilitating sensor comparison, the crop factor is used to convert true focal lengths to a focal length which would produce the same FOV on a full-frame sensor.

Let's say we're using a 50 mm lens with a micro 4/3 sensor, which has a crop factor of 2.0. To "erase" the cropping effect due to the sensor, we can multiply the focal length by the crop factor. So a 50 mm micro 4/3 lens is equivalent to a 75 mm full-frame lens.

## Dolly Zoom

If we change the focal length and the position of the camera (object distance) in the right way, we can get a nice effect called the **dolly zoom**. 

![dolly1.png]{: width="150"}

Here's a picture of the Dolly Zoom in action. The camera moves away from the subject, while increasing focal length. The key is to combine these in such a way that the size of the subject in the image remains *fixed*.

![dolly.png]{: width="600"}

Here's a sample result, starting from the left and ending with the right. Notice how we can only fix the size of objects in one plane parallel to the sensor. The woman stays the same size, but the apparent size of the man in the back (in a further plane) changes. In other words, perspective relationships are *not preserved*.

Note: the same effect was used to capture the pictues of the woman in the Perspective Distortion section.

**Clarification**: Both focal length and object distance contribute to keeping the woman the same size. However, it can be wrongly concluded that a higher focal length produces more lens compression. As we established earlier, changes in perspective relationships are due to changing object distance. We just happen to place the camera closer to the subjects when using a wide-angle (low focal length) lens.

# The Exposure Triangle

From now on we assume we're working with digital sensors instead of film, since the differences are minor.

As we saw, the captured image depends on the amount of light hitting the sensor. This quantity is known as the **exposure** $$H$$, and is related to **irradiance** $$E$$ and **exposure time** $$T$$ like so:

$$H=E\times T$$

- Irradiance, controlled by aperture, is the amount of light falling on a unit area of the sensor per second.
- Exposure time, controlled by the shutter speed, is the time in seconds in which the sensor is exposed.

Finally, *after* the light hits the sensor, the brightness of the final image can be increased electronically via a parameter called the **ISO**.

Aperture, shutter speed, and ISO are closely connected, forming the **exposure triangle**.

**Terminology Rant**: The term exposure is confusing. First, as stated earlier, it can refer to a single shutter cycle or the quantity of light hitting the sensor during that shutter cycle. Second, ISO *doesn't affect exposure at all*, but people still include it in the so-called "exposure triangle". Any triangle including ISO should be about brightness, but the "brightness triangle" hasn't caught on yet...

## The Key Takeaway!!

![exptriangle.jpg]{: width="400"}

The image above shows different scales for each component of the triangle. Each component has predictable effects on the brightness of the resulting image, as we'll see in the coming sections, and we can standardize the scales of the components against the change in brightness. What this means is that, for example, one step, or "stop", on the ISO scale *has the same relative effect on brightness* as one step on the aperture scale.

## Shutter Speed

The shutter speed or exposure time $$T$$, measured in fractions of a second, is the time the shutter is open. It has a *linear effect* on the exposure, until the sensor saturates (it cannot integrate any more light). Below are sample shutter speeds, and common uses:
- 1/2000: birds in flight
- 1/1000: sports
- 1/250: portraits
- 1/125: landscapes
- 5: blurring water
- 20: astrophotography

**Terminology Rant**: Why is shutter *speed* measured in seconds? Why is it not called shutter *time*? Now we're stuck with higher shutter speeds having smaller units, like 1/2000, and lower shutter speeds having larger units, like 1/125. (...this is just the beginning – wait until we get to aperture)

Humans can hand-hold down to 1/60. There is sometimes an additional mode called bulb (often symbolized as B), which allows one to keep the shutter open manually as long as desired.

The shutter speed abstracts away many details about the shutter itself. In particular, it applies to mechanical and electronic shutters.

### Common Stops

![shutterstop.png]{: width="500"}

The shutter speeds increase regularly by $$\times 2$$, increasing brightness by $$\times 2$$.[^stopconspiracy]

### Side Effect: Motion Blur

**Motion blur** is self-explanatory. It is usually treated qualitatively.

![motionblur.png]{: width="500"}

Shutter speed is *directly proportional* to **motion blur**.

{% comment %}
As a rule of thumb, the longest exposure should be 1/$$f$$, where $$f$$ is the 35mm-equivalent focal length.
{% endcomment %}

## Aperture

Recall the aperture is a hole in the lens that determines how much light enters the body, or the irradiance $$E$$. The irradiance is proportional to:
1. the *square* of the aperture diameter $$A$$
2. the *inverse square* of the distance between the aperture and the sensor (~ the focal length $$f$$)

(1) is because the diameter is related to the area of a circle as $$\text{area}=\pi(\frac{A}{2})^2$$. If the diameter is $$\times 2$$, the light coming through the circular aperture is $$\times 4$$.

![irradiancedistance.png]{: width="300"}

(2) can be visualized using a cone of light coming from the lens (see the image above). If the distance to the sensor increases $$\times 2$$, by similar triangles, the diameter of the circular base of the cone increases $$\times 2$$. From the same principle as (1), the area of the base of the cone increases $$\times 4$$. Assuming the light exiting the aperture is the same amount, by conservation of energy we can say the amount of light hitting the base of the cone, or the irradiance, decreases $$\times 4$$.

In order for the aperture values to capture both (1) and (2), a unitless quantity called the **f-number** or f-stop is defined as:

$$
N=\frac{f}{A}
$$

![aperturesize1.png]{: width="400"}
*Two lenses with the same aperture N*

We're usually interested in the aperture diameter relative to whatever focal length we're using, so it is customary to write the f-number as a fraction $$f/N$$. For example, referring to the image above:
- $$f$$/2.0 on a 50mm lens means the aperture diameter is 25mm
- $$f$$/2.0 on a 100mm lens means the aperture diameter is 50mm

Following the pattern, it's clear that a telephoto lens (large focal length) with a small f-number requires fat lenses.

**Terminology Rant**: The notation is complete trash! When you change the aperture, you are only controlling the diameter, and not the distance to the sensor. However, the aperture is usually expressed as an f-number instead of the diameter. The f-number is technically $$N$$, but is almost always expressed as $$f/N$$, for some value of $$N$$. The "$$f/$$" is pure notation for the f-number, like a "\$" represents dollars. What is supposedly clever and cool about this notation is that, from the formula for the f-number, $$f/N$$ actually produces the aperture diameter $$A$$. So when we're talking about "aperture", we really mean a ratio of focal length to aperture diameter, which we write to look like the aperture diameter.

It gets worse when we're communicating aperture: consider a) $$f/2.0$$ and b) $$f/2.8$$. We say:
- (a) and (b) are called apertures *or* f-numbers interchangeably
- (a) is a smaller f-number than (b)
- (a) is a higher aperture than (b) 

To summarize, when stating f-numbers or apertures ("set the f-number to ..."), we use the *notation* $$f/N$$, but when comparing f-numbers ("this f-number is higher/lower than that f-number" or "increase/decrease the f-number") we use $$N$$, BUT when comparing apertures, we use the *fraction* $$f/N$$. *Capiche*?

![aperture1.png]{: width="300"}

**Clarification**: It might seem that a smaller aperture would decrease the image circle by "blocking" the outer edge, but it does not. Notice how some rays from any point on the object will always reach the sensor. The edge of the image circle may actually get sharper.

How does the f-number affect brightness? From the formula, if $$N$$ increases $$\times 2$$, then $$A$$ is reduced $$\times 2$$, which by (1) reduces the light by $$\times 4$$.
- $$f$$/2.0 to $$f$$/4.0 reduces light by $$\times 4$$, as described
- To reduce light by $$\times 2$$, increase $$N$$ by $$\times \sqrt{2}$$

### Common Stops

![aperturestop.png]{: width="500"}

The f-numbers increase regularly by $$\times \sqrt{2}$$, decreasing brightness by $$\times 2$$.

### Side-Effect: Depth of Field

The **depth of field** is the range of acceptable focus. Smaller depth of fields produce more blurring of the background and foreground. Like motion blur, it is usually treated qualitatively.

![depthoffield.png]{: width="500"}
*Left: low depth of field; right: high depth of field*

The aperture diameter is *inversely proportional* to the depth of field.
- Increasing $$N$$ by $$\times 2$$ increases depth of field by $$\times 2$$.
- Changing $$N$$ or $$f$$ may not have any effect if the other compensates.

![depthoffield1.png]{: width="300"}

Here's a simplified explanation of how depth of field works:
- As we saw in the section "Changing Focus Distance", there is always a point in the scene (or, really, a plane) in focus on the sensor.
- We can visualize this as a series of light cones (*top subimage*).
- Moving the sensor causes a defocusing or blurring effect, equivalent to moving the point in the scene.
- The blurring must be smaller than our allowable **circle of confusion** (*middle subimage*)
- The circle of confusion corresponds with the **depth of focus**, which corresponds with the depth of field.
- Notice how decreasing the aperture  $$\times 2$$ increases the depth of field $$\times 2$$ (*bottom subimage*)

## ISO

Whereas shutter speed and aperture are physical mechanisms, the ISO is a gain parameter. The ISO multiplies the analog signal *after* light integration but *before* conversion to a digital signal. It ultimately boosts the brightness of the final image.

Note that the ISO is different from raising exposure in post-processing (e.g. Photoshop), since the latter boosts the digital signal. Some cameras offer "expanded ISO", which just boosts the digital signal.

### Common Stops

![isostop.png]{: width="500"}

The ISO increases regularly by $$\times 2$$, increasing brightness by $$\times 2$$.

### Side Effect: Noise

![iso.png]{: width="500"}

The greater the ISO, the noisier the image. The picture above shows increasing ISO while keeping brightness constant via other means, highlighting the increasing noise.

## Exposure Triangle, Again

![talvala.jpg]{: width="400"}

The green blobs represent changes to component(s) on the exposure triangle, while yellow blobs represent changes in brightness or side effects. Each green blob affects *two* adjacent yellow blobs simultaneously. If we move the same number of stops for two components of the triangle at once, we can keep the brightness constant.

Here's an interactive [applet](https://sites.google.com/site/marclevoylectures/applets/variables-that-affect-exposure) demonstrating these tradeoffs.

# Exercises

**Q: Does an $$f/2$$ cell phone lens gather as much light from each patch of the scene as an $$f/2$$ SLR lens?**

**A:** A phone camera lens has a smaller focal length because the sensor is small, which allows the FOV to stay about the same as an SLR (also to fit the form factor of a phone ). With $$f$$ and $$A$$ both smaller, the f-number ($$N=\frac{f}{A}$$) stays constant.

Another way to think about it is that the smaller phone lens gathers less light, but it is a stronger lens (smaller focal length), meaning it concentrates the light more, so the amount of light *per unit area* remains the same.

Now, a phone lens has smaller pixels to keep the resolution, or pixel count, the same (or similar). The same number of pixels are exposed, but each captures fewer photons, so the result is noisier.


[^working]: There is another term called working distance, which measures the distance from the front of the lens to the subject.
[^mfd]: Confusingly, minimum focus distance is measured from the closest object to the *sensor*, not the lens.
[^focallength]: Here we have a tree whose distance from the lens is much larger than the focal length $$f$$. It's not super clear in the diagram but the tree forms an image *just past* $$f$$ (not *at* $$f$$, since it's not infinitely far away).
[^aspectratio]: Mathematically one resolution in MP can be satisfied by multiple aspect ratios, and I'm not sure if there's some kind of standardization so we know which aspect ratio we mean by a given resolution.
[^accommodation]: You might object that the experiment is ruined by various factors, including acommodation of the eyeball. The thing is, no matter how much you focus or defocus, your nose will still look fat and your ears will still be covered by the sides of your head.
[^stopconspiracy]: The ticks on the shutter speed scale don't *exactly* progress by multiples of 2. Some are rounded off since the error is inconsequential for such small fractions and it makes mental math easier. [SO](https://photo.stackexchange.com/questions/49860/is-there-a-sane-reason-why-%C2%B9%E2%81%84%E2%82%81%E2%82%82%E2%82%85-is-not-instead-exactly-half-of-%C2%B9%E2%81%84%E2%82%86%E2%82%80)

[perspectives.png]: /assets/images/image-formation/perspectives.png
[similartriangles.png]: /assets/images/image-formation/similartriangles.png
[alberti.png]: /assets/images/image-formation/alberti.png
[vanishingpoints.png]: /assets/images/image-formation/vanishingpoints.png
[4thvp.png]: /assets/images/image-formation/4thvp.png
[portraitdolly.png]: /assets/images/image-formation/portraitdolly.png
[lowangle.png]: /assets/images/image-formation/lowangle.png
[naturalperspectivedistortion.png]: /assets/images/image-formation/naturalperspectivedistortion.png
[scatter.png]: /assets/images/image-formation/scatter.png
[covsdurer.png]: /assets/images/image-formation/covsdurer.png
[widerpinhole.png]: /assets/images/image-formation/widerpinhole.png
[nowlens.png]: /assets/images/image-formation/nowlens.png
[lensoptics.png]: /assets/images/image-formation/lensoptics.png
[gausslens1.png]: /assets/images/image-formation/gausslens1.png
[lenscross.jpg]: /assets/images/image-formation/lenscross.jpg
[imagecircle.jpg]: /assets/images/image-formation/imagecircle.jpg
[2shutter.jpg]: /assets/images/image-formation/2shutter.jpg
[sensorsizes.jpg]: /assets/images/image-formation/sensorsizes.jpg
[aspectratio1.png]: /assets/images/image-formation/aspectratio1.png
[viewcamera.png]: /assets/images/image-formation/viewcamera.png
[slr.png]: /assets/images/image-formation/slr.png
[pns.jpg]: /assets/images/image-formation/pns.jpg
[phonecamera.jpg]: /assets/images/image-formation/phonecamera.jpg
[mirrorless.jpg]: /assets/images/image-formation/mirrorless.jpg
[changefocus.png]: /assets/images/image-formation/changefocus.png
[tree.png]: /assets/images/image-formation/tree.png
[sensorfov1.png]: /assets/images/image-formation/sensorfov1.png
[focallengthfov.png]: /assets/images/image-formation/focallengthfov.png
[sensorfov.png]: /assets/images/image-formation/sensorfov.png
[dolly1.png]: /assets/images/image-formation/dolly1.png
[dolly.png]: /assets/images/image-formation/dolly.png
[exptriangle.jpg]: /assets/images/image-formation/exptriangle.jpg
[shutterstop.png]: /assets/images/image-formation/shutterstop.png
[motionblur.png]: /assets/images/image-formation/motionblur.png
[irradiancedistance.png]: /assets/images/image-formation/irradiancedistance.png
[aperturesize1.png]: /assets/images/image-formation/aperturesize1.png
[aperture1.png]: /assets/images/image-formation/aperture1.png
[aperturestop.png]: /assets/images/image-formation/aperturestop.png
[depthoffield.png]: /assets/images/image-formation/depthoffield.png
[depthoffield1.png]: /assets/images/image-formation/depthoffield1.png
[isostop.png]: /assets/images/image-formation/isostop.png
[iso.png]: /assets/images/image-formation/iso.png
[talvala.jpg]: /assets/images/image-formation/talvala.jpg