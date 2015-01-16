
# Android Studio 技巧

> 原创：余俊卿 转载请:<yujunqing@meizu.com>

----

### 加速器篇

#### 1、配置最新的jdk

观察bin/studio.sh 代码，发现如下部分：

````
# ---------------------------------------------------------------------# Locate a JDK installation directory which will be used to run the IDE.# Try (in order): STUDIO_JDK, JDK_HOME, JAVA_HOME, "java" in PATH.# ---------------------------------------------------------------------if [ -n "$STUDIO_JDK" -a -x "$STUDIO_JDK/bin/java" ]; then  JDK="$STUDIO_JDK"elif [ -n "$JDK_HOME" -a -x "$JDK_HOME/bin/java" ]; then  JDK="$JDK_HOME"elif [ -n "$JAVA_HOME" -a -x "$JAVA_HOME/bin/java" ]; then  JDK="$JAVA_HOME"else
XXXXXX
````
我们都知道java版本肯定越高越快！但是我们平时需要使用的java版本可能不是最新的。因此建议下载一个最新的jdk给studio用。所以在~/.zshrc 里配置了如下一行

    export STUDIO_JDK=/usr/local/share/jvm/java8