## Announce who's home using facial recognition

#### Perform facial recognition on a camera feed and announce who's home over text-to-speech. All processing and data capture remain in the home.

# Project definition
In this project I show how I have implemented a system for facial recognition with text-to-speech announcements of who is recognised in my hallway. I use this to announce who is recognised over a speaker and trigger automations which are personalised to the person recognised (typically myself or my wife).
# Project requirements
On planning this project I identified several requirements that the project solution must satisfy:
1. All image data from my camera must be stored and processed locally, so that I have complete control over my sensitive personal data. This rules out cloud based solutions such as Google Vision or Amazon Rekognition.
2. My solution must integrate with my home automation platform Home-Assistant, which runs locally in keeping with requirement 1. Head to the Home-Assistant docs for more information.
3. The solution must be fast enough to take action (such as making TTS announcements) within 2 seconds of a recognisable face being presented to the camera. Two seconds is about the length of time it takes to walk down my corridor.
4. The solution must not adversely affect the performance of my home automation platform, either by slowing it down or risking outages due to crashes etc.
5.  It must be straightforward to customise the solution to recognise a reasonable number of specific people, such as myself and my family.
6. Maintenance overheads must be minimal, and those that are necessary must be capable of being automated by Home-Assistant.

I think you will agree these are quite a challenging set of requirements to fulfil, so lets proceed!
# Current status of facial recognition in Home-Assistant
In order to satisfy my requirement that my solution integrates with Home-Assistant, I naturally investigated the facial recognition options presented in the image processing section of the Home-Assistant docs. There are 4 integrations for facial recognition in Home-Assistant. I immediately ruled out the Microsoft integration since this uses a Cloud service, in violation of my requirement that all image data remain on my local network. Both the dlib and openCV integrations looked promising, but both require that I install extra packages, and both must be run on the same machine and in the same environment as Home-Assistant. Since I didn't want the hassle of installing packages, and I did't want to risk any performance degradation on my computer running Home-Assistant, I decided to try out the new Facebox integration, introduced in Home-Assistant release 0.70.
# Introduction to Facebox
The Facebox integration to Home-Assistant takes a notably different approach to the dlib and openCV integrations. Facebox is run in a Docker container and is accessed via a rest API. In contrast, dlib and openCV require that dependent packages must be installed in the same computing environment as Home-Assistant (and therefore use the same computing resources). Faceboxes approach using Docker means that there is no need to install and maintain any additional software packages in the Home-Assistant environment. Furthermore, since Facebox is accessed via a local rest API, the computation required for facial recognition can be performed on any computer capable of running Docker, so there is no risks to the performance of my computer running Home-Assistant.
Free tier limitation: Facebox is a paid product but offers a free tier for non-commercial use. It's main restriction is that you are limited to 20 taught faces, and these must be retaught every time you restart Facebox.
# Teaching Facebox faces
By default Facebox will detect (count) faces in an image an identify a bounding box around them. This is useful in some situations but Facebox becomes really useful when you teach it to recognise specific faces (see the docs). In my case I have taught Facebox both mine and my wifes faces. Facebox can be taught either by uploading images via the web interface exposed by Facebox, or you can use a script such as this official script in GO on this python script on Github. Note that training is only possible when facebox is in recognition mode (i.e. default behaviour of MB_FACEBOX_DISABLE_RECOGNITION=false).
# Using Facebox with Home-Assistant
Head to the Home-Assistant docs for information on configuring Home-Assistant to use Facebox. The Facebox integration must be associated with a camera feed, and in my case the configuration in Home-Assistant is:

```
image_processing:
- platform: facebox
 ip_address: localhost
 port: 8080
 source:
 - entity_id: camera.local_file
```

Run the Facebox Docker container with:
```
sudo docker run -tid --name=facebox --restart=unless-stopped -p 8080:8080 -e MB_KEY=yourkey machinebox/facebox
```
Now start Home-Assistant and all being well you will see...
ADD YOUR IMAGES SHOWING USAGE HERE?
# Optimising resources
Image-classifier components process the image from a camera at a fixed period given by the scan_interval. This leads to excessive computation if the image on the camera hasn't changed (for example if you are using a local file camera to display an image captured by a motion triggered system and this doesn't change often). The default scan_interval is 10 seconds. You can override this by adding to your config scan_interval: 10000 (setting the interval to 10,000 seconds), and then call the scan service when you actually want to process a camera image. So in my setup, I use an automation to call scan when a new image is available.
You can also reduce the time for face detection (counting number of faces only) by setting the environment variable -e MB_FACEBOX_DISABLE_RECOGNITION=true when you run Docker. As the variable name states, this disables facial recognition and in my experience detection time is reduced by 50-75%. Note that the teach endpoint is not available when you disable recognition.
# Maintaining Facebox
As mentioned previously a limitation of the Facebox free tier is that taught faces are lost each time the Facebox container is restarted (on the pair tier faces persist between restarts). Fortunately we can use Home-Assistant to automatically re-teach Facebox our faces after a restart of the Docker container. I implemented an automation which reloads the faces utilising this Github script.
Automation details are described under the "Automating Face Reload" section of the included Code.
# Text-to-speech
Discuss TTS integration
# Automations
Add automations here
In use
Describe how the system performs in practice. Any improvements possible?
# Hardware & Computing
Camera: Hikvision doorbel. This product permits local image/video storage as opposed to Ring or Doorbird.
Speaker: The announcement speaker is integrated into Home Assistant via Chromecast.  The speaker itself is a JBL Flip, but any speaker with a 3.5mm audio jack will do.
Computing: My entire (home automation) set up is running in this device. Intel® Celeron® Processor J1900 with 8GB of RAM.
I am running Home-Assistant on Ubuntu server 18.04 LTS in a Docker container. However the solution in this project is equally suitable if you are running Home-Assistant on any of the other supported computers such as raspberry pi.
Facebox container runs on the same server.
