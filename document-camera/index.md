# Lockdown setup - Working an IPEVO document camera


As most of us, we're working from home for some time. Here is a quick start guide for the [IPEVO V4K](https://www.ipevo.com/products/v4k) document camera.

## Installation

### Hardware

The camera itself is a plug-and-play device: it's recognized as a camera. Two buttons on it to enable/disable the autofocus and adjust the exposure.

### Software

Go to https://www.ipevo.com/software/visualizer#download

As you can see, the IPEVO visualizer is supported by several platforms:

![img1](./img/cam-img1.jpg)

In the following parts, we'll try to work with the Linux binary and the Chrome plugin

## Working with the Chrome plugin

Maybe the easiest solution: there's a plugin for Google Chrome at https://chrome.google.com/webstore/detail/ipevo-visualizer/knibbpmgfkaginjkddpmmpcnknegecok

![img2](./img/cam-img2.jpg)

You can explore the toolbar on the left side to get options on the resolution, camera rotation and so on. You can even draw with basics color on it.

## Working with the Linux binary

There's a DEB package for Debian-based distros. It's a bit "ugly" as it copies a JAR file in `/usr/bin/`. In other words, to run the Linux binary:

```bash
java -jar /usr/bin/visualizer-1.1.3.37.jar 
```

The interface is similar to the Google Chrome plugin. Some additional options for contrast, while balance, etc.

## Including the camera stream in a Microsoft Teams meeting

https://s3-us-west-1.amazonaws.com/files.ipevo.com/download/doc/step_by_step_guide-microsoft_teams.pdf

![img3](./img/cam-img3.jpg)

By default, the camera stream is horizontally flipped. Even if there's an official guide from Ipevo, the only solution is to share the camera software as there's no option to flip the camera in Microsoft Teams... 

![img4](./img/cam-img4.jpg)

## Including the camera stream in a Discord meeting

Solution is similar to Microsoft Teams. However, Discord is a less buggy software on Linux for group meetings compared to Microsoft Teams. With [OBS studio](https://obsproject.com/), we can easily get a cool setup with slides, document camera and even the presenter face if needed :blush:

![img5](./img/cam-img5.jpg)

Lockdown setup with the document camera on the right side over a book and a cutting mat:

![img6](./img/cam-img6.jpg)

## References

- [IPEVO guides](https://www.ipevo.com/products/v4k/support)
- [IPEVO V4K product page](https://www.ipevo.com/products/v4k/)
