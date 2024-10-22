##### Open Env Config

``````bash
open /home/<username>/.bashrc
``````

##### Configuration Template

append those at end of file

``````bash
export HOME=/home/easing
export DESKTOP=$HOME/Desktop
export JAVA_HOME=$HOME/dev/jdk/jdk17
export GRADLE_HOME=$HOME/dev/gradle/gradle-8.7
export ANDROID_SDK=$HOME/dev/android-sdk
export ANDROID_NDK=$HOME/dev/android-ndk
export GRADLE_USER_HOME=$HOME/.gradle
export GRADLE_PROPERTY=$GRADLE_USER_HOME/gradle.properties
export M2_REPO=$HOME/.m2

export PATH=/sbin
export PATH=$PATH:/bin
export PATH=$PATH:/usr/sbin
export PATH=$PATH:/usr/bin
export PATH=$PATH:/usr/local/sbin
export PATH=$PATH:/usr/local/bin
export PATH=$PATH:$JAVA_HOME/bin
export PATH=$PATH:$GRADLE_HOME/bin
export PATH=$PATH:$ANDROID_SDK
export PATH=$PATH:$ANDROID_SDK/platform-tools
``````

##### Apply Changes

``````bash
source /home/<username>/.bashrc
``````

##### Test Effect

``````bash
echo $JAVA_HOME
echo $PATH
``````

