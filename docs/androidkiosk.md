# Android Kiosk Design and Build Notes

Chris Greenhalgh, 2013-09-10

## Overview

The goal is to create a single-purpose "kiosk" using an Android tablet. The initial development target is a Google Nexus 7 (2012, WiFi only, that being the only version I happen to have at the moment).

The implementation strategy is to develop a tailored variant of the CyanogenMod firmware which incorporates all the required modifications.

### Example Scenario

Alex is visiting a small tourist information office in the village of Ruddington. The tablet is fixed to a stand at the side of the room. A periodic animation attracts her attention and invites her to touch the screen to find out about the local area. She approaches the device and touches it, and screen switches to a scrollable grid of digital leaflets, each with a thumbnail image and short title. She taps on one and it fills the screen; she swipes through it, scanning the text and images - it describes a short heritage walk around the village, including a coffee shop. A prompt in the corner of the screen says "Get this leaflet"; she taps this and is presented with options: view on smart phone, send to regular phone, print. She has an Android phone with here and selects smart phone; the screen prompts her to join the Wifi network "Leaflets about Ruddington"; when she does it prompts her to open web browser or scan the QRCode shown. She scans the code and opens her phone's browser, which now shows further options for viewing the leaflet on her phone. She is prompted to first install the Placebooks app from the tablet's WiFi network, which she does. She then downloads the leaflet (Placebook) over the tablet's WiFi network and it opens in the Placebooks app. She checks the start of the tour on her phone and walks out of the office (leaving the device's WiFi network). 

Other deployment possibilities:

*	Local government office or reception area, e.g. parish council.
*	Local library.
*	Village hall or church entrance area.
*	Tourist attraction entrance area, exhibition area or shop.
*	School or college reception area. 
*	Small business reception or waiting area (e.g. hairdressers, beautician).

### Agents / Roles

*	Overall vision - responsible for overall goals of the kiosk deployment, which may be part of a much larger system and/or agenda. May be a committee with members having different (but hopefully compatible) aims.
*	Overall implementation - responsible for "making it happen" and ongoing operation and accounting.
*	Site policy - responsible for decision-making concerning site-specific issues, e.g. constraits on content or use. May be involved in vision setting, or may simply be providing a site for the device, perhaps on a commercial basis. 
*	Site staff - working/present on-site, and potentially able to provide some monitoring and support/facilitation on-site with regard to the device and/or visitors (but typically with some other primary role).
*	Visitor/user - engaging with the device and its services directly.
*	Technology provider - responsible for technical aspects, e.g. device, software, maintenance, installation, perhaps on a commercial basis. 

## Requirements 

Essential:

*	Act as WiFi hotspot for access to on-device content.
*	Host a web server (initially J2EE, to support Placebooks).
*	Host a full-screen kiosk view (app).
*	Prevent cusual users from leaving kiosk app, e.g. preventing access to Home screen.
*	Operate with no Internet access.

Desirable:

*	Host an OSM map server (for Placebooks without Internet connectivity).
*	Boot into functional configuration.
*	Custom time-out and "lock" screen tailored for kiosk use.
*	Support simple monitoring, e.g. obtaining usage information (e.g. via USB flash drive).
*	Support simple configuration, e.g. via USB flash drive.

Optional:

*	Custom branded boot animation.
*	Tailored device status reporting for local support, e.g. power problems (e.g. via boot screen, lock screen, kiosk app).

For any variant with Internet access additional requirements are:

*	No or managed access to Internet connection from other local devices.
*	Remote monitoring support via Internet connection.
*	Remote configuration support via Internet connection.

## Implementation

### Hardware

Initial hardware selection is Google Nexus 7 (WiFi only, 2012). Selection criteria include:

