## Announce who's home using facial recognition

#### Perform facial recognition on a camera feed and announce who's home over text-to-speech. All processing and data capture remain in the home.

# Introduction
In this project I show how I have implemented a system for facial recognition with text-to-speech announcements of who is recognised in my hallway. I use this to announce who is recognised over a speaker and trigger automations which are personalised to the object recognised (typically myself or my wife).
On planning this project I identified several requirements that the project solution must satisfy:

1. All image data from my camera must be stored and processed locally, so that I have complete control over my sensitive personal data. This rules out cloud based solutions such as Google Vision or Amazon Rekognition.
2. My solution must integrate with my home automation platform Home-Assistant, which runs locally in keeping with requirement 1. Head to the Home-Assistant docs for more information.
3. The solution must be fast enough to take action (such as making TTS announcements) within 2 seconds of a recognisable face being presented to the camera. Two seconds is about the length of time it takes to walk down my corridor.
4. The solution must not adversely affect the performance of my home automation platform, either by slowing it down or risking outages due to crashes etc.
5.  It must be straightforward to customise the solution to recognise a reasonable number of specific people, such as myself and my family.
6. Maintenance overheads must be minimal, and those that are necessary must be capable of being automated by Home-Assistant.
I think you will agree these are quite a challenging set of requirements to fulfil, so lets proceed!

# Facial recognition in Home-Assistant
In order to satisfy my requirement that my solution integrates with Home-Assistant, I naturally investigated the facial recognition options presented in the image processing section of the Home-Assistant docs. There are 4 integrations for facial recognition in Home-Assistant. I immediately ruled out the Microsoft integration since this uses a Cloud service, in violation of my requirement that all image data remain on my local network. Both the dlib and openCV integrations looked promising, but both require that I install extra packages, and both must be run on the same machine and in the same environment as Home-Assistant. Since I didn't want the hassle of installing packages, and I did't want to risk any performance degradation on my computer running Home-Assistant, I decided to try out the new Facebox integration.
# Facebox
The Facebox integration to Home-Assistant takes a notably different approach to the dlib and openCV integrations. Facebox is run in a Docker container and is accessed via a rest API. In contrast, dlib and openCV require that dependent packages must be installed in the same computing environment as Home-Assistant (and therefore use the same computing resources). Faceboxes approach using Docker means that there is no need to install and maintain any additional software packages in the Home-Assistant environment. Furthermore, since Facebox is accessed via a local rest API, the computation required for facial recognition can be performed on any computer capable of running Docker, so there is no risks to the performance of my computer running Home-Assistant.
# Hardware & Computing
Camera: Hikvision doorbel. This product permits local image/video storage as opposed to Ring or Doorbird.
Speaker: The announcement speaker is integrated into Home Assistant via Chromecast.  The speaker itself is a JBL Flip, but any speaker with a 3.5mm audio jack will do.
Computing: My entire (home automation) set up is running in this device. Intel® Celeron® Processor J1900 with 8GB of RAM.
I am running Home-Assistant on Ubuntu server 18.04 LTS in a Docker container. However the solution in this project is equally suitable if you are running Home-Assistant on any of the other supported computers such as raspberry pi.
Facebox container runs on the same server.
# Setting up Facebox with Home-Assistant
Head to the Home-Assistant docs for more information.
Free tier limitation: Facebox offers a free of cost tier for non-commercial use. It's main restrictions are  taught faces/tags limited to 20 and revoked State management recording. The faces limitation is null in must home implementations. The State management, however, presents a noticeable challenge. Effectively when the container is re-started, it re-loads "blank" in the absence of a State data file, which means faces/tags need to be re-taught. A potential solution to this limitation is to automate face re-load on restart.
Run the Facebox Docker container with:
sudo docker run -tid --name=facebox --restart=unless-stopped -p 8080:8080 -e MB_KEY=yourkey machinebox/facebox
Customising Facebox
Train Facebox using the script on Github.
# Maintaining Facebox
As mentioned earlier the Facebox free has a limitation worth considering. Namely inability to retain previously loaded faces in the event of docker container restart (Faceprint feature). In order to overcome this limitation, I implemented an automation which reloads the faces utilising the Github script mentioned before. Automation details are described under the "Automating Face Reload" section of the included Code.
