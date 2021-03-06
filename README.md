# DroidBot

## About
A robot which automatically interacts with Android apps.

DroidBot sends keyevent, gestures and simulates system events 
in order to exploit more app states automatically.
DroidBot decides which actions to take based on static analysis result of app
and dynamic device information (view hierarchy).

For more details, refer to my [blog posts](http://honeynet.github.io/droidbot/).

## Introduction
DroidBot mainly does following two things:

1. Setting up device environments, include the contacts, SMS logs, 
call logs, GPS mocking, etc. The target app may have access to these resources, thus we 
prepare them before starting the app.

    Multiple env policies can be used for setting up environments. We support:

    + `none` policy which does not set up any environment;
    + `dummy` policy which just mocks same basic environment for all apps;
    + `static` policy which set up environment according to static information of app,
    for example permissions and files which the app have access to;
    + `file` policy which read environment configurations from a json file.

2. Sending events during the app is running. Events includes touch, drag gestures on screen, 
keyevents, and simulated broadcasts, etc.

    Similarly, we have several policies to produce events:
    
    + `none` policy which does not send any event;
    + `monkey` policy which make use of adb `monkey` tool, to produce randomized events;
    + `random` policy which sends randomized events to device
    + `static` policy produces a list of events based on static information of app. Eg. 
    the intent-filters of each app.
    + `dynamic` policy. It is actually the real human-like policy. It monitors the device 
    states, including the running activities, the foreground window, and the hierarchy of current 
    window and sends events according to these information.
    It avoids going to same state too many times by comparing the window hierarchies, and 
    it sends activity-specific intents based on static analysis of app.
    + `file` policy which generates events from a json file.

## Prerequisite

1. `Python` version `2.7`
2. `Java` version `1.7`
3. `Android SDK`, make sure that `platform_tools` and `tools` added to `PATH`
4. `androidviewclient`, install with `sudo easy_install --upgrade androidviewclient`,
or refer to its [wiki](https://github.com/dtmilano/AndroidViewClient/wiki)
5. (Optional) `DroidBox` version `4.1.1`, 
download from [here](http://droidbox.googlecode.com/files/DroidBox411RC.tar.gz)

## Installation

Clone this repo and use pip install:

```shell
git clone https://github.com/honeynet/droidbot.git
pip install -e droidbot
```

## Usage

### Basic Usage

1. Start an emulator or connect to a device using adb.
2. Start DroidBot:
`droidbot -h`

### Usage with DroidBox (without docker)

DroidBox print sensitive behaviours at runtime, which is useful in malware analysis.
DroidBot can be used with DroidBox and is able to capture DroidBox logs.

Step 1. Start droidbox emulator:

1.1 Download DroidBox image
```
wget https://droidbox.googlecode.com/files/DroidBox411RC.tar.gz
tar xfz DroidBox411RC.tar.gz
```

1.2 Create an avd named droidbox

You can either use android avd manager or use `android create avd` command.

1.3 Start the avd with droidbox image
```
cd DroidBox411RC
sh startemu.sh droidbox
```

Step 2. Start DroidBot:
```
droidbot -a <sample.apk> -event dynamic -duration 100 -o droidbot_out
```

### Usage with Docker

Prepare the environment on your host by creating a folder to be shared with the **DroidBot** Docker container. The folder will be used to load samples to be analyzed in **DroidBot**, and also to store output results from **DroidBot** analysis.
```
mkdir -p ~/mobileSamples/out
```

Now pull the ready-made Docker container (about 1.8 GB after extraction) from Honeynet Project's hub:
```
docker pull honeynet/droidbot
```

or, if you prefer, build your own from the GitHub repo:
```
git clone https://github.com/honeynet/droidbot.git
docker build -t honeynet/droidbot droidbot
```

To run the analysis, copy your sample to the folder you created above, then start the container; you will find results in the "out" subfolder.
```
cp mySample.apk ~/mobileSamples/
docker run -it --rm -v ~/mobileSamples:/samples:ro -v ~/mobileSamples/out:/samples/out honeynet/droidbot /samples/mySample.apk
ls ~/mobileSamples/out
```

## Evaluation

DroidBot is evaluated by comparing with DroidBot default mode (which does nothing)
and adb Monkey tool. The results are in [result](/evaluation_reports/README.md).

Or see my visualized evaluation reports at [DroidBot Posts](http://honeynet.github.io/droidbot/).

## Acknowledgement

1. [AndroidViewClient](https://github.com/dtmilano/AndroidViewClient) 
is an amazing tool that simplifies test script creation.
2. [Androguard](http://code.google.com/p/androguard/)
is well-known for reverse-engineering of Android APKs.