*	[CyanogenMod port exists](http://wiki.cyanogenmod.org/w/Devices) 
*	Supports USB On The Go (i.e. USB Host mode via dual mode socket), e.g. checked with [USB Host Diagnostics App](https://play.google.com/store/apps/details?id=eu.chainfire.usbhostdiagnostics&hl=en).
*	Is capable of running WiFi HotSpot (although this is not supported by default), e.g. checked with [Hotspot Control App](https://play.google.com/store/apps/details?id=eu.chainfire.hotspotcontrol&hl=en)
*	Bluetooth support.
*	Has a screen big enough for (small) kiosk use.
*	Relatively inexpensive and widely available.
*	Reasonably large internal memory (but unfortunately doesn't have an SD Card slot).
*	GSM option.

The device name is "grouper" and the vendor is "asus" ("tilapia" is the device name for the version with GSM, which is directly based on grouper). 

### Firmware

Selected [CyanogenMod](http://www.cyanogenmod.org/) as the base-line firmware, as it seems to be widely used and supports quite a wide range of devices. Using a custom firmware will hopefully allow us to (a) do everything that we want to in a reasonably clean and consistent way and (b) make the end result easier to reproduce (at least in the number of steps required, i.e. install a new custom firmware).

Building of CyanogenMod (at least for Nexus 7 and version 10.1/Android 4.2.2) is quite well described in the [CyanogenMod wiki](http://wiki.cyanogenmod.org/w/Build_for_grouper). Note that you are supposed to have the same (base) version of CyanogenMod (i.e. pre-built from the web) installed on the device before completing the build. This, of course, will require that you unlock the device as per the usual instrustions (i.e. for the Nexus 7, using the Android SDK tool "fastboot oem unlock"), install a custom recovery (typically ClockWorkMod recovery) and install CM. Unlocking and quite likely also installing CM will result in or require a factory reset, so best done on a new/clean device (and always make a backup...).

### WiFi Hotspot Support

WiFi hotspot support (a.k.a. Wifi tethering in the code) is disabled by default on Nexus 7. The availability of this option in the Settings is determined by device/asus/grouper/overlay/frameworks/base/core/res/res/values/config.xml, in particular:

	<string-array name="config_tether_wifi_regexs">
		<item>wlan0</item>
	</string-array> 

	<integer-array name="config_tether_upstream_types">
		<item>0</item>
		<item>1</item>
		<item>7</item>
	</integer-array>

This assumes (a) the WiFi interface name is "wlan0" and (b) these are suitable tether types (0=mobile, 1=WiFi, 7=Bluetooth - these are CommunicationManager.TYPE_... values).

Status: works on my nexus 7, i.e. exposes hotspot settings and when enabled another device can connect and communicate. 

Note: hotspot is NOT automatically re-enabled after reboot, even if it was running before.

### Web server

The first test service is [Placebooks](https://github.com/horizon-institute/Bridging-the-Rural-Divide--Placebooks) which is based on a J2EE web service. So the web server being tried is Jetty, more specifically the Android Jetty port/hosting app [i-Jetty](http://code.google.com/p/i-jetty/), which is open source.

The i-Jetty [Build Instructions](http://code.google.com/p/i-jetty/wiki/BuildInstructions) are mostly correct, but it seems to be necessary to edit i-jetty/i-jetty-ui/pom.xml and change it to use a more recent android-maven-plugin, i.e. change:

       <plugin>
          <groupId>com.jayway.maven.plugins.android.generation2</groupId>
          <artifactId>maven-android-plugin</artifactId>
          <version>2.9.0-beta-5</version>
	... 

to 

       <plugin>
          <groupId>com.jayway.maven.plugins.android.generation2</groupId>
          <artifactId>android-maven-plugin</artifactId>
          <version>3.6.1</version>
	... 

i-Jetty does not have a configuration option to start at boot, so this will probably need to be added, e.g. a configuration option to start on the booted broadcast intent.

### Placebooks web service

[Placebooks](https://github.com/horizon-institute/Bridging-the-Rural-Divide--Placebooks) is based on a J2EE web service, the source of which is in the placebooks-webapp subdirectory. 

Here are some build notes...
*	There is an ant build.xml file. 
*	On Ubuntu with open-jdk JAVA_HOME will need to set to something like /usr/lib/jvm/java-1.7.0-openjdk-amd64 before ant will work.  
*	A build.properties file must be created, which apparently needs to specify catalina.home, hamcrest.home and junit.home. However they are only used in the default classpath (which adds all .jar files in catalina.home, and adds the other directly to the path). The following build.properties file will suffice to compile:

	# local copy of files
	catalina.home: lib/apache-tomcat-6.0.32-deployer
	# dummy
	hamcrest.home: .
	# dummy
	junit.home: .
	# android jar for dex - can change it to a different version of the platform that is installed
	dx.jar: /home/pszcmg/android/android-sdk-linux/build-tools/17.0.0/lib/dx.jar

*	src/placebooks.properties contains some application properties. This includes server URL, persistence information, proxy configuration (if any), map server configuration, configuration of other external services. Currently this needs to be copied by hand to war/WEB-INF/classes/ before "ant war".
*	minimum ant build targets are "ant compile.gwt war"

For desktop testing regular Jetty is available, e.g. [jetty 9](http://download.eclipse.org/jetty/stable-9/dist/). The generated placebooks.war can be copied directly to jetty's webapps/ directory for desktop use.

There seems to be an intermittent (or initial) problem with uploading images, perhaps due to the media directory (by default in placebooks/); I'm not sure when/how this resolves itself at the moment. 

To deploy to i-jetty on Android all Java class files must be converted with DEX. These can then be zipped up together in (say) WEB-INF/lib/classes.dex. E.g. something like the following added to the build.xml, called before "ant war":

	<!-- dex for Android i-jetty -->
	<target name="dex" description="dex class file for Android" depends="compile">
		<mkdir dir="${warfile.dir}/dexclasses"/>
		<unzip dest="${warfile.dir}/dexclasses">
			<patternset>
				<include name="**/*.class"/>
			</patternset>
			<fileset dir="${war.dir}/WEB-INF/lib">
				<include name="*.jar"/>
			</fileset>
		</unzip>
		<copy todir="${warfile.dir}/dexclasses">
			<fileset dir="${war.dir}/WEB-INF/classes">
				<include name="**/*.class"/>
			</fileset>
		</copy>	

		<!-- dex tool from android-sdk -->
		<java jar="${dx.jar}" fork="true">
			<jvmarg value="-Xmx2048M"/>
			<jvmarg value="-Xms512M"/>
                 	<arg value="--dex"/>
 	        	<arg value="--verbose"/>
                	<arg value="--core-library"/>
                	<arg value="--output=${war.dir}/WEB-INF/lib/classes.dex"/>
                	<arg value="--positions=lines"/>
                	<arg value="${warfile.dir}/dexclasses/"/>
		</java>

	</target>

Note: I had to increase dex memory for it to work (as above). Also note dex.jar is in different locations for different versions of the Android platform - try "find"...

Status: still waiting for dex to succeed.

### Kiosk view

I was just thinking of an app which is a full screen web view...

### Prevent leaving app to home

Some stuff on [stackoverflow](http://stackoverflow.com/questions/2068084/kiosk-mode-in-android), but here is a complete [how-to](http://thebitplague.wordpress.com/2013/04/05/kiosk-mode-on-the-nexus-7/):

*	Add HOME intent to the main activity.
*	Create and use a theme without status or title bars.
*	Change behaviour when power added to boot rather than charge ("fastboot oem off-mode-charge 0"). 
*	Modify /system/build.prop as if it hard hw keys, thereby disabling the soft keys.

Potentially also (from the Stackoverflow article) "to stop any other programs from opening by mistake use an Accessibility Service to check for Window State Changed, compare the package to a white or black list and use ActivityManager.killBackgroundProcesses to kill if it shouldn't run."

As an alternative to disabling the soft keys you can avoid starting any other apps (so app list empty), and handle back.

### OSM Map Server

### Custom time-out and lock


### Custom boot animation

### Limit internet access


