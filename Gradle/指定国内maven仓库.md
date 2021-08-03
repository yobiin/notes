用户目录下进入.gradle目录，新建`init.gradle`文件，写入如下内容：

~~~shell
gradle.projectsLoaded {
    rootProject.buildscript {
        repositories {
            maven { url "https://maven.aliyun.com/repository/public" }
            maven { url "https://maven.aliyun.com/repository/google" }
        }
    }
}

allprojects {
    repositories {
        maven { url "https://maven.aliyun.com/repository/public" }
        maven { url "https://maven.aliyun.com/repository/google" }
    }

}
~~~

